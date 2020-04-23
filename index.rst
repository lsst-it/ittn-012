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
.. Second Title

1. Keeping to know the structure order

   .. note::

      Keeping to know the structure order

2. Keeping to know the structure order
3. Keeping to know the structure order


Extractors
----------

Firewall
^^^^^^^^

.. _table-FwExtractors:

.. table:: Firewall Extractors.

    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    | Number |        Name             |                 Description                   |    Type      |    SourceField   |  DstField       |      Configurations     |
    +========+=========================+===============================================+==============+==================+=================+=========================+
    |   1    |  Source Name            | Replace source name with a shrink version     | Substring    |   source         | source          |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   2    |  Extract Involve IPs    | Grabs the source and destination IP           | Split&Index  |   message        | src_and_dst_IP  |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   3    |  Source IP with Port    | Takes out the source IP only with the port    | Split&Index  |   src_and_dst_IP | src_IP          |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   4    |  Destination IP         | Grabs the destination IP                      | Split&Index  |   src_and_dst_IP | dst_IP          |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   5    |  Replace Destination IP | Replace a clean destination IP                | Split&Index  |   dst_IP         | dst_IP          |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   6    |  Remove Port Source IP  | Takes out the port from the source IP         | Split&Index  |   src_IP         | src_IP          |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   7    |  Source Geolocation     | Places the source IP through the LookUp table | LookUP Table |   src_IP         | src_geolocation |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   8    |  VPN Username and IP    | Takes the username and IP                     | Split&Index  |   message        | userIP_and_Name |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   9    |  User and Remote IP     | Takes the user and IP into username field     | Split&Index  |   message        | username        |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   10   |  VPN Username           | Replace the VPN username                      | Split&Index  |   username       | username        |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   11   |  VPN User IP            | Takes the remote VPN IP                       | Split&Index  |   username       | vpnIP           |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   12   |  Replace VPN User IP    | Replaces tje VPN IP clean                     | Split&Index  |  userIP_and_Name | vpnIP           |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    |   13   |  VPN User Location      | Runs the IP through the LookUp table          | LookUP Table |   vpnIP          | vpn_location    |        index [0,5]      |
    +--------+-------------------------+-----------------------------------------------+--------------+------------------+-----------------+-------------------------+
    

II. 
   - Name:                  Extract Involve IPs 
   - Description: 
   - Type:                  Split & Index 
   - Source Field:          message 
   - New Field:             src_and_dst_IP 
   - Configuration:
      i-.  index:           "2"
      ii-. split_by:        "{TCP}"

III. 
   - Name:                  Source IP with Port 
   - Description: 
   - Type:                  Split&Index 
   - Source Field:          src_and_dst_IP 
   - New Field:             src_IP 
   - Configuration:
      i-.  index:           "1"
      ii-. split_by:        "->"

IV. 
   - Name:                  Destination IP 
   - Description: 
   - Type:                  Split & Index 
   - Source Field:          src_and_dst_IP 
   - New Field:             dst_IP 
   - Configuration:
      i-.  index:           "2"
      ii-. split_by:        "->"

V. 
   - Name:                  Replace Destination IP 
   - Description: 
   - Type:                  Split & Index 
   - Source Field:          dst_IP 
   - New Field:             dst_IP 
   - Configuration:
      i-. index:             "1"
      ii-. split_by:         ":"

VI. 
   - Name:                   Remove Port from Source IP 
   - Description: 
   - Type:                   Split & Index 
   - Source Field:           src_IP 
   - New Field:              src_IP 
   - Configuration:
      i-.  index:            "1"
      ii-. split_by:         ":"

VII. 
   - Name:                   Source Geolocation 
   - Description: 
   - Type:                   LookUP Table 
   - Source Field:           src_IP 
   - New Field:              src_geolocation 
   - Configuration:
      i-. lookup_table_name: "GeoLocation"

VIII. 
   - Name:                   VPN Username and IP 
   - Description: 
   - Type:                   Split & Index 
   - Source Field:           message 
   - New Field:              userIP_and_Name 
   - Configuration:
      i-.  index:            "2"
      ii-. split_by:         ":"

IX. 
   - Name:                   User and Remote IP 
   - Description: 
   - Type:                   Split & Index 
   - Source Field:           message 
   - New Field:              username 
   - Configuration:
      i-.  index:            "1"
      ii-. split_by:         ":"

X. 
   - Name:                   VPN Username 
   - Description: 
   - Type:                   Split & Index 
   - Source Field:           username 
   - New Field: username 
   - Configuration:
      i-.  index:            "1"
      ii-. split_by:         "/"

XI. 
   - Name:                   VPN User IP 
   - Description:
   - Type:                   Split & Index
   - Source Field:           username 
   - New Field:              vpnIP 
   - Configuration:
      i-.  index:            "2"
      ii-. split_by:         "/"

XII. 
   - Name:                   Replace VPN User IP 
   - Description: 
   - Type:                   Split & Index 
   - Source Field:           userIP_and_Name 
   - New Field:              vpnIP 
   - Configuration:
    -.  index:            "2"
      ii-. split_by:         "/"

XIII. 
   - Name:                   VPN User Location 
   - Description: 
   - Type:                   LookUP Table 
   - Source Field:           vpnIP 
   - New Field:              vpn_location 
   - Configuration:
     - lookup_table_name: "GeoLocation"



Network
^^^^^^^

a. S

Servers
^^^^^^^

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

