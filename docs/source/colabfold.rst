======================
Running ColabFold
======================

Some instructions for running `ColabFold`_ on epyc. I presented some of these instructions and a general overview of AlphaFold2 and ColabFold during an SBDR seminar (`slides`_).

These are basic instructions for connecting to epyc and running a ColabFold job from the command line. It is also possible to use Microsoft's `VSCode`_ to connect if you are wanting a better experience.

Basic command line instructions
===============================

.. Note::
   These instructions assume you have a remote shell open on epyc, which is outfitted with 2 NVIDIA A100-80 GPUs.

Activate ``colabfold`` conda environment.
*****************************************

``colabfold_batch`` is the command line tool you will be using. It is installed into a preconfigured conda Python environment named ``colabfold``. If your default shell is configured properly you should be able to activate the ``colabfold`` conda environment with this command:

.. code-block:: bash

   conda activate colabfold


For most users with the default ``bash`` shell the above command should just work. If you are using ``tcsh`` or if you have twiddled with your shell configuration files in the past you may get a warning that ``conda`` can't be found. This means your shell is not yet configured to use conda. You can try initializing ``conda`` with this command:

.. code-block:: bash

   /usr/local/anaconda3/bin/conda init


Then logout and log back in to ensure the changes are applied to your shell.

You can view the available conda environments:

.. code-block:: bash

   conda env list


and activate the ``colabfold`` environment.

.. code-block:: bash

   conda activate colabfold


If this worked your shell prompt should look something like this with the name of the active conda environment in parentheses at the beginning of your prompt:

.. code-block:: bash

   (colabfold) [16:58]username@epyc:~$


.. Attention::

   If you are struggling to get the ``colabfold`` conda environment activated or run into other problems please contact Scott

Create a working directory
**************************

You will want to enforce some organization for your ``colabfold`` data so make a directory.

.. code-block:: bash

   mkdir colabfold_data


and make a dedicated directory for your protein/system of interest.

.. code-block:: bash

   cd colabfold_data
   mkdir my_prot


Create your fasta sequence file
*******************************

This is quite simple if you have a single chain. For example create a file named ``my_prot.fasta`` (you can of course name it whatever you want)

.. code-block::
   :caption: An example of a ``fasta`` file

   >1RDR_1|Chain A|POLIOVIRUS 3D POLYMERASE|Human poliovirus 1 (12081)
   GEIQWMRPSKEVGYPIINAPSKTKLEPSAFHYVFEGVKEPAVLTKNDPRLKTDFEEAIFSKYVGNKITEVDEYMKEAVDHYAGQLMSLDINTEQMCLEDAMYGTDGLEALDLSTSAGYPYVAMGKKKRDILNKQTRDTKEMQKLLDTYGINLPLVTYVKDELRSKTKVEQGKSRLIEASSLNDSVAMRMAFGNLYAAFHKNPGVITGSAVGCDPDLFWSKIPVLMEEKLFAFDYTGYDASLSPAWFEALKMVLEKIGFGDRVDYIDYLNHSHHLYKNKTYCVKGGMPSGCSGTSIFNSMINNLIIRTLLLKTYKGIDLDHLKMIAYGDDVIASYPHEVDASLLAQSGKDYGLTMTPADKSATFETVTWENVTFLKRFFRADEKYPFLIHPVMPMKEIHESIRWTKDPRNTQDHVRSLCLLAWHNGEEEYNKFLAKIRSVPIGRALLLPEYSTLYRRWLDSF

To fold a single chain this is all you will need in your ``my_prot`` directory.

Run ColabFold on a Monomer
**************************

There are many options available when running ``colabfold_batch`` which you can see with the ``--help`` flag.

.. code-block:: bash

   colabfold_batch --help

If you just want to use the default settings it's as simple as:

.. code-block:: bash

   colabfold_batch my_prot.fasta output_dir

This will read your fasta sequence, calculate an MSA using **MMseqs2**, perform **AlfaFold2** inference, and output all results to the ``output_dir`` directory.

If you want to use **amber** to relax the model provided by AF2 and use the A100 GPUs to make relaxation even faster you would provide the ``--amber`` and ``--use-gpu-relax`` command line options:

.. code-block:: bash

   colabfold_batch --amber --use-gpu-relax --model-type auto my_prot.fasta output_dir

Run ColabFold on a Multimer
***************************

Under the hood **ColabFold** uses the inference models from **AlphaFold2** to predict a 3D structure from your sequence. There are four different **AlphaFold2** models available, including ``alphafold2_multimer_v1``, ``alphafold2_multimer_v2``, and ``alphafold2_multimer_v3``. The default is ``auto`` (which uses ``alphafold2_ptm`` for monomers and ``alphafold2_multimer_v3`` for complexes.)

If you are predicting a multimer there are some gotchas when preparing the fasta file. Talk to me if you run into errors. Essentially you need to create your fasta file like this (with a ``:`` after each chain, but **not** after the last chain)

.. code-block::
   :caption: An example of a ``multimer.fasta`` file to predict a homo hexamer.

   > 1BJP_homohexamer
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR:
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR:
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR:
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR:
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR:
   PIAQIHILEGRSDEQKETLIREVSEAISRSLDAPLTSVRVIITEMAKGHFGIGGELASKVRR

And then fire off your ``colabfold_batch`` job:

.. code-block:: bash

   colabfold_batch --amber --use-gpu-relax --model-type alphafold2_multimer_v3 multimer.fasta output_dir_for_multimer

Monitoring the GPU status
*************************

You can use ``gpustat`` to see the status of our two A100s which should output something like this:

.. code-block:: bash

   (colabfold) [17:14]username@epyc:~$gpustat
   epyc Thu Jul 20 17:26:13 2023  535.54.03
   [0] NVIDIA A100 80GB PCIe | 35'C,   0 % |  1007 / 81920 MB | gdm(63M) gdm(47M)
   [1] NVIDIA A100 80GB PCIe | 35'C,   0 % |   874 / 81920 MB |

The default GPU that ``colabfold_batch`` will use is ``0``, but if multiple jobs pile up on the first GPU and the second one (``1``) is unused then that is not very good. You can specify which GPU you would like to use by setting the ``CUDA_VISIBLE_DEVICES`` environment variable in your shell just before submitting the job.

.. code-block:: bash

   export CUDA_VISIBLE_DEVICES=1

This would make the second GPU the target for jobs.

.. Note::
   ``0`` = first GPU
   ``1`` = second GPU

Using Microsoft Visual Studio Code
##################################

The benefit of using VSCode is that you have a nice environment for editing files (rather than using `vim` in a terminal).

I'll write these instructions up later.

.. _ColabFold: https://github.com/sokrypton/ColabFold
.. _slides: https://docs.google.com/presentation/d/1UTAKlQKiFwy4Vz4GmS0MuoNuwBKDRfmkfTO2Meo1XdM/edit?usp=sharing
.. _VSCode: https://code.visualstudio.com/