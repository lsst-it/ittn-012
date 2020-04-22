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

LookUP Tables
^^^^^^^^^^^^^

Dashboard
^^^^^^^^^

Centralized Logging System
--------------------------

a.Top Access to Servers
b.Recent Root Access
c.Failed Sudo Access
d.Top Access to NetDevices
e.Flapping Interfaces
f.Successfull Loggins
g.Failed Logins
h.DNS hits LS/Dev
i.Top Servers Talkers
j.NetDev Interface Change State
k.Top NetDev Talkers
l.Authorized VPN Users Location
m.Potencial Attacks through IP GeoLocation
n.VPN Location - Username - IP


Extractors
^^^^^^^^^^

Firewall
--------

a. Name:         Source Name 
   Description: 
   Type:         Substring
   Source Field: source
   New Field:    source
   Configuration:
      - end_index:   "5"
      - begin_index: "0"

b. Name:         Extract Involve IPs
   Description: 
   Type:         Split & Index
   Source Field: message
   New Field:    src_and_dst_IP
   Configuration:
      - index:    "2"
      - split_by: "{TCP}"
      
c. Name:         Source IP with Port
   Description: 
   Type:         Split & Index
   Source Field: src_and_dst_IP
   New Field:    src_IP
   Configuration:
      - index:    "1"
      - split_by: "->"
   
d. Name:         Destination IP 
   Description: 
   Type:         Split & Index
   Source Field: src_and_dst_IP
   New Field:    dst_IP
   Configuration:
      - index:    "2"
      - split_by: "->"
   
e. Name:         Replace Destination IP
   Description: 
   Type:         Split & Index
   Source Field: dst_IP
   New Field:    dst_IP
   Configuration:
      - index:    "1"
      - split_by: ":"

f. Name:         Remove Port from Source IP
   Description: 
   Type:         Split & Index
   Source Field: src_IP
   New Field:    src_IP
   Configuration:
      - index:    "1"
      - split_by: ":"

g. Name:         Source Geolocation
   Description: 
   Type:         LookUP Table
   Source Field: src_IP
   New Field:    src_geolocation
   Configuration:
      - lookup_table_name: "GeoLocation"

h. Name:         VPN Username and IP
   Description: 
   Type:         Split & Index
   Source Field: message
   New Field:    userIP_and_Name
   Configuration:
      - index:    "2"
      - split_by: ":"

i. Name:         User and Remote IP
   Description: 
   Type:         Split & Index
   Source Field: message
   New Field:    username
   Configuration:
      - index:    "1"
      - split_by: ":"

j. Name:         VPN Username
   Description: 
   Type:         Split & Index
   Source Field: username
   New Field:    username
   Configuration:
      - index:    "1"
      - split_by: "/"
   
k. Name:         VPN User IP
   Description: 
   Type:         Split & Index
   Source Field: username
   New Field:    vpnIP
   Configuration:
      - index:    "2"
      - split_by: "/"

l. Name:         Replace VPN User IP
   Description: 
   Type:         Split & Index
   Source Field: userIP_and_Name
   New Field:    vpnIP
   Configuration:
      - index:    "2"
      - split_by: "/"

m. Name:         VPN User Location
   Description: 
   Type:         LookUP Table
   Source Field: vpnIP
   New Field:    vpn_location
   Configuration:
      - lookup_table_name: "GeoLocation"
   


Network
-------

a. S

Servers
-------

<<<<<<< HEAD
Firewall
--------
=======

>>>>>>> e9445c0... updates

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

