# Templates and atlases

In the following, we describe how the outcomes of analyses that derive from or
produce [templates and atlases](../common-principles.md#definitions) are organized
as BIDS-Derivatives.

These outcomes typically involve quantitative maps, feature maps, parcellations,
segmentations, and other knowledge annotations such as landmarks in
individual- or group-level spaces.

In BIDS, a template is considered any aggregation of continuous- or discrete- 
valued data.  Some templates also serve as the authoritative definition of a
space and are used to bring other imaging data into alignment so that it can be 
aggregated.

In BIDS, an atlas is considered a collection of related templates.

For templates that do not aggregate data over more than one subject, the
organization follows the standards for BIDS raw and derivatives.  The following
entities MAY be employed to specify template-derived results:

-    [`tpl-<label>`](../glossary.md#template-entities) is REQUIRED to specify derivatives defining
     a [template](../common-principles.md).
-    [`space-<label>`](../glossary.md#space-entities) is REQUIRED to disambiguate derivatives defined with
     respect to different [coordinate systems](../appendices/coordinate-systems.md), following the general
     BIDS-Derivatives specifications.
-    [`atlas-<label>`](../glossary.md#atlas-entities) MAY be used to group several templates together
-    [`seg-<label>`](../glossary.md#segmentation-entities) is REQUIRED to disambiguate dicrete-valued
     templates with multiple realizations (for instance, segmentations and parcellations created with different criteria) -    [`scale-<label>`](../glossary.md#scale-entities) is REQUIRED to disambiguate different 'scales' or voxel resoutions
     when a template has multiple levels of detail.

The filename pattern for subject-level derivatives follows the general BIDS-Derivatives pattern:

```Text
<pipeline_name>/
    sub-<label>/
        <datatype>/
            <source_entities>[_space-<space>][_cohort-<label>][seg-<label>][_scale-<label>][_res-<label>][_den-<label>][_desc-<label>]_<suffix>.<extension>
```

[`atlas-<label>`](../glossary.md#atlas-entities), [`seg-<label>`](../glossary.md#segmentation-entities),
and [`scale-<label>`](../glossary.md#scale-entities) are discussed later in section
[Filenames of derivatives with atlases in their provenance](#filenames-of-derivatives-with-atlases-in-their-provenance).

For derivatives of template- and altas-generating pipelines, which typically aggregate
several sessions and/or subjects, the derivatives-specific
[`tpl-<label>` entity](../glossary.md#template-entities) can be thought of as the
group-level substitute to the usage of [`sub-<label>`](../glossary.md#subject-entities)
at the subject-level, and MAY be employed as follows:

```Text
<pipeline_name>/
    tpl-<label>/
        [cohort-<label>/]
           [<datatype>/]
               tpl-<label>_<source_entities>[_cohort-<label>][_atlas-<label>][seg-<label>][_scale-<label>][_res-<label>][_den-<label>][_desc-<label>]_<suffix>.<extension>
```

where [`suffix`](../glossary.md#suffix-common_principles) will generally be existing BIDS raw modalities
(such as `T1w`) or `dseg`, `probseg`, or `mask` to encode dicrete-valued knowledge.
In terms of [`extension`](../glossary.md#extension-common_principles), `nii[.gz]`, `dscalar.nii[.gz]`,
`dlabel.nii[.gz]`, `label.gii[.gz]`, `tsv`, or `json`.
Please note that the [`<datatype>/` directory](../glossary.md#data_type-common_principles) is RECOMMENDED.
The [`<datatype>/` directory](../glossary.md#data_type-common_principles) MAY be omitted in the case
only one data type (such as `anat/`) is stored under the `tpl-<label>` directory.
The [`cohort-<label>` directory and entity](../glossary.md#cohort-entities) MUST be specified for templates
with several cohorts.

Both subject-level and group-level results can coexist in a single pipeline directory:

```Text
<pipeline_name>/
    sub-<label>/
        <datatype>/
            <source_entities>[_space-<space>][_cohort-<label>][_atlas-<label>][seg-<label>][_scale-<label>][_res-<label>][_den-<label>][_desc-<label>]_<suffix>.<extension>
    tpl-<label>/
        [cohort-<label>/]
           [<datatype>/]
               <source_entities>[_cohort-<label>][_space-<space>][_atlas-<label>][seg-<label>][_scale-<label>][_res-<label>][_den-<label>][_desc-<label>]_<suffix>.<extension>
```

## Single-subject templates

Early digital templates such as MNI's
'[Colin 27 Average Brain, Stereotaxic Registration Model](https://www.mcgill.ca/bic/software/tools-data-analysis/anatomical-mri/atlases/colin-27)'
([Holmes et al., 1998](https://doi.org/10.1097/00004728-199803000-00032)) were built by examining single individuals.
For example, the outputs of the pipeline that generated 'Colin27' would have been organized as follows:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "colin27-pipeline": {
      "sub-01": {
         "anat": {
            "sub-01_label-brain_mask.nii.gz": "",
            "sub-01_label-head_mask.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
            "sub-01_T1w.json": "",
         },
      },
   }
})
}}

In the presence of conflicting files, for example, when there are several resolutions,
additional entities MUST be specified:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "colin27-pipeline": {
      "sub-01": {
         "anat": {
            "sub-01_res-1_label-brain_mask.nii.gz": "",
            "sub-01_res-1_label-head_mask.nii.gz": "",
            "sub-01_res-1_T1w.nii.gz": "",
            "sub-01_res-1_T1w.json": "",
            "sub-01_res-2_T1w.nii.gz": "",
            "sub-01_res-2_T1w.json": "",
         },
      },
   }
})
}}

## Multi-subject templates and deriving an existing template

Atlasing multiple individual brains is a higher-than-first-level analysis,
as it requires first generating derivatives for the individuals (for example,
a transformation to align them into a standardized space) and later aggregate
and distill the sample-pooled knowledge and feature maps.

**Multi-subject templates**.
While at the subject level analysis it is the individual brain that establishes
stereotaxy.
At higher-than-first-level stereotaxy is supported by templates, which are
encoded through the [`tpl-<label>` entity](../glossary.md#template-entities).
For the pipeline that generated the MNI152NLin2009cAsym, the outputs would look
like the following example:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mni152nlin2009casym-pipeline": {
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MNI152NLin2009cAsym_res-1_label-brain_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-eye_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-face_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-head_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-CSF_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-GM_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_label-WM_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_T1w.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-1_T1w.json": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-brain_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-eye_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-face_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-head_mask.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-CSF_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-GM_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_label-WM_probseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_T1w.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_res-2_T1w.json": "",
         },
      },
   }
})
}}

**Multi-cohort templates.**
In the case that the template-generating pipeline derives
several cohorts, the file structure must employ the
[`cohort-<label>` directory and entity](../glossary.md#cohort-entities).

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mnipediatricasym-pipeline": {
      "tpl-MNIPediatricAsym": {
         "cohort-1": {
            "anat": {
               "tpl-MNIPediatricAsym_cohort-1_res-1_PD.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_T1w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_T2w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_desc-brain_mask.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_label-CSF_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_label-GM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-1_label-WM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_PD.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_T1w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_T2w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_desc-brain_mask.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_label-CSF_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_label-GM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-1_res-2_label-WM_probseg.nii.gz": "",
            },
         },
         "...": "",
         "cohort-6": {
            "anat": {
               "tpl-MNIPediatricAsym_cohort-6_res-1_PD.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_T1w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_T2w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_desc-brain_mask.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_label-CSF_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_label-GM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-1_label-WM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_PD.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_T1w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_T2w.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_desc-brain_mask.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_label-CSF_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_label-GM_probseg.nii.gz": "",
               "tpl-MNIPediatricAsym_cohort-6_res-2_label-WM_probseg.nii.gz": "",
            },
         },
      },
   }
})
}}

**Storing spatial transforms.**
Since multi-subject templates involve the spatial normalization of
subjects by means of image registration processes, it is RECOMMENDED to store
the resulting transforms for each of the subjects employed to create the
output.
Please note that the specification for spatial transforms (BEP 014) is currently
under development, and therefore, the specification of transforms files may
change in the future.
As these are subject-level results, they follow the standard derivatives conventions
with a `sub-<label>` directory to house these derivatives:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mni152nlin2009casym-pipeline": {
      "sub-001": {
         "anat": {
            "sub-001_from-T1w_to-MNI152NLin2009cAsym_mode-image_xfm.h5": "",
            "sub-001_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-001_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-152": {
         "anat": {
            "sub-152_from-T1w_to-MNI152NLin2009cAsym_mode-image_xfm.h5": "",
            "sub-152_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-152_T1w.nii.gz": "",
         },
      },
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MNI152NLin2009cAsym_res-1_label-brain_mask.nii.gz": "",
            "...": "",
            "tpl-MNI152NLin2009cAsym_res-2_T1w.json": "",
         },
      },
   }
})
}}

**Using `atlas-` to group related templates.**

The following example shows how 'Colin27' could have encoded the Automated Anatomical Labeling (AAL)
atlas ([Tzourio-Mazoyer et al., 2002](https://doi.org/10.1006/nimg.2001.0978)), which was originally
defined on the Colin27 space:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "colin27-pipeline": {
      "sub-01": {
         "anat": {
            "sub-01_atlas-AAL_dseg.json": "",
            "sub-01_atlas-AAL_dseg.nii.gz": "",
            "sub-01_atlas-AAL_dseg.tsv": "",
            "sub-01_atlas-AAL_probseg.nii.gz": "",
            "sub-01_atlas-AAL_seg-brain_mask.nii.gz": "",
            "sub-01_atlas-AAL_seg-head_mask.nii.gz": "",
            "sub-01_atlas-AAL_T1w.nii.gz": "",
            "sub-01_atlas-AAL_T1w.json": "",
         },
      },
   }
})
}}

As [the authors of 'Colin27' indicate](https://www.mcgill.ca/bic/software/tools-data-analysis/anatomical-mri/atlases/colin-27),
it is aligned with MNI305:

> In 1998, a new atlas with much higher definition than MNI305s was created at the MNI.
> One individual (CJH) was scanned 27 times and the images linearly registered to create
> an average with high SNR and structure definition (Holmes et al., 1998).
> This average was linearly registered to the average 305.
> Ironically, this dataset was not originally intended for use as a stereotaxic template
> but as the sub-strate for an ROI parcellation scheme to be used with
> ANIMAL non-linear spatial normalization (Collins et al., 1995),
> i.e. it was intended for the purpose of segmentation, NOT stereotaxy.
> As a single brain atlas, it did not capture anatomical variability and was, to some degree,
> a reversion to the Talairach approach.
>
> However, the high definition proved too attractive to the community and,
> after non-linear mapping to fit the MNI305 space, it has been adopted
> by many groups as a stereotaxic template.

Therefore, this pipeline potentially could have produced outputs in the realigned T1w space
before alignment to the MNI305 template.
To disambiguate in this case, we employ the [`space-<label>` entity](../glossary.md#space-entities):

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "colin27-pipeline": {
      "sub-01": {
         "anat": {
            "sub-01_space-MNI305_atlas-AAL_dseg.json": "",
            "sub-01_space-MNI305_atlas-AAL_dseg.nii.gz": "",
            "sub-01_space-MNI305_atlas-AAL_dseg.tsv": "",
            "sub-01_space-MNI305_atlas-AAL_probseg.nii.gz": "",
            "sub-01_space-MNI305_atlas-AAL_seg-brain_mask.nii.gz": "",
            "sub-01_space-MNI305_atlas-AAL_seg-head_mask.nii.gz": "",
            "sub-01_space-MNI305_atlas-AAL_T1w.nii.gz": "",
            "sub-01_space-MNI305_atlas-AAL_T1w.json": "",
            "sub-01_space-T1w_atlas-AAL_label-brain_mask.nii.gz": "",
            "sub-01_space-T1w_atlas-AAL_label-head_mask.nii.gz": "",
            "sub-01_space-T1w_atlas-AAL_T1w.nii.gz": "",
            "sub-01_space-T1w_atlas-AAL_T1w.json": "",
         },
      },
   }
})
}}

For example, the [PS13 templates](https://doi.org/10.18112/openneuro.ds004401.v1.3.0),
a molecular imaging brain template of Cyclooxygenase-1 (PET),
was generated in two standard spaces: `MNI152Lin` and `fsaverage`.
Here, the `atlas-` entity is not used, since `tpl-` along with other entities is sufficient to disambiguate:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "ps13-pipeline": {
      "tpl-ps13": {
         "pet": {
            "tpl-ps13_space-fsaverage_desc-nopvc_dseg.nii.gz": "",
            "tpl-ps13_space-fsaverage_desc-pvc_dseg.nii.gz": "",
            "tpl-ps13_space-fsaverage_dseg.json": "",
            "tpl-ps13_space-fsaverage_dseg.tsv": "",
            "tpl-ps13_space-fsaverage_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_desc-nopvc_pet.nii.gz": "",
            "tpl-ps13_space-fsaverage_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage_desc-pvc_pet.nii.gz": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_desc-nopvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage-L_den-164k_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage-L_den-164k_desc-pvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_stat-std_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_stat-std_desc-nopvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_stat-std_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-L_den-164k_stat-std_desc-pvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_desc-nopvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_desc-pvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_stat-std_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_stat-std_desc-nopvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_stat-std_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage_hemi-R_den-164k_stat-std_desc-pvc_pet.shape.gii": "",
            "tpl-ps13_space-fsaverage_stat-std_desc-nopvc_pet.json": "",
            "tpl-ps13_space-fsaverage_stat-std_desc-nopvc_pet.nii.gz": "",
            "tpl-ps13_space-fsaverage_stat-std_desc-pvc_pet.json": "",
            "tpl-ps13_space-fsaverage_stat-std_desc-pvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_desc-nopvc_dseg.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_desc-pvc_dseg.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_dseg.json": "",
            "tpl-ps13_space-MNI152Lin_dseg.tsv": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_desc-spmvbmNopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_desc-spmvbmNopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_desc-spmvbmPvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_desc-spmvbmPvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_stat-std_desc-spmvbmNopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_stat-std_desc-spmvbmNopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_stat-std_desc-spmvbmPvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-1p5_stat-std_desc-spmvbmPvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-fnirtNopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-fnirtNopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-fnirtPvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-fnirtPvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-nopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-nopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-pvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_desc-pvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-fnirtNopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-fnirtNopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-fnirtPvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-fnirtPvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-nopvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-nopvc_pet.nii.gz": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-pvc_pet.json": "",
            "tpl-ps13_space-MNI152Lin_res-2_stat-std_desc-pvc_pet.nii.gz": "",
         },
      },
   }
})
}}

###----####----

**Producing a new template AND atlas.**
Segmentations are often performed with reference to a *custom* standard space.
In this case, a feature template map is generated from all the participant(s)
in the study, and the segmentation's artifacts are produced with reference to that
template.

Either by generating the template space with aligning to a pre-existing template,
or by estimating a transform between templates by means of image registration,
a new template definition MUST be employed if the new template generates
a new [*space*](../common-principles.md#definitions).
For example, let's imagine that PS13 first generated a template nuclear imaging
map and after that, a corresponding segmentation was defined.
In that case, the [`seg-<label>`] SHOULD be use to specify these segmentaitons:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "ps13-pipeline": {
      "tpl-PS13": {
         "pet": {
            "tpl-PS13_desc-nopvc_dseg.nii.gz": "",
            "tpl-PS13_desc-pvc_dseg.nii.gz": "",
            "tpl-PS13_dseg.json": "",
            "tpl-PS13_dseg.tsv": "",
            "tpl-PS13_stat-std_desc-fnirtNopvc_pet.json": "",
            "tpl-PS13_stat-std_desc-fnirtNopvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-fnirtPvc_pet.json": "",
            "tpl-PS13_stat-std_desc-fnirtPvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-nopvc_pet.json": "",
            "tpl-PS13_stat-std_desc-nopvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-pvc_pet.json": "",
            "tpl-PS13_stat-std_desc-pvc_pet.nii.gz": "",
         },
      },
   }
})
}}

Let's complete the above example by adding two new segmentations to the existing
template:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "ps13-with-atlases-pipeline": {
      "tpl-PS13": {
         "pet": {
            "tpl-PS13_seg-Economo1916_desc-nopvc_dseg.nii.gz": "",
            "tpl-PS13_seg-Economo1916_desc-pvc_dseg.nii.gz": "",
            "tpl-PS13_seg-Economo1916_dseg.json": "",
            "tpl-PS13_seg-Economo1916_dseg.tsv": "",
            "tpl-PS13_seg-RamonCajal1908_desc-nopvc_dseg.nii.gz": "",
            "tpl-PS13_seg-RamonCajal1908_desc-pvc_dseg.nii.gz": "",
            "tpl-PS13_seg-RamonCajal1908_dseg.json": "",
            "tpl-PS13_seg-RamonCajal1908_dseg.tsv": "",
            "tpl-PS13_desc-nopvc_dseg.nii.gz": "",
            "tpl-PS13_desc-pvc_dseg.nii.gz": "",
            "tpl-PS13_dseg.json": "",
            "tpl-PS13_dseg.tsv": "",
            "tpl-PS13_stat-std_desc-fnirtNopvc_pet.json": "",
            "tpl-PS13_stat-std_desc-fnirtNopvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-fnirtPvc_pet.json": "",
            "tpl-PS13_stat-std_desc-fnirtPvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-nopvc_pet.json": "",
            "tpl-PS13_stat-std_desc-nopvc_pet.nii.gz": "",
            "tpl-PS13_stat-std_desc-pvc_pet.json": "",
            "tpl-PS13_stat-std_desc-pvc_pet.nii.gz": "",
         },
      },
   }
})
}}

where the `seg-RamonCajal1908` and `seg-Economo1916` hypothetically define
two different atlases (please note that, often, atlases are named after
the first author and indicating a year of a reference communication).
The original *default* or *implicit* atlas' artifacts such as
the `tpl-PS13_desc-nopvc_dseg.nii.gz` segmentation,
which were originally generated with the PS13 template,
MAY take an [`atlas-<label>`](../glossary.md#atlas-entities) if they
need to be differentiated from the original template and atlas dataset.
However, it is RECOMMENDED that these *default* or *implicit* atlases employed
in the *custom* space they were generated only define
[`atlas-<label>`](../glossary.md#atlas-entities)
if it is necessary to disambiguate two or more atlases.

A further example of these template-and-atlas specifications
is the *Spatially Unbiased Infratentorial Template (SUIT)*
([Diedrichsen, 2006](https://doi.org/10.1016/j.neuroimage.2006.05.056)):

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "suit-pipeline": {
      "tpl-SUIT": {
         "anat": {
            "CHANGES": "",
            "LICENSE": "",
            "README.md": "",
            "dataset_description.json": "",
            "tpl-SUIT_T1w.nii.gz": "",
            "tpl-SUIT_seg-Buckner2011_dseg.json": "",
            "tpl-SUIT_seg-Buckner2011n17_dseg.label.gii": "",
            "tpl-SUIT_seg-Buckner2011n17_dseg.nii.gz": "",
            "tpl-SUIT_seg-Buckner2011n17_dseg.tsv": "",
            "tpl-SUIT_seg-Buckner2011n17_stat-confidence_probseg.nii.gz": "",
            "tpl-SUIT_seg-Buckner2011n7_dseg.label.gii": "",
            "tpl-SUIT_seg-Buckner2011n7_dseg.nii.gz": "",
            "tpl-SUIT_seg-Buckner2011n7_dseg.tsv": "",
            "tpl-SUIT_seg-Buckner2011n7_stat-confidence_probseg.nii.gz": "",
            "tpl-SUIT_seg-Diedrichsen2009_dseg.json": "",
            "tpl-SUIT_seg-Diedrichsen2009_dseg.label.gii": "",
            "tpl-SUIT_seg-Diedrichsen2009_dseg.nii.gz": "",
            "tpl-SUIT_seg-Diedrichsen2009_dseg.tsv": "",
            "tpl-SUIT_seg-Diedrichsen2009_probseg.nii.gz": "",
            "tpl-SUIT_flat.surf.gii": "",
            "tpl-SUIT_sulc.shape.gii": "",
         },
      },
   }
})
}}

In this case, a new T1w template of the cerebellum was created, and two different
segmentations (`Diedrichsen2009`, and `Buckner2011`) were generated with respect to
the T1w template.

**Deriving from an existing template**.
For example, the MIAL67ThalamicNuclei
([Najdenovska et al., 2018](https://doi.org/10.1038/sdata.2018.270))
pipeline could display the following structure:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mial67thalamicnuclei-pipeline": {
      "sub-01": {
         "anat": {
            "sub-01_seg-ThalamicNuclei_dseg.json": "",
            "sub-01_seg-ThalamicNuclei_dseg.tsv": "",
            "sub-01_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-67": {
         "anat": {
            "sub-67_seg-ThalamicNuclei_dseg.json": "",
            "sub-67_seg-ThalamicNuclei_dseg.tsv": "",
            "sub-67_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-67_T1w.nii.gz": "",
         },
      },
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_dseg.json": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_dseg.tsv": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_res-1_dseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_res-1_probseg.nii.gz": "",
         },
      },
   }
})
}}

where the derivatives of anatomical processing of the 67 subjects that were
employed to generate the segmentation coexist with the template structure.

The inheritance principle applies uniformly, allowing the segmentation
metadata be stored only once at the root of the pipeline directory and
apply to all the individual subject segmentations:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mial67thalamicnuclei-pipeline": {
      "seg-ThalamicNuclei_dseg.json": "",
      "seg-ThalamicNuclei_dseg.tsv": "",
      "sub-01": {
         "anat": {
            "sub-01_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-67": {
         "anat": {
            "sub-67_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-67_T1w.nii.gz": "",
         },
      },
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_dseg.json": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_dseg.tsv": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_res-1_dseg.nii.gz": "",
            "tpl-MNI152NLin2009cAsym_seg-MIAL67ThalamicNuclei_res-1_probseg.nii.gz": "",
         },
      },
   }
})
}}

This directory structure can be generally applied when the atlas is derived into several
template spaces, for example:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mial67thalamicnuclei-pipeline": {
      "tpl-MIAL67ThalamicNuclei_dseg.json": "",
      "tpl-MIAL67ThalamicNuclei_dseg.tsv": "",
      "seg-ThalamicNuclei_dseg.json": "",
      "seg-ThalamicNuclei_dseg.tsv": "",
      "sub-01": {
         "anat": {
            "sub-01_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-01_space-MNI152NLin6Asym_T1w.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-67": {
         "anat": {
            "sub-67_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-67_space-MNI152NLin6Asym_T1w.nii.gz": "",
            "sub-67_T1w.nii.gz": "",
         },
      },
      "space-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_dseg.nii.gz": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_probseg.nii.gz": "",
         },
      },
      "space-MNI152NLin6Asym": {
         "anat": {
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin6Asym_res-1_dseg.nii.gz": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin6Asym_res-1_probseg.nii.gz": "",
         },
      },
   }
})
}}

In the case the pipeline generated segmentations of the original subjects in
their native T1w space (for example, to compare with the original segmentation given by
`seg-ThalamicNuclei`), the above example translates into:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mial67thalamicnuclei-pipeline": {
      "tpl-MIAL67ThalamicNuclei_dseg.json": "",
      "tpl-MIAL67ThalamicNuclei_dseg.tsv": "",
      "seg-ThalamicNuclei_dseg.json": "",
      "seg-ThalamicNuclei_dseg.tsv": "",
      "sub-01": {
         "anat": {
            "sub-01_seg-MIAL67ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-67": {
         "anat": {
            "sub-67_seg-MIAL67ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-67_T1w.nii.gz": "",
         },
      },
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_dseg.nii.gz": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_probseg.nii.gz": "",
         },
      },
   }
})
}}

Without any loss in generality, we can store subjects' spatially normalizing
transforms, as well as transforms between template spaces:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
   "mial67thalamicnuclei-pipeline": {
      "tpl-MIAL67ThalamicNuclei_dseg.json": "",
      "tpl-MIAL67ThalamicNuclei_dseg.tsv": "",
      "seg-ThalamicNuclei_dseg.json": "",
      "seg-ThalamicNuclei_dseg.tsv": "",
      "sub-01": {
         "anat": {
            "sub-01_from-T1w_to-MNI152NLin2009cAsym_mode-image_xfm.h5": "",
            "sub-01_from-T1w_to-MNI152NLin6Asym_mode-image_xfm.h5": "",
            "sub-01_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-01_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-01_space-MNI152NLin6Asym_T1w.nii.gz": "",
            "sub-01_T1w.nii.gz": "",
         },
      },
      "...": "",
      "sub-67": {
         "anat": {
            "sub-67_from-T1w_to-MNI152NLin2009cAsym_mode-image_xfm.h5": "",
            "sub-67_from-T1w_to-MNI152NLin6Asym_mode-image_xfm.h5": "",
            "sub-67_seg-ThalamicNuclei_dseg.nii.gz": "",
            "sub-67_space-MNI152NLin2009cAsym_T1w.nii.gz": "",
            "sub-67_space-MNI152NLin6Asym_T1w.nii.gz": "",
            "sub-67_T1w.nii.gz": "",
         },
      },
      "tpl-MNI152NLin2009cAsym": {
         "anat": {
            "tpl-MNI152NLin2009cAsym_from-MNI152NLin6Asym_mode-image_xfm.h5": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_dseg.nii.gz": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin2009cAsym_res-1_probseg.nii.gz": "",
         },
      },
      "tpl-MNI152NLin6Asym": {
         "anat": {
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin6Asym_res-1_dseg.nii.gz": "",
            "tpl-MIAL67ThalamicNuclei_space-MNI152NLin6Asym_res-1_probseg.nii.gz": "",
         },
      },
   }
})
}}

## Tabular data

The `[probseg|dseg|mask].tsv` file indexes and labels each node/parcel/region within the atlas.
This file resembles the typical Look Up Table (LUT) often shared with atlases.
This file will be essential for downstream workflows that generate matrices or other derived files within which node/parcel/region information is required,
as the index/label fields will be used to reference the original anatomy the index/labels are derived from.
Additional fields can be added with their respective definition/description in the sidecar json file.

This is described in the [imaging derivatives](./imaging.md#common-image-derived-labels) section of the BIDS specification

Example:
```Text
index	label	network_label	hemisphere
1	Heschl's Gyrus	Somatomotor	left
2	Heschl's Gyrus	Somatomotor	right
```

## Template metadata

The `tpl-<label>_description.json` file provides metadata to uniquely identify, describe and characterize the template, as well as give proper attribution to the creators.
Additionally, SpatialReference serves the important purpose of unambiguously identifying the space the template is in.

<!-- This block generates a metadata table.
These tables are defined in
  src/schema/rules/sidecars
The definitions of the fields specified in these tables may be found in
  src/schema/objects/metadata.yaml
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_sidecar_table([
       "derivatives.common_derivatives.TemplateDescription",
   ]) }}

Example:

```JSON
{
  "Name": "FSL's MNI ICBM 152 non-linear 6th Generation Asymmetric Average Brain Stereotaxic Registration Model",
  "Authors": [
    "David Kennedy",
    "Christian Haselgrove",
    "Bruce Fischl",
    "Janis Breeze",
    "Jean Frazie",
    "Larry Seidman",
    "Jill Goldstein"
  ],
  "BIDSVersion": "1.1.0",
  "Curators": "FSL team",
  "SpatialReference": "https://templateflow.s3.amazonaws.com/tpl-MNI152NLin6Asym_res-02_T1w.nii.gz",
  "Resolution": "Matched with original template resolution (2x2x3 mm^3)",
  "License": "See LICENSE file",
  "RRID": "SCR_002823",
  "ReferencesAndLinks": [
    "https://doi.org/10.1016/j.neuroimage.2012.01.024",
    "https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Atlases"
  ],
  "Species": "Human"
}
```
