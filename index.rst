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

a. Source Name (Type Substring)
   Description: Trying to extract data from source into source, leaving the original intact.


b. Extract Involve IPs (Type Split & Index)
   Description: Trying to extract data from message into src_and_dst_IP, leaving the original intact.

c. Source IP (Type Split & Index)
   Description: Trying to extract data from src_and_dst_IP into src_IP, leaving the original intact.

d. Destination IP 

e. Replace Destination IP

f.

g.

h.

i.

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

