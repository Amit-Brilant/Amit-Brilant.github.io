---
title: Ovarian Carcinoma Diagnosis from Transvaginal Ultrasound
summary: M.Sc. thesis. A three-stage computer-vision pipeline that triages transvaginal ultrasound, localises the lesion, and classifies it as benign or malignant by fusing CNN embeddings with morphological features. 0.952 accuracy and 0.964 AUC at the patient level. Code and dataset are public.
date: 2026-09-01

authors:
  - me

tags:
  - Deep Learning
  - Computer Vision
  - Medical Imaging
  - PyTorch
  - Object Detection
  - Explainable AI

featured: true

image:
  caption: 'The three-stage pipeline'
  focal_point: Smart

layout: showcase

showcase_title: Detection and Diagnosis of Ovarian Carcinoma from Complex Cysts Using Computer Vision and Ultrasound Imaging
showcase_authors: Amit Brilant
showcase_affiliation: Ben-Gurion University of the Negev &nbsp;·&nbsp; in collaboration with Meir Medical Center
showcase_venue: M.Sc. Thesis &nbsp;·&nbsp; Advisors Prof. Ofer Hadar and Prof. Oshrit Hoffer &nbsp;·&nbsp; September 2026
showcase_teaser: pipeline.png
showcase_teaser_alt: Three-stage pipeline from ultrasound image through triage, lesion detection and benign versus malignant classification
showcase_teaser_caption: A patient is called by the majority vote of their frames.

showcase_buttons:
  - name: Code
    url: https://github.com/Amit-Brilant/ovarian-cancer-tvus-pipeline
    icon: github
  - name: Dataset
    url: https://doi.org/10.5281/zenodo.20383744
    icon: link

highlights:
  - value: 0.964
    title: Patient-level AUC
    text: With 0.952 accuracy on the held-out test cohort for the benign versus malignant decision, the stage that carries the clinical weight.
  - value: 3
    title: Stages, each a different problem
    text: Triage is classification, localisation is detection, and the malignancy call needs shape, not just texture. One model for all three would have compromised each.
  - value: 11
    title: Descriptors from SAM masks
    text: Segment Anything turns each detected box into a mask, and the mask into geometry. Fusing that geometry with CNN embeddings is what separates this from a plain image classifier.
  - value: "1,082"
    title: Images from 260 patients, released
    text: De-identified images, annotations, clinical variables, partitions and trained weights, published under CC BY 4.0.

acknowledgements:
  - <strong>Acknowledgements.</strong> M.Sc. thesis at Ben-Gurion University of the Negev, supervised by Prof. Ofer Hadar and Prof. Oshrit Hoffer. Clinical data from Meir Medical Center.
  - This work is for research purposes only. It is not a clinical device.
---

## Abstract

Ovarian carcinoma is diagnosed late because early disease is quiet and the imaging is
ambiguous. Transvaginal ultrasound is the first-line examination: it is cheap and it is
everywhere, but reading it is difficult and the reading varies between examiners. Structured
systems such as O-RADS reduce that variation without removing it, because an expert still has
to record what the lesion looks like.

This thesis builds a three-stage pipeline over transvaginal ultrasound in postmenopausal women
that reads the examination the way a gynaecologist does: triage the frame, find the lesion,
then characterise it. On the held-out test cohort the third stage reaches **0.952 accuracy and
0.964 AUC at the patient level**.

## Why three stages

A single end-to-end classifier is the obvious first attempt and the wrong one here. The three
questions are genuinely different problems.

**Triage** is a whole-image classification question. An EfficientNet-B7 backbone, trained with
a clinical CAM-loss, separates normal from pathological frames.

**Localisation** is a detection question. The malignancy decision depends on the lesion, not on
the surrounding tissue or the acoustic shadow behind it. A YOLOv8m detector finds the lesion and
hands the region of interest to the next stage. Faster R-CNN and YOLO11 were trained as
alternatives and compared.

**Characterisation** is where texture alone stops being enough. Clinicians read solidity, wall
irregularity and shape. So this stage fuses two representations: EfficientNet-B7 embeddings of
the cropped region, and eleven descriptors computed from a segmentation mask. Six fusion
configurations were trained and compared.

## Where the masks come from

![From image to detector box to SAM mask](roi_sam_pipeline.png)

The dataset has bounding boxes, not masks, and hand-segmenting lesions at scale was not
realistic. Segment Anything (SAM, ViT-H) closes that gap: each detector box is used as a
prompt, SAM returns a mask, and the descriptors are computed from it. No reader is involved at
inference time, and the masks regenerate deterministically from the released boxes.

This is the part of the pipeline I would defend first. It turns a foundation model into a
feature extractor for a problem it was never trained on, and gives the classifier the exact
properties clinicians actually use.

## Results

| Stage | Task | Model | Result |
|---|---|---|---|
| I | Normal vs pathological | EfficientNet-B7, clinical CAM-loss | Accuracy / AUC 0.974 / 0.997 (test, patient level) |
| II | Lesion localisation | YOLOv8m | mAP@0.5 = 0.746 (validation) |
| III | Benign vs malignant, Configuration 2 | EfficientNet-B7 on the crop | Accuracy / AUC 0.905 / 0.945 (test, patient level) |
| III | Benign vs malignant, Configuration 6 | + 11 SAM-derived descriptors | **Accuracy / AUC 0.952 / 0.964** (test, patient level) |

![Patient-level accuracy of readers, a clinical baseline and two pipeline configurations](results.png)

Two readings of this chart matter more than the headline. The descriptors add a real margin
over the image-only model. And a simple clinical baseline, CA-125 with age, is already strong
on this cohort, which is exactly why the comparison is shown rather than hidden. The test
cohort is 21 patients, so one patient moves accuracy by 4.8 points and most of these
differences are not statistically significant.

## Explaining the decision

- **Grad-CAM** on the last convolutional layer of the Stage III backbone, showing which region
  of the lesion drove the call
- **SHAP** and PCA over an XGBoost clinical pre-classifier, showing which clinical variables
  carry the signal independently of the imaging
- Bootstrap confidence intervals and paired tests rather than single point estimates

## The released data

![Examples from the released dataset](dataset_samples.png)

1,082 transvaginal B-mode images from 260 postmenopausal women, 2012 to 2023, de-identified and
released with the annotations, clinical variables, descriptors, partitions, per-patient
predictions and trained weights. The code reimplements every experiment as one package, with
unit tests, so the pipeline runs end to end from the public release.

## Limitations

A medical imaging result without its boundary conditions is not a result.

- Single-centre cohort. Multi-centre external validation is the highest-priority follow-up
- 21 test patients for Stage III
- Stage I separation may partly reflect acquisition differences between the groups, and should
  not be read as a screening result
- Postmenopausal women only. No physical-scale calibration, so geometric descriptors are
  relative within this cohort. Doppler vascularity is not modelled

## Stack

PyTorch, EfficientNet-B7, YOLOv8, Segment Anything (SAM), XGBoost, SHAP, Grad-CAM,
scikit-learn, pytest, Python 3.10.
