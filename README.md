# Multi-model Inference Engine with vLLM deployed on Linode LKE

This repository contains Kubernetes manifests for deploying vLLM (a high-throughput and memory-efficient inference engine for large language models) on a Linode LKE cluster with support for multiple model variants.

## Architecture

### Multi-Model Setup (Recommended)
- **Nginx Router**: Intelligent routing based on model selection
- **vLLM 8B Deployment**: Meta-Llama-3-8B-Instruct (1 GPU)
- **vLLM 70B Deployment**: Meta-Llama-3-70B-Instruct (2 GPUs with tensor parallelism)
- **Model Services**: ClusterIP services for internal routing
- **Storage**: PersistentVolumeClaim for Hugging Face model cache
- **GPU**: Multi-GPU support with automatic routing

## Files

### Core Files
- `pvc.yaml` - PersistentVolumeClaim for model cache
- `gpu-test.yaml` - GPU verification job

### Single Model (Legacy)
- `vllm-deployment.yaml` - Original single model deployment
- `vllm-service.yaml` - LoadBalancer service for single model

### Multi-Model Setup
- `vllm-8b-deployment.yaml` - 8B model deployment
- `vllm-70b-deployment.yaml` - 70B model deployment  
- `vllm-model-services.yaml` - ClusterIP services for models
- `nginx-router-config.yaml` - Nginx routing configuration
- `nginx-router-deployment.yaml` - Nginx router + LoadBalancer service

## Features

- ✅ Multiple model support with intelligent routing
- ✅ Automatic request routing based on model parameter
- ✅ Independent scaling per model
- ✅ Hugging Face model caching via shared PVC
- ✅ Readiness probe health checks
- ✅ GPU memory optimization per model
- ✅ OpenAI-compatible API
- ✅ Tensor parallelism support for large models

## Deployment

### Option 1: Multi-Model Setup (Recommended)

```bash
# Deploy storage and core infrastructure
kubectl apply -f pvc.yaml
kubectl apply -f gpu-test.yaml

# Deploy model services
kubectl apply -f vllm-model-services.yaml

# Deploy individual models
kubectl apply -f vllm-8b-deployment.yaml
kubectl apply -f vllm-70b-deployment.yaml

# Deploy nginx router with load balancer
kubectl apply -f nginx-router-config.yaml
kubectl apply -f nginx-router-deployment.yaml

# Check deployments
kubectl get deployments
kubectl get pods -o wide
kubectl get services
```

### Option 2: Single Model (Legacy)

```bash
# Deploy storage
kubectl apply -f pvc.yaml

# Deploy single model
kubectl apply -f vllm-deployment.yaml
kubectl apply -f vllm-service.yaml

# Check status
kubectl get pods -o wide
kubectl get services
```

## Usage

### Multi-Model Setup

Get the external IP:
```bash
kubectl get service vllm-router-service
```

Access at: `http://<EXTERNAL-IP>/v1/chat/completions`

**Request 8B Model:**
```bash
curl http://<EXTERNAL-IP>/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3-8B-Instruct",
    "messages": [{"role": "user", "content": "What is AI?"}],
    "max_tokens": 100
  }'
```

**Request 70B Model:**
```bash
curl http://<EXTERNAL-IP>/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3-70B-Instruct",
    "messages": [{"role": "user", "content": "What is AI?"}],
    "max_tokens": 100
  }'
```

**Health Checks:**
```bash
# Overall health
curl http://<EXTERNAL-IP>/health

# 8B model health
curl http://<EXTERNAL-IP>/health/8b

# 70B model health
curl http://<EXTERNAL-IP>/health/70b
```

### Python Client Example

```python
from openai import OpenAI

# Using 8B model
client = OpenAI(
    api_key="anything",
    base_url="http://<EXTERNAL-IP>/v1"
)

response = client.chat.completions.create(
    model="meta-llama/Meta-Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)

# Using 70B model - just change the model name
response = client.chat.completions.create(
    model="meta-llama/Meta-Llama-3-70B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

## Requirements

- Kubernetes cluster with multi-GPU nodes (NVIDIA)
- Linode LKE or similar
- kubectl configured
- For 70B model: Minimum 2 GPUs (with tensor parallelism)

## Model Specifications

### Meta-Llama-3-8B-Instruct
- GPU: 1x NVIDIA GPU
- GPU Memory: ~16GB
- Memory Utilization: 95%
- Max Context Length: 4096 tokens
- Typical Response Time: 50-200ms

### Meta-Llama-3-70B-Instruct
- GPU: 2x NVIDIA GPU (tensor parallel)
- GPU Memory: ~80GB (40GB per GPU)
- Memory Utilization: 90%
- Max Context Length: 8192 tokens
- Typical Response Time: 200-800ms

## Scaling

### Add More Model Variants

1. Create new deployment file (e.g., `vllm-mistral-deployment.yaml`)
2. Update `vllm-model-services.yaml` with new service
3. Update `nginx-router-config.yaml` with routing rules
4. Apply new manifests

### Auto-Scaling (Advanced)

To enable HPA for individual models:

```bash
kubectl autoscale deployment vllm-8b --min=1 --max=3 --cpu-percent=80
```

## Troubleshooting

### Check Router Status
```bash
kubectl logs -f deployment/nginx-router
```

### Check Model Status
```bash
kubectl logs -f deployment/vllm-8b
kubectl logs -f deployment/vllm-70b
```

### Verify Routing
```bash
# Check if router can reach models
kubectl exec -it <router-pod> -- curl http://vllm-8b-service:8000/health
kubectl exec -it <router-pod> -- curl http://vllm-70b-service:8000/health
```

## Future Enhancements

- [ ] More model variants (Mistral, Qwen, etc.)
- [ ] Auto-scaling (HPA) per model
- [ ] Prometheus metrics integration
- [ ] Request queuing and prioritization
- [ ] Model version management
- [ ] A/B testing support
- [ ] Custom model loading strategies
- [ ] Caching layer (Redis)
