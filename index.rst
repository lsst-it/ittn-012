:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. note::

   **This technote is not yet published.**

   Graylog k8s deployment and configuration


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

1. LSST Firewall Syslogs
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

2. LSST Network Syslogs
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

3. LSST Servers Syslogs
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

Extractors
----------

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


Servers
^^^^^^^

.. _table-ServerExtractors:

.. table:: Servers Extractors.

    +--------+---------------------+-----------------------------------------------+--------------+---------------+-------------+-----------------------------------------+
    | Number |        Name         |                 Description                   |    Type      |  SourceField  |  DstField   |           Configurations                |
    +========+=====================+===============================================+==============+===============+=============+=========================================+
    |   1    |  FQDN to IP resolve | Take the FQDN and resolve it into the IP      | LookUP Table |     source    | fqdn_to_ip  | lookup_table_name: "Resolve FQDN to IP" |
    +--------+---------------------+-----------------------------------------------+--------------+---------------+-------------+-----------------------------------------+
    

Dashboards
----------

Centralized Logging System
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _table-CLSDashboard:

.. table:: CLS Dashboard.

    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    | Number |                Name                       |                                         Search Query                                            | Stacked Fields  |  Pie Chart  |  Data Table |
    +========+===========================================+=================================================================================================+=================+=============+=============+
    |   1    |  Top Access to Servers                    | message:"Started Session" AND collector:"servers" AND NOT message:"root" OR NOT message:"admin" |    none         |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   2    |  Recent Root Access                       | message:"Started Session" AND collector:"servers" AND message:"root"                            |    none         |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   3    |  Failed Sudo Access                       | collector:servers AND message:"FAILED SU"                                                       |    none         |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   4    |  Failed Queries                           | source:dns?.ls.lsst.org OR source:dns1.dev.lsst.org OR message:"named" AND message:"failed"     |    none         |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   5    |  Succesfull Logins                        | message:"Started Session" AND collector:"servers" AND NOT message:"root" OR NOT message:"admin" |    none         |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   6    |  Top Access to NetDevices                 | message:"Login Success" AND collector:"network"                                                 |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   7    |  Flapping Interfaces                      | collector:network AND message:"flapping"                                                        |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   8    |  NetDev Logins                            | message:"Login Success" AND collector:"network"                                                 |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   9    |  Failed Logins                            | collector:network AND message:"Invalid-Credentials"                                             |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   10   |  DNS hits LS/Dev                          | source:dns?.ls.lsst.org OR source:dns1.dev.lsst.org OR message:"named"                          |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   11   |  Top Servers Talkers                      | collector:servers                                                                               |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   12   |  NetDev Interface Change State            | collector:network AND message: "changed state"                                                  |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   13   |  Top NetDev Talkers                       | collector:network                                                                               |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   14   |  Authorized VPN Users Location            | Runs the IP through the LookUp table                                                            |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   15   |  Potencial Attacks through IP Geolocation | Runs the IP through the LookUp table                                                            |   none          |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    |   16   |  VPN Location - Username - IP             | collector:firewall AND source:openv                                                             | username, vpnIP |             |             |
    +--------+-------------------------------------------+-------------------------------------------------------------------------------------------------+-----------------+-------------+-------------+
    

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

..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   Hierarchical instructions for graylog deployment over GKE and all configurations for dashboards, extractors and lookup tables

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa

