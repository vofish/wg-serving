# Storage Component

This directory contains component configuration for storage on the deployment. `base/` contains a template for loading model weights from a PersistentVolumeClaim. The user will need to specify PVC name (PVC_NAME) and model path in the base patch located in `base/deployment.patch.yaml`. The storage component is only targeted to work with vllm at the moment.

## Support
Current storage component supports the following storage systems:

### GKE
- GCSFuse
    - This configuration expects a pre-existing GCSFuse backed PVC (eg: model-pvc)
    - This configuration will require user to manually add/replace MODEL_PATH value with actual GCS model path.

### Azure (AKS)
- **Blobfuse (CSI)** — `azure/blobfuse`
    - Composes `storage/vllm/base` (PVC mount at `/data/models`, `MODEL_PATH`, `--model=/data/models/$(MODEL_PATH)`, `--download-dir=/data/models`) and adds an in-memory `emptyDir` volume `azure-blobfuse-cache` (optional client-side cache; mount to the container is cluster/driver-specific if needed).
    - Expects a pre-provisioned PVC whose PV uses the [Azure Blob CSI driver](https://github.com/kubernetes-sigs/blob-csi-driver) (fuse mount). Replace placeholder `PVC_NAME` with your claim name (e.g. via `catalog generate --pvc-name ...` or kustomize replacements).
- **XLA cache** — `azure/xla-cache`
    - Sets `VLLM_XLA_CACHE_PATH=/data/models` and aligns model-data mount/PVC with the storage base (same shape as `tpu/xla-cache` for cross-cloud parity). Use when your vLLM build benefits from a fixed XLA cache path on mounted weights.

**Note:** `--pvc-name gcs-fuse-csi-static-pvc` is a GKE-oriented example; on AKS use the name of your Blob CSI PVC. `--container-image` pointing at a GKE/TPU image is syntactically valid but usually wrong for AKS GPU nodes—use a GPU-capable vLLM image for H100.
