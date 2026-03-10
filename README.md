# vLLM Kubernetes Deployment

This repository contains Kubernetes manifests for deploying vLLM (a high-throughput and memory-efficient inference engine for large language models) on a Linode LKE cluster.

## Architecture

- **Deployment**: vLLM server running Meta-Llama-3-8B-Instruct model
- **Service**: LoadBalancer service exposing the API on port 80
- **Storage**: PersistentVolumeClaim for Hugging Face model cache
- **GPU**: Single NVIDIA GPU support

## Files

- `vllm-deployment.yaml` - Main vLLM deployment configuration
- `vllm-service.yaml` - LoadBalancer service for external access
- `pvc.yaml` - PersistentVolumeClaim for model cache
- `gpu-test.yaml` - GPU verification job

## Features

- ✅ Hugging Face model caching via PVC
- ✅ Readiness probe health checks
- ✅ GPU memory optimization (0.95 utilization)
- ✅ OpenAI-compatible API
- ✅ Max model length set to 4096 tokens

## Deployment

```bash
# Apply all manifests
kubectl apply -f .

# Check pod status
kubectl get pods -o wide

# Get service endpoint
kubectl get service vllm-service
```

## Usage

Once deployed, access the inference API at:
```
http://<EXTERNAL-IP>/v1/chat/completions
```

### Example Request

```bash
curl http://172.232.243.67/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3-8B-Instruct",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

## Requirements

- Kubernetes cluster with GPU node (NVIDIA)
- Linode LKE or similar
- kubectl configured

## Future Enhancements

- [ ] Multiple model support
- [ ] Auto-scaling (HPA)
- [ ] Monitoring & metrics
- [ ] Liveness probes
- [ ] Security policies
- [ ] Request queuing
