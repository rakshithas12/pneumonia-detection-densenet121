# Model and inference

The backend implements DenseNet121 with block configuration `(6, 12, 24, 16)` and a two-class linear head. The expected class mapping is `0 = NORMAL`, `1 = PNEUMONIA`. `backend/app/model/loader.py` looks for `backend/artifacts/model/best_robustness_finetuned_densenet121.pth`, loads it using `torch.load`, unwraps supported checkpoint dictionaries, and applies strict state-dict matching. It sets the model to evaluation mode. No checkpoint is included in this package.

Inference resizes to 224×224, converts to three-channel grayscale, tensorizes, normalizes with ImageNet means/stds, runs the model, then applies temperature scaling (`T = 1.0726476907730103`). Pneumonia is selected when its probability is at least 0.16. Confidence is the selected class probability; uncertainty is one minus confidence; uncertainty above 0.10 is marked `UNCERTAIN / REVIEW`.

Grad-CAM targets `features.denseblock4`. It provides a visualization of activation attribution and is not a causal explanation or evidence of pathology. See the root README for checkpoint placement and validation limitations.
