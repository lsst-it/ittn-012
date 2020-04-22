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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^^^^^
.. Second Title

1. Keeping to know the structure order

   .. note::

      Keeping to know the structure order

2. Keeping to know the structure order
3. Keeping to know the structure order


Extractors
^^^^^^^^^^

Firewall
--------

.. _table-FwExtractos:

.. table:: Firewall Extractors.

    +------------------------+------------+-------------+----------+--------------+--------------+--------------------+
    |       Number           |     Name   | Description |   Type   | Source Field |   New Field  |   Configurations   |
    | (header rows optional) |            |             |          |              |              |                    |
    +========================+============+=============+==========+==============+==============+====================+
    | body row 1, column 1   | column 2   | column 3    | column 4 |              |              |                    |
    |                        | with many  | spans       |          |              |              |                    |
    |                        | rows       | both        |          |              |              |                    |
    +------------------------+------------+ rows        +----------+--------------+--------------+--------------------+
    | body row 2             | ...        |             | ...      |              |              |                    |
    +------------------------+------------+-------------+----------+--------------+--------------+--------------------+



I. 
   - Name:                  Source Name 
   - Description:  
   - Type:                  Substring 
   - Source Field:          source 
   - New Field:             source 
   - Configuration:
      i-.  end_index:       "5"
      ii-. begin_index:     "0"

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
   - Type:                  Split & Index 
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
-------

a. S

Servers
-------

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

