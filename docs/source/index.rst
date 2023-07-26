Welcome to SIBYLS Beamline documentation!
=========================================

These documentation pages are intended primarily for user's of the `SIBYLS Beamline`_.

.. _SIBYLS Beamline: https://bl1231.als.lbl.gov


Instructions
------------

:doc:`colabfold`
   How to run ColabFold on ``epyc``, our GPU-enabled Linux machine equipped with dual `NVIDIA A100-80`_ GPUs.

:doc:`nomachine`
   How to configure `NoMachine NX`_ client and connect to the beamline computers

.. _NoMachine NX: https://www.nomachine.com/
.. _NVIDIA A100-80: https://www.nvidia.com/en-us/data-center/a100/

.. Hidden TOCs

.. toctree::
   :maxdepth: 2
   :caption: SSH Stuff
   :hidden:

   ssh/config
   ssh/keys

.. toctree::
   :maxdepth: 2
   :caption: General
   :hidden:

   colabfold/basic_cli
   colabfold/vscode
   nomachine
