# TinyLlama-1.1b AWS

## Configuration
| Kind | Model Server | Model | Provider | Accelerator |
| --- | --- | --- | --- | --- |
| Deployment | vLLM | tinyllama-1.1b | AWS | GPU L4 |

## Usage

The template can be deployed with the following commands:

```
kustomize build core/deployment/vllm/tinyllama-1.1b/aws | kubectl apply -f -
```
