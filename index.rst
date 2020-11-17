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

Release Testing
===============

The mechanisms for deploying the software to the test stand are covered in other documents :cite:`TSTN-019`.
The testing assumes that all components have been delivered to the test stand and the components are in the lowest commandable state.
The next sections will cover the tests that should be run on a new release.
They are roughly specified in an order, but that does not mean the order must be adhered to for every test run.
However, all of the tests should be run unless there are serious extenuating circumstances.

The testing routine currently employs the LSST Operations Visualization Environment (LOVE) and Jupyter notebooks via SAL-aware nublado for test execution. 
If notebooks are used for the testing, an effort should be made to convert it into a script
that LOVE can run.
How to use LOVE and nublado are covered in other documentation.
The use of LOVE assumes that the *ScriptQueue*\ s are already ``ENABLED`` for operating.
Future work will be to employ a system where scripts can be delivered to the *ScriptQueue*\  to automate the test running process.

.. _Enabled_Test:

Enabled Test
------------

This test involves bringing all components from their lowest commandable state to ``ENABLED`` state.
This may involve some specific configuration for the test stand that is being used to conduct the test.
To test this, LOVE is used to run scripts to bring up the components.
First, LOVE is used to bring both of the *ScriptQueue*\ s to ``ENABLED`` state via the provided interfaces.
Next, the other observatory systems, the *Watcher* and both *Schedulers*\ , are brought up by using the ``set_summary_state.py`` script from either *ScriptQueue*\ . 
Next, the Auxiliary Telescope (AT) components are to be brought up using the AT *ScriptQueue* with the ``auxtel/enable_atcs.py`` and ``auxtel/enable_latiss.py`` scripts.
The current configuration for the NTS is:

auxtel/enable_atcs.py

.. code-block:: bash

  athexapod: ncsa_202002.yaml
  atdome: test
  ataos: current

auxtel/enable_latiss.py

.. code-block:: bash

  atcamera: Normal

The LATISS systems might not be running yet except for the *ATSpectrograph*, so one may need to use this configuration in the ``enable_latiss.py`` script in order to not have it fail on timeouts waiting for components that aren't running.

.. code-block:: bash

 ignore:
   - atcamera
   - atheaderservice
   - atarchiver

Next, the Main Telescope (MT) components are to be brought up using the MT *ScriptQueue* with the ``maintel/enable_mtcs.py`` and ``maintel/enable_comcam.py`` scripts.
The current configuration for the NTS is:

maintel/enable_mtcs.py

.. code-block:: bash

  mtm1m3: Default

maintel/enable_comcam.py

.. code-block:: bash

  cccamera: Normal

After each step, you should verify that all components under command have transitioned to the ``ENABLED`` state.

Standby Test
-------------

For this test, the observatory systems (\ *Watcher*\ , *ScriptQueue*\ s and *Scheduler*\ s) are not brought to ``STANDBY`` state as operationally they should always be running unless a software upgrade is being performed.

First, send the MT components to ``STANDBY`` by using the ``standby_mtcs.py`` and ``standby_comcam.py`` scripts from the MT *ScriptQueue*\ .
Next, send the AT components to ``STANDBY`` by using the ``standby_atcs.py`` and ``standby_latiss.py`` scripts from the AT *ScriptQueue*\ .
If some LATISS systems are not running, use the ignore list used for the ``enable_lattis.py`` script.

Track Target Test
-----------------

This test assumes that all AT and MT components are in ``ENABLED`` state.
Normally, this test is run as a long term one, either overnight or for a few days.
To run the test, launch the ``auxtel/track_target.py`` and ``maintel/track_target.py`` scripts from the respective *ScriptQueue*\ s.
The same configuration for both scripts is used since it is tuned to avoid altitude limits when running the telescopes for long durations.

.. code-block:: bash

  ra: 20.
  dec: -77.
  rot_value: -88.
  rot_type: PhysicalSky
  track_for: 18000

The last configuration parameter sets a "blocking" time on the script. This allows you to queue up enough scripts for a long duration test.
The scripts need to be relaunched every so often in order to avoid hitting limits on the rotators.
The number of scripts necessary for the queue is determined by the duration the test should be performed.
At the conclusion of the test, both AT and MT should be commanded to stop tracking.
This is accomplished by using the ``run_command.py`` script in the MT *ScriptQueue*\  with the following configuration.

.. code-block:: bash

  cmd: stopTracking
  component: MTPtg

and in the AT *ScriptQueue* with the following configuration.

.. code-block:: bash

  cmd: stopTracking
  component: ATPtg

Calibration Test
----------------

This test assumes that all AT and MT components are in ``ENABLED`` state.
The ComCam test is currently run from a notebook that can be found `here <https://github.com/mareuter/notebooks/blob/master/LSST/CSC_Testing/CC_Calibration.ipynb>`_.
The test parks the MT, takes a set of bias and dark frames, slews the MT to the flat field position, takes a set of flats and slews the MT back to the park position.
The LATISS systems to perform this type of test are still unavailable.

Restart Test
------------

This test can either be done with the system in ``ENABLED`` or ``STANDBY``\ .
The test can only be accomplished by running the ``set_summary_state.py`` script with all component states set to ``OFFLINE``\ .
The *Watcher* and *ScriptQueue*\ s should be done last.
Once all components are verified to be ``OFFLINE``, the entire system needs to be turned off and restarted.
The procedure for turning off the different subsystems is not covered in this document.
Once the entire systems have been restarted, the :ref:`Enabled_Test` should be run again.

.. rubric:: References

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
    :style: lsst_aa
