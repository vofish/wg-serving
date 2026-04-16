# LLaMA3-8B Azure

## Configuration
| Kind | Model Server | Model | Provider | Accelerator |
| --- | --- | --- | --- | --- |
| Deployment | vLLM | llama3-8b | Azure | GPU H100 |

This overlay composes **H100**, **spot** (AKS spot node pool label/toleration), **blobfuse** (storage base + Azure-aligned cache volume), and **xla-cache** (same env/volume pattern as the TPU catalog component for parity).

### Scheduling notes
- **Spot + H100:** Pods use `nodeSelector` entries from both the [H100 component](../../../components/azure/resources/gpu/1-H100/h100.yaml) (`karpenter.azure.com/sku-gpu-name: H100`) and the [spot component](../../../components/azure/resources/spot/spot.yaml) (`kubernetes.azure.com/scalesetpriority: spot`). Nodes must satisfy **both** labels for scheduling (e.g. GPU spot node pools or Karpenter NodePool that sets equivalent labels).

## Usage

The template can be deployed with the following commands:

```
kustomize build core/deployment/vllm/llama3-8b/azure | kubectl apply -f -
```

### External `catalog generate` (example)

If your tooling resolves components by short name, an Azure-equivalent invocation may look like:

```
catalog generate --kind core/deployment --model-server vllm --model llama3-8b --provider azure --components h100,spot,blobfuse,xla-cache --container-image <gpu-vllm-image:tag> --pvc-name <your-blob-csi-pvc> --model-path vllm_models/llama3-8b-hf --catalog-root "wg-serving/serving-catalog"
```

- **`--container-image`:** Use an image built for **NVIDIA GPU** on Linux, not a GKE TPU-only image, unless you know it is multi-backend.
- **`--pvc-name`:** Must be an existing **Azure Blob CSI** (fuse) PVC in the namespace; avoid GCS-specific claim names unless that is what you created.
- **`--model-path`:** Same semantics as GKE: relative path under `/data/models` for weights on the mounted volume.