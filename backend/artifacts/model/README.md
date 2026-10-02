# Model checkpoint

The GitHub-ready package does not include the trained checkpoint. The original project contains a file with the expected name, but this packaging environment has no PyTorch installed, so it could not verify its state-dict compatibility, classifier output dimensions, or provenance. The notebooks record several checkpoint variants, and no authorized download URL or redistribution terms are documented.

To enable backend inference, obtain the intended compatible model from an authorized source and place it here without renaming:

```text
backend/artifacts/model/best_robustness_finetuned_densenet121.pth
```

The backend constructs the architecture in `backend/app/model/densenet.py` and loads the checkpoint with strict state-dict matching. If the file is missing or incompatible, the backend cannot perform inference. The application reports standby when the file is absent.

Do not commit the checkpoint unless its identity and redistribution rights have been confirmed. It is ignored by the root `.gitignore`.
