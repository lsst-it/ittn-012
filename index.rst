:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. note::

   **This technote is not yet published.**

   Graylog k8s deployment and configuration

.. sectnum::

Introduction
============

Hierarchical construction, deployment and configuration of a Graylog chart over GKE

Requirements
============

Google Cloud Platform account
-----------------------------

In order to correctly deploy the chart over GKE (Google Kubernetes Engine), it is
needed for you to have a payed account, and sufficient priviledges to create a 
cluster and nodes among it.


Creating the Cluster
====================


GCloud and kubectl extension
============================


Helm charts and values.yaml
===========================


Ingress Controller
===================


Deploying the charts
====================


Configuring Graylog
===================
.. Main Title

Adding the Inputs
-----------------

1. 
LSST Firewall Syslogs

- allow_override_data: true
- bind_address: 0.0.0.0
- expand_structured_data: true
- force_rdns: false
- number_worker_threads: 2
- override_source: <empty>
- port: 7514
- recv_buffer_size: 262144
- store_full_message: true

Add it, and then "More actions -> Add Static Field":
- Field Name  collector
- Field Value: firewall

2. 
LSST Network Syslogs

- allow_override_data: true
- bind_address: 0.0.0.0
- expand_structured_data: true
- force_rdns: false
- number_worker_threads: 1
- override_source: <empty>
- port: 6514
- recv_buffer_size: 262144
- store_full_message: true

Add it, and then "More actions -> Add Static Field":
- Field Name  collector
- Field Value: network

3. 
LSST Servers Syslogs

- allow_override_data: true
- bind_address: 0.0.0.0
- expand_structured_data: true
- force_rdns: false
- number_worker_threads: 1
- override_source: <empty>
- port: 5514
- recv_buffer_size: 262144
- store_full_message: true

Add it, and then "More actions -> Add Static Field":
- Field Name  collector
- Field Value: servers   


LookUP Tables
-------------

For Graylog to be able of doing some processing with the incoming logs, you need to create LookUP Tables. This allows you to use any of the incomming inputs and process them 
into something you need. 

.. _table-LookUPTable:

.. table:: LookUP Tables.

    +--------+-----------------------+---------------------------------------------------------+------------------+--------------------+
    | Number |        Name           |  Description                                            |  Data Adapter    |  Caches            |
    +========+=======================+=========================================================+==================+====================+
    |   1    |  Source GeoLocation   | Extract and Process Source IP into coordinates          | locate-ip        | store-geolocation  |
    +--------+-----------------------+---------------------------------------------------------+------------------+--------------------+
    |   2    |  Resolve FQDN into IP | Pick the FQDN from a device and translate it into an IP | resolve-dns-type | dns-resolves-cache |
    +--------+-----------------------+---------------------------------------------------------+------------------+--------------------+


Data Adapters
^^^^^^^^^^^^^

This are the escense of the Tables. There are many types (such us CSV Files, Whois for IPs, Ransomware blocklist, among others). The Adapters take the input, i.e. source (which
fot the matters of this example will be an FQDN), and process is according to the engine you select; so, if you selected "DNS Lookup", it will resolve the FQDN into an IP, or if
you select "Randomware blocklist" it will look into an external database and check if the IP is listed there.

.. _table-DataAdapters:

.. table:: Data Adapters.

    +--------+-------------------+------------------+--------------------------------------------------------------------------------------------+
    | Number |        Name       |   Field          | Settings                                                                                   |
    +========+===================+==================+============================================================================================+
    |   1    |  Locate IP        | locate-ip        | File path: /usr/share/graylog/GeoLite2-City.mmdb, DB Type: City database, Refresh: disable |
    +--------+-------------------+------------------+--------------------------------------------------------------------------------------------+
    |   2    |  Resolve DNS name | resolve-dns-type | LookUP Type: Resolve hostname to IPv4, DNS Server: 8.8.4.4, Request Timeout: 10000ms       |
    +--------+-------------------+------------------+--------------------------------------------------------------------------------------------+


Caches
^^^^^^

Determines if you wanna store the processed data from the Data Adapters, where (volatile or storage) and for how long.

.. _table-Caches:

.. table:: Caches.

    +--------+--------------------+--------------------+--------------+-----------------------+--------------------+
    | Number |        Name        |   Field            | Max Entries  |  Expire After Access  | Expire after Write |
    +========+====================+====================+==============+=======================+====================+
    |   1    |  Store GeoLocation | store-geolocation  |    1000      |        60s            |      disable       |
    +--------+--------------------+--------------------+--------------+-----------------------+--------------------+
    |   2    |  DNS Resolve Cache | dns-resolves-cache |     500      |        30s            |      disable       |
    +--------+--------------------+--------------------+--------------+-----------------------+--------------------+



Extractors
----------

Let's say that the source name isn't right (or is not the one you wanted), but the correct one is in between the message field, or that you would like to have a field with the 
username of the user that is running the command and you see that the username is contained in another field. There's were Extractors come in handy: they allow you to extrac a
determine pattern from all logs arrived and turn it into a new field. Extractors also allow you to run the extracted content through a LookUP table, meaning you can process 
and manage the content (like looking an FQDN through a DNS resolver).

Firewall
^^^^^^^^

.. _table-FwExtractors:

.. table:: Firewall Extractors.

    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    | Number |        Name             |                 Description                   |    Type      |    SourceField   |  DstField       |          Configurations          |
    +========+=========================+===============================================+==============+==================+=================+==================================+
    |   1    |  Source Name            | Replace source name with a shrink version     | Substring    |   source         | source          | index [0,5]                      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   2    |  Extract Involve IPs    | Grabs the source and destination IP           | Split&Index  |   message        | src_and_dst_IP  | index=2 & split="{TCP}"          |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   3    |  Source IP with Port    | Takes out the source IP only with the port    | Split&Index  |   src_and_dst_IP | src_IP          | index=1 & split="->"             |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   4    |  Destination IP         | Grabs the destination IP                      | Split&Index  |   src_and_dst_IP | dst_IP          | index=2 & split="->"             |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   5    |  Replace Destination IP | Replace a clean destination IP                | Split&Index  |   dst_IP         | dst_IP          | index=1 & split=":"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   6    |  Remove Port Source IP  | Takes out the port from the source IP         | Split&Index  |   src_IP         | src_IP          | index=1 & split=":"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   7    |  Source Geolocation     | Places the source IP through the LookUp table | LookUP Table |   src_IP         | src_geolocation | lookup_table_name: "GeoLocation" |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   8    |  VPN Username and IP    | Takes the username and IP                     | Split&Index  |   message        | userIP_and_Name | index=2 & split=":"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   9    |  User and Remote IP     | Takes the user and IP into username field     | Split&Index  |   message        | username        | index=1 & split=":"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   10   |  VPN Username           | Replace the VPN username                      | Split&Index  |   username       | username        | index=1 & split="/"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   11   |  VPN User IP            | Takes the remote VPN IP                       | Split&Index  |   username       | vpnIP           | index=2 & split="/"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   12   |  Replace VPN User IP    | Replaces tje VPN IP clean                     | Split&Index  |  userIP_and_Name | vpnIP           | index=2 & split="/"              |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+
    |   13   |  VPN User Location      | Runs the IP through the LookUp table          | LookUP Table |   vpnIP          | vpn_location    | lookup_table_name: "GeoLocation" |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+----------------------------------+


.. notes::

   Firewall Extractors JSON

   {
   "extractors": [
    {
      "title": "Extract involve IPs",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 1,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "src_and_dst_IP",
      "extractor_config": {
        "index": 2,
        "split_by": "{TCP}"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN Username and IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 7,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "userIP_and_Name",
      "extractor_config": {
        "index": 2,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "User and Remote IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 8,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "username",
      "extractor_config": {
        "index": 2,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Remove Port from Source IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 5,
      "cursor_strategy": "copy",
      "source_field": "src_IP",
      "target_field": "src_IP",
      "extractor_config": {
        "index": 1,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Destination IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 3,
      "cursor_strategy": "copy",
      "source_field": "src_and_dst_IP",
      "target_field": "dst_IP",
      "extractor_config": {
        "index": 2,
        "split_by": "->"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Source IP with Port",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 2,
      "cursor_strategy": "copy",
      "source_field": "src_and_dst_IP",
      "target_field": "src_IP",
      "extractor_config": {
        "index": 1,
        "split_by": "->"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN Username",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 9,
      "cursor_strategy": "copy",
      "source_field": "username",
      "target_field": "username",
      "extractor_config": {
        "index": 1,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN User IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 10,
      "cursor_strategy": "copy",
      "source_field": "username",
      "target_field": "vpnIP",
      "extractor_config": {
        "index": 2,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Source Name",
      "extractor_type": "substring",
      "converters": [],
      "order": 0,
      "cursor_strategy": "copy",
      "source_field": "source",
      "target_field": "source",
      "extractor_config": {
        "end_index": 5,
        "begin_index": 0
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Replace VPN User IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 11,
      "cursor_strategy": "copy",
      "source_field": "userIP_and_Name",
      "target_field": "vpnIP",
      "extractor_config": {
        "index": 2,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Replace Destination IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 4,
      "cursor_strategy": "copy",
      "source_field": "dst_IP",
      "target_field": "dst_IP",
      "extractor_config": {
        "index": 1,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Source Name",
      "extractor_type": "substring",
      "converters": [],
      "order": 0,
      "cursor_strategy": "copy",
      "source_field": "source",
      "target_field": "source",
      "extractor_config": {
        "end_index": 5,
        "begin_index": 0
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Destination IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 3,
      "cursor_strategy": "copy",
      "source_field": "src_and_dst_IP",
      "target_field": "dst_IP",
      "extractor_config": {
        "index": 2,
        "split_by": "->"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Extract involve IPs",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 1,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "src_and_dst_IP",
      "extractor_config": {
        "index": 2,
        "split_by": "{TCP}"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Source Geolocation",
      "extractor_type": "lookup_table",
      "converters": [],
      "order": 6,
      "cursor_strategy": "copy",
      "source_field": "src_IP",
      "target_field": "src_geolocation",
      "extractor_config": {
        "lookup_table_name": "GeoLocation"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "User and Remote IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 8,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "username",
      "extractor_config": {
        "index": 2,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN Username",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 9,
      "cursor_strategy": "copy",
      "source_field": "username",
      "target_field": "username",
      "extractor_config": {
        "index": 1,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN User Location",
      "extractor_type": "lookup_table",
      "converters": [],
      "order": 12,
      "cursor_strategy": "copy",
      "source_field": "vpnIP",
      "target_field": "vpn_location",
      "extractor_config": {
        "lookup_table_name": "GeoLocation"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Replace Destination IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 4,
      "cursor_strategy": "copy",
      "source_field": "dst_IP",
      "target_field": "dst_IP",
      "extractor_config": {
        "index": 1,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN User IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 10,
      "cursor_strategy": "copy",
      "source_field": "username",
      "target_field": "vpnIP",
      "extractor_config": {
        "index": 2,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "VPN Username and IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 7,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "userIP_and_Name",
      "extractor_config": {
        "index": 2,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Source IP with Port",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 2,
      "cursor_strategy": "copy",
      "source_field": "src_and_dst_IP",
      "target_field": "src_IP",
      "extractor_config": {
        "index": 1,
        "split_by": "->"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Remove Port from Source IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 5,
      "cursor_strategy": "copy",
      "source_field": "src_IP",
      "target_field": "src_IP",
      "extractor_config": {
        "index": 1,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Replace VPN User IP",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 11,
      "cursor_strategy": "copy",
      "source_field": "userIP_and_Name",
      "target_field": "vpnIP",
      "extractor_config": {
        "index": 2,
        "split_by": "/"
      },
      "condition_type": "none",
      "condition_value": ""
    }
  ],
  "version": "3.1.4"
  }


Network
^^^^^^^

.. _table-NetExtractors:

.. table:: Network Extractors.

    +--------+---------------------+-----------------------------------------------+--------------+------------------+-----------------+---------------------+
    | Number |        Name         |                 Description                   |    Type      |    SourceField   |  DstField       |     Configurations  |
    +========+=====================+===============================================+==============+==================+=================+=====================+
    |   1    |  Extract Source     | Extract the hostname with the port            | Split&Index  |   message        | s_id            | index=1 & split=":" |
    +--------+---------------------+-----------------------------------------------+--------------+------------------+-----------------+---------------------+
    |   2    |  Hostname Extractor | Filter out the port, and replace source field | Split&Index  |   s_id           | source          | index=2 & split=":" |
    +--------+---------------------+-----------------------------------------------+--------------+------------------+-----------------+---------------------+


.. notes::

   Network Extractors JSON
   {
   "extractors": [
    {
      "title": "Extract Source",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 0,
      "cursor_strategy": "copy",
      "source_field": "message",
      "target_field": "s_id",
      "extractor_config": {
        "index": 1,
        "split_by": ":"
      },
      "condition_type": "none",
      "condition_value": ""
    },
    {
      "title": "Hostname Extractor",
      "extractor_type": "split_and_index",
      "converters": [],
      "order": 0,
      "cursor_strategy": "copy",
      "source_field": "s_id",
      "target_field": "source",
      "extractor_config": {
        "index": 2,
        "split_by": "\""
      },
      "condition_type": "none",
      "condition_value": ""
    }
   ],
   "version": "3.1.4"
   }

Servers
^^^^^^^

.. _table-ServerExtractors:

.. table:: Servers Extractors.

    +--------+---------------------+-----------------------------------------------+--------------+---------------+-------------+-----------------------------------------+
    | Number |        Name         |                 Description                   |    Type      |  SourceField  |  DstField   |           Configurations                |
    +========+=====================+===============================================+==============+===============+=============+=========================================+
    |   1    |  FQDN to IP resolve | Take the FQDN and resolve it into the IP      | LookUP Table |     source    | fqdn_to_ip  | lookup_table_name: "Resolve FQDN to IP" |
    +--------+---------------------+-----------------------------------------------+--------------+---------------+-------------+-----------------------------------------+
    
.. notes::

    {
    "extractors": [
    {
      "title": "FQDN to IP resolve",
      "extractor_type": "lookup_table",
      "converters": [],
      "order": 0,
      "cursor_strategy": "copy",
      "source_field": "source",
      "target_field": "fqdn_to_ip",
      "extractor_config": {
        "lookup_table_name": "fqdn-to-ip"
      },
      "condition_type": "none",
      "condition_value": ""
    }
   ],
   "version": "3.1.4"
   }

Dashboards
----------

Centralized Logging System
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _table-CLSDashboard:

.. table:: CLS Dashboard.

    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    | Number |                Name                       |                                         Search Query                                            |                Type              | Field/Stacked Fields   |
    +========+===========================================+=================================================================================================+==================================+========================+
    |   1    |  Top Access to Servers                    | message:"Started Session" AND collector:"servers" AND NOT message:"root" OR NOT message:"admin" | Quick Value with Pie Chart&Table | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   2    |  Recent Root Access                       | message:"Started Session" AND collector:"servers" AND message:"root"                            | Quick Value with Pie Chart&Table | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   3    |  Failed Sudo Access                       | collector:servers AND message:"FAILED SU"                                                       | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   4    |  Failed Queries                           | source:dns?.ls.lsst.org OR source:dns1.dev.lsst.org OR message:"named" AND message:"failed"     | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   5    |  Succesfull Logins                        | message:"Started Session" AND collector:"servers" AND NOT message:"root" OR NOT message:"admin" | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   6    |  Top Access to NetDevices                 | message:"Login Success" AND collector:"network"                                                 | Quick Value with Pie Chart&Table | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   7    |  Flapping Interfaces                      | collector:network AND message:"flapping"                                                        | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   8    |  NetDev Logins                            | message:"Login Success" AND collector:"network"                                                 | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   9    |  Failed Logins                            | collector:network AND message:"Invalid-Credentials"                                             | Quick Value                      | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   10   |  DNS hits LS/Dev                          | source:dns?.ls.lsst.org OR source:dns1.dev.lsst.org OR message:"named"                          | Quick Value with Pie Chart&Table | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   11   |  Top Servers Talkers                      | collector:servers                                                                               | Histogram                        | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   12   |  NetDev Interface Change State            | collector:network AND message: "changed state"                                                  | Histogram                        | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   13   |  Top NetDev Talkers                       | collector:network                                                                               | Histogram                        | source/none            |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   14   |  Authorized VPN Users Location            | Runs the IP through the LookUp table                                                            | GeoMap                           | src_location/none      |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   15   |  Potencial Attacks through IP Geolocation | Runs the IP through the LookUp table                                                            | GeoMap                           | src_location/none      |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    |   16   |  VPN Location - Username - IP             | collector:firewall AND source:openv                                                             | Quick Value with Table           | source/username, vpnIP | 
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+----------------------------------+------------------------+
    

Common Issues and Solutions
===========================

Fail index
----------

Due to many reasons, one of them you ran out of space in the data pod, index might crush, preventing graylog to right more indices into it. The most common way of noticing it, is because
graylog will find nothing through the search query. To solve it, you can dump the fail indexes through a curl:

.. note::

   Log into a pod that can reach the local k8s network:
      kubectl exec -it -n graylog graylog-elasticsearch-data-0 -- /bin/bash

   Run the following command:
      curl -XPUT -H "Content-Type: application/json"  http://localhost:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'

   If everything goes well, you should get the following output from the above command:                                                                                                                                 
      {"acknowledged":true}

Missing GeoLite Database
------------------------

Since GeoLite is done through an API, there is no persistent storage for it in the GKE environment. In order to workaround this issue, you can manually copy the database into the graylog pods:

.. note::

      for i in {0,1,2}; do kubectl cp ~/GeoLite2-City_20200414/GeoLite2-City.mmdb graylog/graylog-$i:/usr/share/graylog/GeoLite2-City.mmdb; done
