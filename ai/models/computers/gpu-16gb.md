# Desktop

NVIDIA GeForce RTX 4080 16GB with Intel Core i7-14700KF (64GB RAM)

> System configured with `OLLAMA_KV_CACHE_TYPE=q8_0` and ~3 GB VRAM reserved for display and OS processes (~13 GB usable model VRAM budget).
> **Network Notice**: On this machine, Ollama runs on port **11435** (`OLLAMA_HOST=127.0.0.1:11435`), while port **11434** is reserved for [OllamaRouter](https://github.com/jetelain/OllamaRouter). Direct model management and alias creation must use port `11435`.

## Models installation

```bash
# Option A: Maximum Reasoning Precision (UD-Q8_K_XL - Recommended for 64GB Desktop RAM)
ollama pull jetelain/Qwen3.6-35B-A3B
ollama pull jetelain/Qwen3.6-35B-A3B:code

# Option B: Maximum Generation Speed (UD-IQ3_S - 115+ tokens/s)
ollama pull jetelain/Qwen3.6-35B-A3B:i3
ollama pull jetelain/Qwen3.6-35B-A3B:i3-code

# Dense Model: Full GPU offload up to 48k context
ollama pull jetelain/Qwen3.8-27B:iq3
```

## Aliases with optimal context length

```bash
# 160k context: ultra-large context (Q8 achieves ~58 t/s; i3 achieves ~115 t/s; draft_num_predict=2)
curl http://localhost:11435/api/create -H "Content-Type: application/json" -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B",
  "parameters": {
    "num_ctx": 163840
  }
}'

# 160k context: coding tasks alias for OpenCode and Visual Studio
curl http://localhost:11435/api/create -H "Content-Type: application/json" -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:code",
  "parameters": {
    "num_ctx": 163840
  }
}'

# 48k context: largest size maintaining 100% Full GPU offload (66/66 layers) at ~40 t/s (max context >= 20 t/s: 56k)
curl http://localhost:11435/api/create -H "Content-Type: application/json" -d '{
  "model": "Qwen3.8-27B",
  "from": "jetelain/Qwen3.8-27B:iq3",
  "parameters": {
    "num_ctx": 49152
  }
}'

# 24k context: high-speed speculative decoding alias maintaining 100% Full GPU offload with MTP at ~77 t/s
curl http://localhost:11435/api/create -H "Content-Type: application/json" -d '{
  "model": "Qwen3.8-27B:mtp",
  "from": "jetelain/Qwen3.8-27B:iq3",
  "parameters": {
    "num_ctx": 24576,
    "draft_num_predict": 2
  }
}'
```

*Note: If you prefer maximum speed (~115 t/s) over Q8 precision (~58 t/s), change the `from` field in the Qwen3.6-35B-A3B aliases to `jetelain/Qwen3.6-35B-A3B:i3` and `jetelain/Qwen3.6-35B-A3B:i3-code`.*

## Benchmark results (Tokens / second)

```
Qwen 3.6 35B A3B (i3): [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k] ── 192k ──── 256k
                       ~109 - 122 tokens/s (Prompt: 1030 - 1175 t/s, Draft Acc: 82-96%)   (98.5 t/s)
                       ▲ Max 100% GPU VRAM: 48k     ▲ Recommended Alias: 160k            ▲ Max (≥ 20 t/s): 256k+

Qwen 3.6 35B A3B (q4): [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k]
                       ~84 - 100 tokens/s (Prompt: 645 - 757 t/s, Draft Acc: 77-93%)
                       ▲ Partial Offload (22-25 CPU ovf) ▲ Tested up to 160k

Qwen 3.6 35B A3B (q8): [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k]
                       ~54 - 59 tokens/s (Prompt: 329 - 367 t/s, Draft Acc: 79-91%)
                       ▲ Partial Offload (31-33 CPU ovf) ▲ Recommended Alias: 160k (Highest Quality)

Qwen 3.8 27B (i3 No MTP): [24k ── 32k ── 48k] ──────────── 56k ───────────── 64k ──────── 72k ─── 80k
                          100% GPU (~40.0 - 40.4 t/s)    (23.1 t/s)        (14.7 t/s)   (18.7 t/s) (20.1 t/s)
                          ▲ Max 100% GPU: 48k            ▲ Max ≥ 20 t/s: 56k

Qwen 3.8 27B (i3 + MTP):  [16k ── 20k ── 24k] ──────────── 28k ──────── 32k ──────── 40k ──────── 48k
                          100% GPU (~71 - 77 t/s)        (52.8 t/s)     (46.0 t/s)   (34.6 t/s)   (27.4 t/s)
                          ▲ Max 100% GPU MTP: 24k                                                 ▲ Max ≥ 20 t/s: 48k
```

---

### Detailed Measurement Data

#### 1. Qwen3.6-35B-A3B: Quantization & Context Sweep

Because Qwen3.6-35B-A3B is a Sparse Mixture-of-Experts (MoE) model activating only **~3.2B parameters per token**, memory transfers across the PCIe bus are minimal compared to dense models. Even when a large portion of the model is offloaded to DDR5 host memory, token generation remains exceptionally fast.

##### A. UD-IQ3_S (`jetelain/Qwen3.6-35B-A3B:i3`, 14.3 GB, MTP Draft=2)

| Context Size | GPU Offload | Host Memory (CUDA_Host) | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | **42/42 layers (100% VRAM)** | 0 MiB | 15,810 MiB | 1,083.6 t/s | **118.5 t/s** | 91.4% |
| **64k (65536)** | **42/42 layers (100% GPU)** | 2,224 MiB | 15,820 MiB | 1,175.3 t/s | **122.0 t/s** | 86.1% |
| **80k (81920)** | **42/42 layers (100% GPU)** | 2,724 MiB | 15,806 MiB | 1,117.3 t/s | **117.1 t/s** | 92.8% |
| **96k (98304)** | **42/42 layers (100% GPU)** | 3,224 MiB | 15,806 MiB | 1,091.3 t/s | **111.6 t/s** | 87.5% |
| **112k (114688)**| **42/42 layers (100% GPU)** | 3,724 MiB | 15,810 MiB | 1,081.7 t/s | **113.1 t/s** | 92.8% |
| **128k (131072)**| **42/42 layers (100% GPU)** | 4,124 MiB | 15,814 MiB | 1,063.7 t/s | **109.4 t/s** | 82.4% |
| **160k (163840)**| **42/42 layers (100% GPU)** | 4,724 MiB | 15,724 MiB | 1,029.7 t/s | **114.9 t/s** | 95.6% |
| **192k (196608)**| **42/42 layers (100% GPU)** | 5,324 MiB | 15,789 MiB | 1,122.8 t/s | **111.0 t/s** | 91.4% |
| **256k (262144)**| **42/42 layers (100% GPU)** | 6,524 MiB | 15,820 MiB | 1,023.9 t/s | **98.5 t/s** | 83.8% |

##### B. UD-Q4_K_XL (21.3 GB, MTP Draft=2)

| Context Size | GPU Offload | CPU Overflow | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | ~10.5 GB in VRAM | 22 overflowing | 15,810 MiB | 757.0 t/s | **100.5 t/s** | 87.5% |
| **64k (65536)** | ~10.5 GB in VRAM | 23 overflowing | 15,814 MiB | 701.2 t/s | **93.4 t/s** | 82.4% |
| **80k (81920)** | ~10.5 GB in VRAM | 23 overflowing | 15,814 MiB | 694.1 t/s | **87.2 t/s** | 76.9% |
| **96k (98304)** | ~10.4 GB in VRAM | 24 overflowing | 15,810 MiB | 724.3 t/s | **91.7 t/s** | 90.0% |
| **112k (114688)**| ~10.4 GB in VRAM | 24 overflowing | 15,810 MiB | 668.0 t/s | **92.6 t/s** | 92.8% |
| **128k (131072)**| ~10.4 GB in VRAM | 25 overflowing | 15,814 MiB | 654.4 t/s | **91.1 t/s** | 90.0% |
| **160k (163840)**| ~10.3 GB in VRAM | 25 overflowing | 15,810 MiB | 645.2 t/s | **84.3 t/s** | 83.8% |

##### C. UD-Q8_K_XL (`jetelain/Qwen3.6-35B-A3B`, 36.4 GB, MTP Draft=2)

| Context Size | GPU Offload | CPU Overflow | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | ~10.0 GB in VRAM | 31 overflowing | 15,798 MiB | 333.1 t/s | **42.5 t/s** | 78.9% |
| **64k (65536)** | ~9.8 GB in VRAM | 32 overflowing | 15,810 MiB | 367.4 t/s | **59.4 t/s** | 91.4% |
| **80k (81920)** | ~9.8 GB in VRAM | 32 overflowing | 15,814 MiB | 363.0 t/s | **57.0 t/s** | 88.7% |
| **96k (98304)** | ~9.8 GB in VRAM | 32 overflowing | 15,810 MiB | 358.8 t/s | **54.7 t/s** | 80.3% |
| **112k (114688)**| ~9.7 GB in VRAM | 33 overflowing | 15,810 MiB | 329.1 t/s | **58.9 t/s** | 88.7% |
| **128k (131072)**| ~9.7 GB in VRAM | 33 overflowing | 15,810 MiB | 328.7 t/s | **54.6 t/s** | 81.3% |
| **160k (163840)**| ~9.6 GB in VRAM | 33 overflowing | 15,806 MiB | 355.6 t/s | **58.2 t/s** | 90.0% |

*Key finding: Both Q4 and Q8 effortlessly maintain high throughput (84 t/s and 58 t/s respectively) at 160k context, far exceeding the 20 t/s threshold. For maximum reasoning quality and accuracy, the official Q8 model (`jetelain/Qwen3.6-35B-A3B`) is the recommended default on this 64 GB system.*

#### 2. Qwen3.8-27B (UD-IQ3_S, No MTP vs MTP Draft=2)

**A. Standard Generation (`draft_num_predict: 0`)**

| Context Size | GPU Offload | CPU Offload | Peak VRAM | Prompt Eval Speed | Generation Speed |
|---|---|---|---|---|---|
| **24k (24576)** | **66/66 layers (100%)** | 0 layers | 14,663 MiB | 943.9 t/s | **40.4 t/s** |
| **32k (32768)** | **66/66 layers (100%)** | 0 layers | 14,975 MiB | 1,033.0 t/s | **40.3 t/s** |
| **48k (49152)** | **66/66 layers (100%)** | 0 layers | 15,599 MiB | 1,021.8 t/s | **40.0 t/s** |
| **56k (57344)** | 65/66 layers | 1 layer | 15,739 MiB | 858.1 t/s | **23.1 t/s** |
| **64k (65536)** | 63/66 layers | 3 layers | 15,785 MiB | 649.8 t/s | **14.7 t/s** |
| **72k (73728)** | 61/66 layers | 5 layers | 15,783 MiB | 765.4 t/s | **18.7 t/s** |
| **80k (81920)** | 58/66 layers | 8 layers | 15,649 MiB | 748.3 t/s | **20.1 t/s** |

**B. Speculative Decoding (`draft_num_predict: 2`)**

| Context Size | GPU Offload | CPU Offload | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **16k (16384)** | **66/66 layers (100%)** | 0 layers | 15,177 MiB | 998.1 t/s | **71.4 t/s** | 78.9% |
| **20k (20480)** | **66/66 layers (100%)** | 0 layers | 15,353 MiB | 1,005.6 t/s | **73.7 t/s** | 80.3% |
| **24k (24576)** | **66/66 layers (100%)** | 0 layers | 15,529 MiB | 1,017.4 t/s | **77.1 t/s** | 88.7% |
| **28k (28672)** | 64/66 layers | 2 layers | 15,451 MiB | 912.4 t/s | **52.8 t/s** | 84.9% |
| **32k (32768)** | 63/66 layers | 3 layers | 15,467 MiB | 885.3 t/s | **46.0 t/s** | 83.8% |
| **40k (40960)** | 61/66 layers | 5 layers | 15,521 MiB | 800.2 t/s | **34.6 t/s** | 80.0% |
| **48k (49152)** | 58/66 layers | 8 layers | 15,396 MiB | 695.9 t/s | **27.4 t/s** | 80.3% |

*Key finding: Because Qwen3.8-27B is a dense model, running even 1-3 layers on CPU creates a memory bus bottleneck that drops performance below 20 t/s at 64k context. However, with 100% GPU offload (at 48k context without MTP or 24k context with MTP), it achieves 40.0 t/s and 77.1 t/s respectively.*

#### 3. Quantization Format Comparison (at 48k Context)

| Model & Quantization | File Size | GPU Offload | Generation Speed | Prompt Speed | Evaluation Summary |
|---|---|---|---|---|---|
| **Qwen3.6-35B-A3B UD-IQ3_S** | 14.3 GB | **42/42 (100%)** | **118.5 t/s** | **1,083.6 t/s** | **Max Speed**: Blazing fast, fits entirely in 16GB VRAM budget. |
| **Qwen3.6-35B-A3B UD-Q4_K_XL** | 21.3 GB | ~10.5 GB VRAM (22 CPU ovf) | **100.5 t/s** | 757.0 t/s | Excellent balance, runs >84 t/s up to 160k context. |
| **Qwen3.6-35B-A3B UD-Q8_K_XL** | 36.4 GB | ~10.0 GB VRAM (31 CPU ovf) | **42.5 - 59.4 t/s** | 333.1 - 367.4 t/s | **Max Precision (Recommended)**: Near-lossless FP16 quality at ~58 t/s at 160k. |
| **Qwen3.8-27B UD-IQ3_S** | 11.2 GB | **66/66 (100%)** | **40.0 t/s** | **1,021.8 t/s** | **Optimal for 27B**: 100% GPU offload up to 48k context. |
| **Qwen3.8-27B UD-Q4_K_XL** | 16.4 GB | 44/66 (22 on CPU) | 10.1 t/s | 596.7 t/s | Fails ≥ 20 t/s target (22 layers on CPU). |
| **Qwen3.8-27B UD-Q8_K_XL** | 29.3 GB | ~25/66 (41 on CPU) | ~3 - 4 t/s | ~200 t/s | Severe memory bandwidth stall on CPU. |

