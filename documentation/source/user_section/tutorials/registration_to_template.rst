.. _registration-to-template:

Registration to template
########################

This tutorial demonstrates how to use SCT's command-line scripts to register single-modality anatomical MRI scans to the PAM50 Template. SCT recommends that you read through the :ref:`pam50` page before starting this tutorial, as it provides an overview of the template, as well as context for why it is used.

The following processing pipeline will be covered in this tutorial:

  #. Prerequisite steps for registration

     a. Spinal cord segmentation (``sct_deepseg_sc``)
     b. Vertebral/disc labeling (``sct_label_vertebrae``)

  #. Single-modality registration to template (``sct_register_to_template``)
  #. Post-registration steps

     a. Transforming template objects into subject coordinate spaces (``sct_warp_template``)
     b. Computing spinal cord metrics (``sct_process_segmentation``)

.. TODO: Update PAM50 template page with information from Registration section

.. warning::

   This tutorial uses sample MRI images that must be retrieved beforehand. Please download and unzip `sct_course_london20.zip <https://osf.io/bze7v/?action=download>`_ , then open up the unzipped folder in your terminal and verify its contents using ``ls``.

   .. code:: sh

      ls
      # Output:
      # multi_subject single_subject

   We will be using images from the ``single_subject/data`` directory, so navigate there and verify that it contains subdirectories for various MRI image contrasts using ``ls``.

   .. code:: sh

      cd single_subject/data
      ls
      # Output:
      # dmri  fmri  LICENSE.txt  mt  README.txt  t1  t2  t2s


Spinal cord segmentation
************************

.. note::

   If you have already completed the :ref:`spinalcord-segmentation` tutorial, your ``t2/`` directory should contain a file called ``t2_seg.nii.gz``. If so, you may skip this section and simply reuse this file instead.

Vertebral/disc labeling
***********************

Single modality registration to template
****************************************

Transforming template objects into subject coordinate spaces
************************************************************

Computing spinal cord metrics
*****************************

