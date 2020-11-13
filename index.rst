:tocdepth: 1

.. sectnum::

.. note::

   As new release of the control system software are available, a standardized set of tests should be run in order to provided a minimum confidence level for system functionality. This document will outline the required tests to accomplish this goal.

Introduction
============

To ensure the best result when deploying new versions of the control system to the summit, the new version should be run through a set of tests on a dedicated test stand.
The list of tests should exercise as many aspects of the operation system as possible to ensure best coverage.
However, the test stands are limited by not having hardware backing on many of the systems.
This implies that there WILL be gaps in coverage and the same operational interactions may still fail when the new version is deployed to the summit.
Since we cannot have a second copy of the Rubin Observatory, we must accept this drawback as a risk for upgrading.
There are plans for having a test stand with the capability of more hardware simulators, but that is a work in progress.
However, that expansion of hardware simulators still will not cover all necessary systems.
The project does have a hardware backed simulator for the LATISS and ComCam camera systems and this is currently deployed at the NCSA test stand (NTS).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
