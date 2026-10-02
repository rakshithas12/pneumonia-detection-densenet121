# Evaluation records

The internal test values are reported by `notebooks/Untitled3_(1).ipynb` cell 65 for 5,439 images: accuracy 0.9638, precision 0.9478, sensitivity 0.9816, specificity 0.9460, F1 0.9644, ROC-AUC 0.9960, and confusion matrix `[[2573,147],[50,2669]]`. The notebook states the test set was not used for training or threshold selection. It does not establish patient-independent splitting.

The same notebook reports Kermany external testing at frozen threshold 0.16 on 624 images: accuracy 0.9535, precision 0.9308, sensitivity 1.0000, specificity 0.8761, F1 0.9642, ROC-AUC 0.9998, confusion matrix `[[205,29],[0,390]]`. The notebook also has a separate 0.50 operating point; another external notebook contains a 622-image construction and alternate checkpoint attempts. Keep those experiments distinct.

These are notebook-reported metrics, not clinical validation. The checkpoint is absent from this package, and the recorded checkpoint variants prevent independent reproduction here. Preprocessing differs between training and backend inference. No patient-level split is documented.
