# Methodology record

The training notebook records the Kaggle `reflex7/cxr-data-set`, filters to Normal and Pneumonia, reports 36,284 images before exact MD5 deduplication and 30,283 after, then performs an image-level stratified 70/15/15 split. It reports zero corrupt images in the scan and no exact cross-class duplicate groups. No patient-independent grouping is documented.

The notebook records ImageNet-pretrained DenseNet121, a classifier-training stage with frozen features, then fine-tuning of DenseBlock4 and `norm5`; weighted cross-entropy; AdamW; augmentation during training; and early stopping. It includes resumed and robustness experiments. The committed `training_configuration.csv` differs from notebook values, so this repository does not establish one unambiguous final training recipe.

Training data is converted to RGB, resized to 224×224, tensorized, and ImageNet-normalized. Backend inference instead converts images to grayscale repeated across three channels before normalization. Do not assume identical input distributions. This documentation records the difference; it does not change the model pipeline.
