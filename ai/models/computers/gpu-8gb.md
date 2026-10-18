# Laptop

NVIDIA RTX 4070 8GB Mobile with Intel Core Ultra 9 185H (64GB RAM)

> System configured with `OLLAMA_KV_CACHE_TYPE=q8_0` and no VRAM reservation required for display and OS processes (thanks to Intel Integrated Graphics).

## Models installation

```bash
ollama pull jetelain/Qwen3.5-4B
ollama pull jetelain/Qwen3.5-9B
ollama pull jetelain/Qwen3.6-35B-A3B:i3
ollama pull jetelain/Qwen3.6-35B-A3B:i3-code
```

## Aliases with optimal context length
```bash
# 78k context: largest size maintaining 100% Full GPU offload (~113 t/s)
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-4B",
  "from": "jetelain/Qwen3.5-4B",
  "parameters": {
    "num_ctx": 79872
  }
}'

# 88k context: largest size maintaining target >= 22 t/s
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-9B",
  "from": "jetelain/Qwen3.5-9B",
  "parameters": {
    "num_ctx": 90112
  }
}'

# 160k context: ultra-large context maintaining >= 30 t/s
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B:i3",
  "parameters": {
    "num_ctx": 163840
  }
}'

# 160k context: coding tasks alias for OpenCode and Visual Studio
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:i3-code",
  "parameters": {
    "num_ctx": 163840
  }
}'
```

## Benchmark results (Tokens / second)

```
Qwen 3.5 4B:      [48k ── 64k ── 78k] ─────── 80k ──────── 96k ─────── 128k
                   100% GPU (~112 t/s)      (95.4 t/s)    (63 t/s)    (49 t/s)
                   ▲ Max 100% GPU: 78k

Qwen 3.5 9B:      [48k ── 64k ── 72k ── 80k ── 88k] ─────── 96k ─────── 128k
                          ~22 - 25.6 tokens/s             (18.7 t/s)  (17.2 t/s)
                          ▲ Max ≥ 22 t/s: 88k

Qwen 3.6 35B A3B: [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k] ── 192k ── 256k
                                   ~31 - 41 tokens/s                   (29.7 t/s) (21.9 t/s)
                                   ▲ Max ≥ 30 t/s: 160k                ▲ Max Context (≥ 20 t/s): 256k

Qwen 3.8 27B (Dense): [48k]
                      2.36 tokens/s (Prompt: 0.58 t/s)
                      ❌ Severe DDR5 bandwidth bottleneck on 8GB VRAM GPU
```

---

### Detailed Measurement Data

#### 1. Qwen3.5-4B (UD-Q4_K_XL, 2.79 GB, MTP Draft=2)

*Architecture: 33 layers, 4 KV heads, head dim 160.*

Because the entire 4B model weighs only 2.79 GB, it leaves ample headroom in the 8GB VRAM budget for KV cache and computational buffers. It runs with **100% Full GPU offload** up to 78k context.

| Context Size | GPU Offload | CPU Overflow | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | **33/33 layers (100% GPU)** | 0 layers | 3.62 GB | 183.2 t/s | **104.4 t/s** | 83.8% |
| **64k (65536)** | **33/33 layers (100% GPU)** | 0 layers | 3.76 GB | 191.5 t/s | **112.1 t/s** | 92.8% |
| **68k (69632)** | **33/33 layers (100% GPU)** | 0 layers | 3.80 GB | 185.0 t/s | **103.8 t/s** | 86.1% |
| **72k (73728)** | **33/33 layers (100% GPU)** | 0 layers | 3.83 GB | 178.4 t/s | **94.9 t/s** | 69.5% |
| **76k (77824)** | **33/33 layers (100% GPU)** | 0 layers | 3.87 GB | 189.2 t/s | **114.4 t/s** | 92.8% |
| **78k (79872)** | **33/33 layers (100% GPU)** | 0 layers | **3.88 GB** | 188.7 t/s | **112.7 t/s** | 90.0% |
| **79k (80896)** | 84% GPU | 16% CPU | 3.84 GB | 165.2 t/s | **98.6 t/s** | 94.1% |
| **80k (81920)** | 84% GPU | 16% CPU | 3.85 GB | 156.4 t/s | **95.4 t/s** | 94.1% |
| **96k (98304)** | 74% GPU | 26% CPU | 3.76 GB | 124.0 t/s | **63.1 t/s** | 94.1% |
| **112k (114688)**| 67% GPU | 33% CPU | 3.71 GB | 105.2 t/s | **54.7 t/s** | 94.1% |
| **128k (131072)**| 60% GPU | 40% CPU | 3.68 GB | 89.6 t/s | **49.2 t/s** | 95.6% |

*Key finding: Qwen3.5-4B is the fastest model on this laptop. 78k context (`79872` tokens) is the exact maximum boundary where 100% of the model and KV cache fit entirely inside the RTX 4070 VRAM, sustaining peak generation speeds above 110 t/s.*

#### 2. Qwen3.5-9B (UD-Q4_K_XL, 5.71 GB, MTP Draft=2)

*Architecture: 33 layers, 4 KV heads, head dim 256.*

With a 5.71 GB model footprint, base weights occupy ~70% of available VRAM. Ollama offloads 21 out of 33 layers to the GPU, keeping remaining layers in fast DDR5 host memory.

| Context Size | GPU Offload | Host RAM | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | 62% GPU (21/33 layers) | 2.74 GB | 4.43 GB | 42.9 t/s | **25.6 t/s** | 84.9% |
| **56k (57344)** | 60% GPU (20/33 layers) | 2.91 GB | 4.38 GB | 41.5 t/s | **25.2 t/s** | 80.3% |
| **64k (65536)** | 58% GPU (19/33 layers) | 3.12 GB | 4.34 GB | 39.8 t/s | **23.2 t/s** | 76.9% |
| **72k (73728)** | 56% GPU (18/33 layers) | 3.35 GB | 4.32 GB | 38.2 t/s | **23.3 t/s** | 86.1% |
| **80k (81920)** | 54% GPU (18/33 layers) | 3.60 GB | 4.28 GB | 36.1 t/s | **21.8 t/s** | 88.7% |
| **88k (90112)** | 52% GPU (17/33 layers) | 3.85 GB | 4.23 GB | 34.5 t/s | **22.4 t/s** | 94.1% |
| **96k (98304)** | 50% GPU (16/33 layers) | 4.10 GB | 4.17 GB | 31.0 t/s | 18.7 t/s | 88.7% |
| **104k (106496)**| 48% GPU (16/33 layers) | 4.35 GB | 4.12 GB | 28.5 t/s | 20.2 t/s | 100.0% |
| **112k (114688)**| 48% GPU (16/33 layers) | 4.60 GB | 4.22 GB | 27.2 t/s | 19.7 t/s | 92.8% |
| **120k (122880)**| 46% GPU (15/33 layers) | 4.88 GB | 4.18 GB | 25.4 t/s | 17.6 t/s | 86.1% |
| **128k (131072)**| 45% GPU (15/33 layers) | 5.15 GB | 4.14 GB | 24.1 t/s | 17.2 t/s | 91.4% |

*Key finding: Qwen3.5-9B provides higher reasoning fidelity than 4B while easily maintaining the ≥ 20 t/s requirement up to 88k context (`90112` tokens). Above 88k context, KV cache allocation forces more layers into system RAM, dropping throughput under 20 t/s.*

#### 3. Qwen3.6-35B-A3B (UD-IQ3_S, 14.3 GB, MTP Draft=2)

*Architecture: 41 layers, 2 KV heads, head dim 128, MoE 8/60 active experts (~3.2B active params).*

Because Qwen3.6-35B-A3B is a Sparse MoE model activating only 3B parameters per token and uses an ultra-compact 2-head GQA design (~22 KB per token KV cache at `q8_0`), it scales across massive context windows without bottlenecking the system memory bus.

| Context Size | GPU Offload | Host RAM (CUDA_Host) | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | 30% GPU / 70% CPU | 10.89 GB | 4.50 GB | 39.5 t/s | **41.4 t/s** | 88.7% |
| **56k (57344)** | 29% GPU / 71% CPU | 11.02 GB | 4.39 GB | 38.2 t/s | **36.9 t/s** | 90.0% |
| **64k (65536)** | 28% GPU / 72% CPU | 11.20 GB | 4.25 GB | 38.0 t/s | **39.7 t/s** | 90.0% |
| **80k (81920)** | 27% GPU / 73% CPU | 11.55 GB | 4.09 GB | 38.9 t/s | **35.4 t/s** | 82.4% |
| **96k (98304)** | 25% GPU / 75% CPU | 11.90 GB | 3.92 GB | 37.1 t/s | **33.9 t/s** | 84.9% |
| **112k (114688)**| 24% GPU / 76% CPU | 12.25 GB | 3.68 GB | 36.5 t/s | **34.2 t/s** | 90.0% |
| **128k (131072)**| 23% GPU / 77% CPU | 12.60 GB | 3.58 GB | 35.0 t/s | **31.6 t/s** | 72.5% |
| **160k (163840)**| 20% GPU / 80% CPU | 13.30 GB | 3.20 GB | 33.8 t/s | **31.3 t/s** | 87.5% |
| **192k (196608)**| 20% GPU / 80% CPU | 14.00 GB | 3.27 GB | 31.2 t/s | **29.7 t/s** | 92.8% |
| **256k (262144)**| 19% GPU / 81% CPU | 15.40 GB | 3.35 GB | 26.5 t/s | **21.9 t/s** | 86.1% |

*Key finding: Despite being a 35B parameter model, Qwen3.6-35B-A3B is faster than 9B dense across all context lengths! Attention layers remain resident in GPU VRAM while the sparse experts execute out of DDR5 host RAM, sustaining >30 tokens/s up to 160k context and reaching 21.9 tokens/s even at 256k context.*

#### 4. Qwen3.8-27B (UD-IQ3_S, 11.2 GB, Dense): 8GB VRAM Limitation

*Architecture: 65 layers, 4 KV heads, head dim 256.*

Because Qwen3.8-27B is a dense model, generating every single token requires streaming all 11+ GB of quantized weights across the memory bus. On an 8GB VRAM GPU, 63% of layers are relegated to host DDR5 RAM.

| Context Size | GPU Offload | CPU Offload | Peak VRAM | Prompt Eval Speed | Generation Speed | Status |
|---|---|---|---|---|---|---|
| **48k (49152)** | 37% GPU (24/65 layers) | 63% CPU (41 layers) | 5.02 GB | 0.58 t/s | **2.36 t/s** | ❌ Memory bus stall |

*Key finding: Dense models larger than 12B cannot achieve interactive speeds (>20 t/s) when partially offloaded across PCIe / DDR5 system memory. Qwen3.8-27B should be reserved for systems with 16GB+ VRAM or unified memory AI servers (e.g. GMKtec Evo-X2).*

---

### 5. Hardware Comparison Summary (at 48k & 64k Context)

| Model & Quantization | File Size | Memory Split (64k) | Generation Speed (64k) | Prompt Speed (64k) | Evaluation Summary |
|---|---|---|---|---|---|
| **Qwen3.5-4B UD-Q4_K_XL** | 2.79 GB | **100% GPU** (3.76 GB VRAM) | **112.1 t/s** | **191.5 t/s** | **Max Speed**: Full GPU offload up to 78k, ultra-responsive daily driver. |
| **Qwen3.5-9B UD-Q4_K_XL** | 5.71 GB | 58% GPU / 42% CPU (4.34 GB VRAM) | **23.2 t/s** | 39.8 t/s | **Best Dense Quality**: Meets ≥ 20 t/s goal up to 88k context. |
| **Qwen3.6-35B-A3B UD-IQ3_S** | 14.3 GB | 28% GPU / 72% CPU (4.25 GB VRAM) | **39.7 t/s** | 38.0 t/s | **Best Overall**: Exceptional MoE scaling up to 160k (31 t/s) and 256k (22 t/s). |
| **Qwen3.8-27B UD-IQ3_S** | 11.2 GB | 37% GPU / 63% CPU (5.02 GB VRAM) | 2.36 t/s (48k) | 0.58 t/s | ❌ Dense RAM bottleneck on 8GB GPU. |