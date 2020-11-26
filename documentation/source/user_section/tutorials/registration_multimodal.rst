.. _registration-multimodal:

now that you have registered your anatomical data, the next obvious step is to also register another image that was aquired during the same session (e.g. dti, mt image) the way you do that is by registering your <> data to the anatomical data that was aquired during the same session, then you would concatenate the warping fields. warping between anatomical <-> template, and the metric <-> anatomical. if you concatenate the two then you have a single transformation between the template and the metric. because it is a bijective warping field you also have the metric/template transofrmation.

and once you have that, you can bring the template objects (atlas, wm tracts) inside the subject space and then you can quantify mtr, f8, etc. in various wm tracts. and you can also take advantage of the automatic vertebral labeling and you can compute an average csa between t2 and t5 for example.


Registration (Multimodal)
#########################

.. code:: sh

   cd ../mt


Segment mt data
***************

.. code:: sh

   sct_deepseg_sc -i mt1.nii.gz -c t2 -qc ~/qc_singleSubj


Create a mask
*************

.. code:: sh

   sct_create_mask -i mt1.nii.gz -p centerline,mt1_seg.nii.gz -size 35mm -f cylinder -o mask_mt1.nii.gz


Register mt0 on mt1
*******************

.. code:: sh

   sct_register_multimodal -i mt0.nii.gz -d mt1.nii.gz -dseg mt1_seg.nii.gz -param step=1,type=im,algo=slicereg,metric=CC -m mask_mt1.nii.gz -x spline -qc ~/qc_singleSubj


Compute MTR
***********

.. code:: sh

   sct_compute_mtr -mt0 mt0_reg.nii.gz -mt1 mt1.nii.gz

Register template to MT1
************************

.. code:: sh

   sct_register_multimodal -i $SCT_DIR/data/PAM50/template/PAM50_t2.nii.gz -iseg $SCT_DIR/data/PAM50/template/PAM50_cord.nii.gz -d mt1.nii.gz -dseg mt1_seg.nii.gz -param step=1,type=seg,algo=centermass:step=2,type=seg,algo=bsplinesyn,slicewise=1,iter=3 -m mask_mt1.nii.gz -initwarp ../t2/warp_template2anat.nii.gz -qc ~/qc_singleSubj

Warp template to MT
*******************

.. code:: sh

   mv warp_PAM50_t22mt1.nii.gz warp_template2mt.nii.gz
   sct_warp_template -d mt1.nii.gz -w warp_template2mt.nii.gz -qc ~/qc_singleSubj