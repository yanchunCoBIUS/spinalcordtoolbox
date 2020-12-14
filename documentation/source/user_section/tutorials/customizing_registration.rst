.. _customizing-registration-section:

Customizing the ``sct_register_to_template`` command
####################################################

While the default usage of ``sct_register_to_template`` is simple enough, the underlying command provides many options to adapt the registration process to your specific data and pipeline. The subsections below provide an overview of common tweaks to the command.

Because choosing the right configuration for your data can be overwhelming, SCT provides a forum where you can ask for clarification and guidance.

The ``-param`` argument
***********************

The flag -param lets you select registration parameters at each step. Below is a sample input for ``-param``:

.. code-block::

   # Note: Command has been split up for readability. Normally, you would input this with no line breaks.
   -param step=0,type=label,dof=Tx_Ty_Tz_Sz:
          step=1,type=imseg,algo=centermassrot,metric=MeanSquares,iter=10,smooth=0,gradStep=0.5,slicewise=0,smoothWarpXY=2,pca_eigenratio_th=1.6:
          step=2,type=seg,algo=bsplinesyn,metric=MeanSquares,iter=3,smooth=1,gradStep=0.5,slicewise=0,smoothWarpXY=2,pca_eigenratio_th=1.6

This long string of values defines a 3-step transformation. Each step is separated by a ``:`` character, and begins with ``step=#``, where ``#`` can be ``0, 1, 2, etc``. At each step, a distinct transformation is computed:

* **Step 0:** Straighten the spinal cord, then match the subject labels to the template labels.
* **Step 1:** Nonrigid deformation, first pass. Deals with large deformations in the spinal cord.
* **Step 2:** Nonrigid deformation, second pass. Applies fine cord shape adjustments.

Typically, step 0 is not altered. However, Steps 1 and 2 can be tweaked, and additional steps (e.g. 3, 4) can be added. Some of the common parameters to tweak include:

   .. figure:: https://raw.githubusercontent.com/spinalcordtoolbox/doc-figures/master/registration_to_template/sct_register_to_template-param-algo.png
      :align: right
      :figwidth: 40%

      Visualization of algorithms to choose from for the ``algo`` parameter of ``-param``.

* ``algo`` This is the algorithm used to compute the nonrigid deformation. Choice of algorithm depends on how coarse/fine you want your transformation to be. This depends on which step you are modifying (Step 1, step 2, step 3, etc.) as well as the nature of the spinal cord you are working with.

   - **translation**: axial translation (x-y)
   - **rigid**: transaltion + rotation about z axis
   - **affine**:
   - **b-splinesyn**: based on ants binaries
   - **syn**: (not bspline regularized)
   - **slicereg**: ANTs+Us, regualarized translation across slices that is regularized across S-I (used with segmentation for a pre-alignment cord centerline + template cord centerline -- basically alignment of the core centerline)
   - **centermassrot**: similar to slicereg, center of mass of segmentation on each slice and then align with center of mass in the template space, also does a rotation in case your subject turned neck or sometimes compression
   - **columnwise**: in case of highly compressed cords (suggested by UofT Alan Martin) it basically takes the segmentation and tries to match the compressed segmentation with the cord segmentation of the template. nonlinear deformation with much more degree of freedom than the syn based approaches

* ``type``: Carefully chose type={im, seg} based on the quality of your data, and the similarity with the template. Ideally, you would always choose type=im . However, if you find that there are artifacts of image features (e.g., no CSF/cord contrast) that could compromise the registration, then use type=seg instead. Of course, if you choose type=seg , make sure your segmentation is good (manually adjust it if it is not). By default, the sct_register_to_template relies on the segmentations only because it was found to be more robust to the existing variety of MRIs. This last step that is basically a slicewise nonlinear deformation there are a lot of algorithms that are available in SCT. Those algorithms can either be run on the image or could be run on the segmentation. Highly artifacted EPI the csf is missing on one slice, appearing on other slice, distortions, DIPS artifacts, etc. You know that your segmetnation is correct because you manually corrected it, you might want to rely on the segmentation mroe than the image in order to register your spinal cord to the template (there is a possibility to do that)
* ``metric``: Adjust metric based on type. With type=im , use metric=CC (accurate but long) or MI (fast, but requires enough voxels) With type=seg , use metric=MeanSquares.
* ``slicewise``: All transformations are constrained in Z direction, though estimation can be done slice-wise or volume-wise: -param slicewise={0, 1}

The ``-ref`` argument
*********************

The flag ``-ref`` lets you select the destination for registration: either the template (default) or the subject’s native space. The main difference is that when ``-ref template`` is selected,
the cord is straightened, whereas with ``-ref subject``, it is not.

When should you use ``-ref subject``? If your image is acquired axially with highly anisotropic resolution (e.g. 0.7x0.7x5mm), the straightening will produce through-plane interpolation errors. In that case, it is better to register the template to the subject space to avoid such inaccuracies.

The ``-ldisc`` argument
***********************

The approach described previously uses two labels at the mid-vertebral level to register the template, which is fine if you are only interested in a relatively small region (e.g. C2 —> C7). However, if your volume spans a large superior-inferior length (e.g., C2 —> L1), the linear scaling between your subject and the template might produce inaccurate vertebral level matching between C2 and L1. In that case, you might prefer to rely on all inter-vertebral discs for a more accurate registration.

Conversely, if you have a very small FOV (e.g., covering only C3/C4), you can create a unique label at disc C3/C4 (value=4) and use -ldisc for registration. In that case, a single translation (no scaling) will be performed between the template and the subject.

.. note::
   If more than 2 labels are provided, ``-ldisc`` is not compatible with ``-ref subject``. For more information, please see the help: sct_register_to_template -h