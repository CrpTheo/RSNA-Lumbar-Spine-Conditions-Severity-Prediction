# RSNA 2024 Lumbar Spine Degenerative Classification
## Overview

This project was developed for the RSNA 2024 Lumbar Spine Degenerative Classification competition, which explored whether artificial intelligence can aid in the detection and classification of degenerative spine conditions using lumbar spine MR images. The challenge focused on five lumbar spine conditions across intervertebral disc levels: Left Neural Foraminal Narrowing, Right Neural Foraminal Narrowing, Left Subarticular Stenosis, Right Subarticular Stenosis, and Spinal Canal Stenosis.

The goal was to classify these conditions at varying severity levels (Normal/Mild, Moderate, or Severe), using MR images across multiple axial and sagittal planes.

## Competition Context

Low back pain, a leading cause of disability, affects hundreds of millions of people worldwide. Often, it is associated with conditions like spinal stenosis, subarticular recess narrowing, and neural foramen compression. MRI imaging provides radiologists with the details necessary for diagnosis, but automation through AI could help standardize and expedite this process. This competition aimed to enhance AI models to aid in the classification of these conditions using MRI data.

## Key Approach

This solution integrates a Transformer-based architecture combined with CNNs for ROI localization. The primary challenge was handling the variability in MRI slice alignment, especially between 2D sagittal and axial planes, which required developing a flexible and robust architecture.

## Key Steps and Solution Design

**1. MRI Visualization and Problem Understanding**

Using the initial guidance from the Anatomy Image Visualization Overview notebook, I focused on 2D Sagittal T1 for foraminal analysis, 2D Axial T2 for subarticular conditions, and both sagittal and axial views for spinal canal conditions. The challenge here was aligning these slices across different views, often without metadata to guide precise alignment.

**2. 2D UNet for ROI Localization**

I employed UNet architectures to segment key regions across different MRI planes:

- Sagittal T1: For foraminal region segmentation
- Sagittal T2: For spinal segmentation
- Axial T2: For subarticular and spinal segmentation

These segmentations allowed me to pinpoint specific vertebral levels and assign slices appropriately. Special thanks to @hengck23 for the methodology on 2D to 3D projection of DICOM data.

**3. Data Augmentation and Imputation**

In order to expand the available data:

I imputed coordinates for unlabeled slices based on adjacent slices and used axial T2 flips to duplicate usable data.
Data augmentation through slice imputation expanded the dataset significantly, improving model robustness.

**4. CNNs for Crop Localization**

Initially, I used full MRI slices to feed into the Transformer, which led to some performance issues. After looking at everyone's work and further experimentation, I developed classical CNN-based methods for extracting ROI crops, especially in critical sagittal and axial slices.

**5. Transformer for Final Predictions**

The core of my solution revolves around Vision Transformers (ViTs). I trained multiple 5-fold cross-validation ViTs, which received sequences of segmented crops from different MRI planes:

- Sagittal T1 crops: Used for foraminal condition predictions
- Sagittal T2 crops: Used for spinal condition predictions
- Axial T2 crops: Used for subarticular and additional spinal condition predictions

These ViTs used ResNet18 as an encoder to extract meaningful features from each crop before passing it through the Transformer architecture.

**6. Positional Encoding in Transformer**

For positional encodings in the Transformer, I experimented with various configurations:

- Absolute vs. relative encodings
- Learnable vs. non-learnable encodings
- Side-distinguishing encodings vs. side-neutral encodings

The best results were achieved by using absolute encodings, without distinguishing sides, and grouping sequences by levels. However, for Axial T2 slices, this approach underperformed, so I retained the original, simpler architecture that worked better despite unstable training.

**7. Inference Pipeline**

The final model uses a multi-stage pipeline for inference:

Use CNN to extract ROI crops from MRI images.
Feed these crops into the ViT models for severity prediction across five lumbar spine conditions.

## Code Structure

- ROI: Processes MRI images to extract ROIs using CNN-based models.
- discriminators: Processes the cropped images to extract ROI in this slice level too.
- predictions: Trains the Vision Transformer on segmented crops from the different extracted crops.
- rsna-submission: Generates predictions for all intervertebral levels across all five conditions.

Note that all these notebooks were run on Kaggle and that even though I tried to adapt a few of them for this Git, most of the paths are still in Kaggle format. Some notebooks also comport pip installation cells that you may want to remove before running locally.


## Final Thoughts

This project was a deeply rewarding and challenging experience. I focused on developing an architecture that could generalize well across different MRI slices, leveraging transformers and CNNs. I would like to extend my gratitude to everyone who contributed knowledge, insights, and data throughout the competition.
Acknowledgments

Special thanks to @hengck23 for his invaluable public insights into DICOM data handling and 2D-to-3D projection techniques.