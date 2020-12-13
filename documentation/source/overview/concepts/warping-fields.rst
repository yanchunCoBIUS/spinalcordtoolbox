.. _warping-fields:

Warping fields
**************

What is a warping field?
========================

Warping fields (also known as deformation fields) are files that represent an image transformation. You can picture warping fields as a set of vector displacements, one for each ``[x, y, z]`` voxel in your 3D image. Warping fields can be used by themselves, or concatenated with other warping fields to merge multiple transformations.

.. code::

   # Apply warping fields to an image file
   sct_apply_transfo -i in.nii.gz -d out.nii.gz -w warp1.nii.gz warp2.nii.gz [...]
   # Concatenate several warping fields into single warping field
   sct_concat_transfo -d out_warp.nii.gz -w in_warp1.nii.gz in_warp2.nii.gz [...]

Warping fields are used throughout SCT:

* When straightening a spinal cord image, two warping fields are created, which define the forward and inverse transformations between the curved anatomical image and the straightened image.
* When registering a spinal cord image to a template, two warping fields are created, which define the forward and inverse transformations between the anatomical image space and the template space.

Warping field conventions
=========================

In the broader ecosystem of MRI software, there are two common conventions for representing warping fields:

* **5D composite format (``[x, y, z, t, v]``)**: Originates from Insight Toolkit (ITK), so it also referred to as the ITK format. This of warping field is used by SCT and Advanced Normalization Tools (ANTs). This format is defined by in the "Vector-Valued Datasets" section of the `NIFTI1 Specification <https://nifti.nimh.nih.gov/pub/dist/src/niftilib/nifti1.h>`_.
* **4D vector format (``[x, y, z, v]``)**: Used by the FMRIB Software Library (FSL) and Statistical Parametric Mapping (SPM) software packages. This format is defined in the "Deformation model" section of the `FSLWiki <https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FNIRT/UserGuide#Deformation_model>`_

For both formats, the ``v`` axis will be of size 3, with each index representing one component of the displacement vector. So, ``v=0`` refers to the ``x`` displacement for each ``[x, y, z]`` coordinate, ``v=1`` refers to the ``y`` displacement, and ``v=2`` refers to the ``z`` displacement.

Compatibility
=============

SCT generates warping fields in the 5D composite ITK format. This format is incompatible with non-ITK software such as FSL or SPM. But, a conversion between the two formats can be performed using ``sct_image``:

.. code::

   # Split ITK warping field into 3 separate volumes containing X, Y and Z displacements
   sct_image -i warp_itk.nii.gz -mcs -o warp_vec.nii.gz
   # Merge into FSL-compatible warping field
   sct_image -i warp_vec_*.nii.gz -concat t -o warp_fsl.nii.gz

.. warning:: This ITK/FSL warping field conversion only works if the source and destination files are in the same voxel space. If not, there is currently no easy way to do this conversion. The issue is being investigated (more details at: https://github.com/neuropoly/spinalcordtoolbox/issues/2525).