# Laptop

NVIDIA RTX 4070 8GB Mobile with Intel Core Ultra 9 185H (64GB RAM)

> System configured with `OLLAMA_KV_CACHE_TYPE=q8_0` and no VRAM reservation required for display and OS processes (thanks to Intel Integrated Graphics).

## Models installation

```bash
# Option A: Balanced Quality & High Speed (UD-Q4_K_XL - Recommended Sweet Spot for 64GB RAM)
ollama pull jetelain/Qwen3.6-35B-A3B:q4

# Option B: Maximum Reasoning Quality (UD-Q8_K_XL - 40GB, fits in 64GB RAM)
ollama pull jetelain/Qwen3.6-35B-A3B:latest
ollama pull jetelain/Qwen3.6-35B-A3B:code

# Option C: Maximum Generation Speed (UD-IQ3_S - 31-41 tokens/s)
ollama pull jetelain/Qwen3.6-35B-A3B:i3
ollama pull jetelain/Qwen3.6-35B-A3B:i3-code

# Dense Models
ollama pull jetelain/Qwen3.5-4B
ollama pull jetelain/Qwen3.5-9B
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

# 160k context: ultra-large context maintaining >= 30 t/s (i3) or >= 23 t/s (q4)
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B:q4",
  "parameters": {
    "num_ctx": 163840
  }
}'

# 160k context: coding tasks alias for OpenCode and Visual Studio
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:q4",
  "parameters": {
    "num_ctx": 163840
  }
}'
```

*Note: If answer quality on `i3` is insufficient, `jetelain/Qwen3.6-35B-A3B:q4` is the recommended default on this laptop (maintaining >30 t/s up to 128k context and 23.5 t/s at 160k). For absolute maximum reasoning accuracy, use `jetelain/Qwen3.6-35B-A3B:latest` (Q8, maintaining ≥ 20 t/s up to 112k context).*

## Benchmark results (Tokens / second)

```
Qwen 3.5 4B (q4 MTP):   [48k ── 64k ── 78k] ─────── 80k ──────── 96k ─────── 128k
                        100% GPU (~112 t/s)      (95.4 t/s)    (63 t/s)    (49 t/s)
                        ▲ Max 100% GPU: 78k

Qwen 3.5 4B (q6 MTP):   [48k ── 64k ── 72k ── 78k ── 80k ── 96k ── 112k ── 128k]
                        ~45 t/s  43 t/s  42 t/s  38 t/s  34 t/s  29 t/s  27 t/s  (22.9 t/s)
                        ▲ Max Context (≥ 20 t/s): 128k

Qwen 3.5 9B:           [48k ── 64k ── 72k ── 80k ── 88k] ─────── 96k ─────── 128k
                               ~22 - 25.6 tokens/s             (18.7 t/s)  (17.2 t/s)
                               ▲ Max ≥ 22 t/s: 88k

Qwen 3.6 35B A3B (i3): [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k] ── 192k ── 256k
                                        ~31 - 41 tokens/s                   (29.7 t/s) (21.9 t/s)
                                        ▲ Max ≥ 30 t/s: 160k                ▲ Max Context (≥ 20 t/s): 256k

Qwen 3.6 35B A3B (q4): [48k ── 64k ── 80k ── 96k ── 112k ── 128k] ──── 160k ── 192k ── 256k
                                        ~30 - 37 tokens/s                  (23.5 t/s)(21.8 t/s)(16.6 t/s)
                                        ▲ Max ≥ 30 t/s: 128k                ▲ Max ≥ 20 t/s: 192k

Qwen 3.6 35B A3B (q8): [48k ── 56k ── 64k ── 80k ── 96k ── 112k] ────── 128k ── 160k ── 256k
                                        ~20 - 25.7 tokens/s                 (17.8 t/s)(16.8 t/s)(13.7 t/s)
                                        ▲ Max ≥ 20 t/s: 112k (Max Quality)

Qwen 3.8 27B (Dense):  [48k]
                       2.36 tokens/s (Prompt: 0.58 t/s)
                       ❌ Severe DDR5 bandwidth bottleneck on 8GB VRAM GPU
```

---

### Detailed Measurement Data

#### 1. Qwen3.5-4B: Quantization & Context Sweep

*Architecture: 33 layers, 4 KV heads, head dim 160.*

##### A. UD-Q4_K_XL with MTP Speculative Decoding (`jetelain/Qwen3.5-4B`, 2.79 GB, MTP Draft=2)

Because the entire Q4 model weighs only 2.79 GB, it leaves ample headroom in the 8GB VRAM budget for KV cache and computational buffers. It runs with **100% Full GPU offload** up to 78k context while MTP draft prediction delivers an instant 2x speedup (>110 tokens/s).

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

##### B. UD-Q6_K_XL with MTP Speculative Decoding (`jetelain/Qwen3.5-4B:q6`, 4.26 GB, MTP Draft=2)

From `unsloth/Qwen3.5-4B-MTP-GGUF` (4.26 GB, includes 33 base layers + 1 embedded MTP speculative decoding layer). Base model weights occupy 4.26 GB, which forces 8 to 16 layers into DDR5 host memory as context scales. However, native Multi-Token Prediction (MTP with 60%–96% acceptance rate) keeps token generation fast, **maintaining >22 tokens/s across the entire 128k context range**:

| Context Size | GPU Offload | CPU Overflow | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | 30/34 layers (77% GPU) | 4 layers (1.30 GB) | 4.47 GB | 15.9 t/s | **45.3 t/s** | 60.7% |
| **64k (65536)** | 26/34 layers (69% GPU) | 8 layers (1.92 GB) | 4.20 GB | 51.5 t/s | **43.4 t/s** | 95.6% |
| **72k (73728)** | 26/34 layers (69% GPU) | 8 layers (1.96 GB) | 4.29 GB | 52.2 t/s | **41.9 t/s** | 95.6% |
| **78k (79872)** | 24/34 layers (65% GPU) | 10 layers (2.21 GB)| 4.14 GB | 48.7 t/s | **38.3 t/s** | 91.4% |
| **80k (81920)** | 24/34 layers (65% GPU) | 10 layers (2.22 GB)| 4.16 GB | 45.9 t/s | **34.0 t/s** | 71.6% |
| **96k (98304)** | 22/34 layers (61% GPU) | 12 layers (2.69 GB)| 4.14 GB | 43.6 t/s | **28.9 t/s** | 71.6% |
| **112k (114688)**| 19/34 layers (56% GPU) | 15 layers (3.13 GB)| 3.99 GB | 36.8 t/s | **27.6 t/s** | 88.7% |
| **128k (131072)**| 18/34 layers (53% GPU) | 16 layers (3.60 GB)| 4.07 GB | 38.5 t/s | **22.9 t/s** | 65.9% |

*Comparison with Standard Q6 (No MTP from `unsloth/Qwen3.5-4B-GGUF`):*
- At 64k context: **43.4 t/s** (MTP) vs 36.1 t/s (Standard, +20%).
- At 78k context: **38.3 t/s** (MTP) vs 26.8 t/s (Standard, +43%).
- At 96k context: **28.9 t/s** (MTP) vs 21.2 t/s (Standard, +36%).
- At 128k context: **22.9 t/s** (MTP) vs 18.6 t/s (Standard, +23%). Without MTP, Q6 drops under the 20 t/s threshold at 128k; **with MTP, it remains comfortably interactive across the entire 128k context window**.

##### C. UD-Q8_K_XL Standard (`Qwen3.5-4B-UD-Q8_K_XL.gguf`, 5.54 GB, No MTP)

With 5.54 GB base weights, the 8GB VRAM capacity cannot maintain 100% GPU offload at any practical context length. Even at 32k context, 8 layers are forced into DDR5 host memory:

| Context Size | GPU Offload | CPU Overflow | Peak VRAM | Prompt Eval Speed | Generation Speed | Evaluation Summary |
|---|---|---|---|---|---|---|
| **32k (32768)** | 25/33 layers (67% GPU) | 8 layers (2.61 GB) | 5.29 GB | 63.3 t/s | 19.6 t/s | ❌ Fails ≥ 20 t/s target |
| **48k (49152)** | 23/33 layers (64% GPU) | 10 layers (2.96 GB)| 5.34 GB | 59.1 t/s | 17.6 t/s | Partial offload DDR5 bottleneck |
| **64k (65536)** | 21/33 layers (61% GPU) | 12 layers (3.41 GB)| 5.28 GB | 49.3 t/s | 17.0 t/s | Partial offload DDR5 bottleneck |
| **72k (73728)** | 21/33 layers (61% GPU) | 12 layers (3.47 GB)| 5.41 GB | 52.8 t/s | 14.0 t/s | Partial offload DDR5 bottleneck |
| **80k (81920)** | 19/33 layers (58% GPU) | 14 layers (3.79 GB)| 5.29 GB | 47.1 t/s | 13.1 t/s | Partial offload DDR5 bottleneck |
| **96k (98304)** | 17/33 layers (54% GPU) | 16 layers (4.35 GB)| 5.13 GB | 46.0 t/s | 12.8 t/s | Partial offload DDR5 bottleneck |
| **112k (114688)**| 17/33 layers (54% GPU) | 16 layers (4.49 GB)| 5.37 GB | 48.3 t/s | 13.1 t/s | Partial offload DDR5 bottleneck |
| **128k (131072)**| 15/33 layers (52% GPU) | 18 layers (4.91 GB)| 5.35 GB | 38.0 t/s | 11.2 t/s | Partial offload DDR5 bottleneck |

*Key finding: For Qwen3.5-4B on an 8GB GPU:
- **Q4 with MTP (`jetelain/Qwen3.5-4B`)**: Undisputed speed champion, 100% GPU offload up to 78k context (~112 t/s) and >49 t/s up to 128k.
- **Q6 with MTP (`jetelain/Qwen3.5-4B:q6`)**: High-precision sweet spot for 4B. Even though KV cache forces 8–16 layers to DDR5 RAM, MTP draft acceptance (60–96%) maintains 23–45 t/s across the entire 128k context range (passing the ≥ 20 t/s interactive threshold).
- **Q8 Standard (`jetelain/Qwen3.5-4B:q8`)**: Avoid on 8GB GPUs (11–19.6 t/s). Dense layer streaming across DDR5 bottleneck without MTP falls below the 20 t/s goal.*

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

#### 3. Qwen3.6-35B-A3B: Quantization & Context Sweep

*Architecture: 41 layers, 2 KV heads, head dim 128, MoE 8/60 active experts (~3.2B active params).*

Because Qwen3.6-35B-A3B is a Sparse MoE model activating only ~3.2B parameters per token and uses an ultra-compact 2-head GQA design (~22 KB per token KV cache at `q8_0`), memory transfers across PCIe and DDR5 host RAM are minimal compared to dense models. Even on an 8GB VRAM GPU, offloading expert layers into the 64GB DDR5 host memory sustains fast token generation across i3, q4, and q8 quantizations.

##### A. UD-IQ3_S (`jetelain/Qwen3.6-35B-A3B:i3`, 14.3 GB, MTP Draft=2)

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

*Key finding: Maximum generation speed (>30 t/s up to 160k context, and 21.9 t/s at 256k), but the aggressive i3 quantization can lead to noticeable quality loss or subtle reasoning flaws on complex coding and analytical tasks.*

##### B. UD-Q4_K_XL (`jetelain/Qwen3.6-35B-A3B:q4`, 21.3 GB, MTP Draft=2)

| Context Size | GPU Offload | Host RAM (CUDA_Host) | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | 20% GPU / 80% CPU (42/42) | 17.84 GB | 4.38 GB | 40.2 t/s | **35.5 t/s** | 90.0% |
| **56k (57344)** | 20% GPU / 80% CPU (42/42) | 17.84 GB | 4.40 GB | 39.7 t/s | **37.0 t/s** | 84.9% |
| **64k (65536)** | 19% GPU / 81% CPU (42/42) | 17.99 GB | 4.29 GB | 39.5 t/s | **35.8 t/s** | 83.8% |
| **80k (81920)** | 18% GPU / 82% CPU (42/42) | 18.32 GB | 4.02 GB | 39.9 t/s | **34.7 t/s** | 90.0% |
| **96k (98304)** | 17% GPU / 83% CPU (42/42) | 18.65 GB | 3.87 GB | 38.3 t/s | **31.1 t/s** | 77.9% |
| **112k (114688)**| 16% GPU / 84% CPU (42/42) | 18.95 GB | 3.65 GB | 35.9 t/s | **29.7 t/s** | 76.6% |
| **128k (131072)**| 15% GPU / 85% CPU (40/42) | 19.09 GB | 3.50 GB | 32.2 t/s | **30.6 t/s** | 87.3% |
| **160k (163840)**| 15% GPU / 85% CPU (33/42) | 19.74 GB | 3.55 GB | 31.9 t/s | **23.5 t/s** | 75.6% |
| **192k (196608)**| 14% GPU / 86% CPU (26/42) | 20.52 GB | 3.45 GB | 28.7 t/s | **21.8 t/s** | 86.1% |
| **256k (262144)**| 14% GPU / 86% CPU (18/42) | 21.72 GB | 3.50 GB | 25.2 t/s | **16.6 t/s** | 81.3% |

*Key finding: **Recommended sweet spot for this laptop.** UD-Q4_K_XL fixes the answer quality and precision issues observed in i3 while easily sustaining >30 tokens/s up to 128k context. It remains comfortably above the 20 tokens/s interactive target up to 192k context (23.5 t/s at 160k, 21.8 t/s at 192k).*

##### C. UD-Q8_K_XL (`jetelain/Qwen3.6-35B-A3B:latest`, 36.4 GB, MTP Draft=2)

| Context Size | GPU Offload | Host RAM (CUDA_Host) | Peak VRAM | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|---|
| **48k (49152)** | 12% GPU / 88% CPU (42/42) | 33.38 GB | 4.47 GB | 26.2 t/s | **23.4 t/s** | 83.8% |
| **56k (57344)** | 11% GPU / 89% CPU (42/42) | 33.65 GB | 4.23 GB | 22.7 t/s | **25.7 t/s** | 87.5% |
| **64k (65536)** | 11% GPU / 89% CPU (42/42) | 33.66 GB | 4.25 GB | 26.9 t/s | **23.0 t/s** | 82.4% |
| **80k (81920)** | 11% GPU / 89% CPU (40/42) | 33.80 GB | 4.00 GB | 26.9 t/s | **22.5 t/s** | 87.5% |
| **96k (98304)** | 11% GPU / 89% CPU (38/42) | 34.00 GB | 4.06 GB | 26.3 t/s | **22.1 t/s** | 76.9% |
| **112k (114688)**| 10% GPU / 90% CPU (34/42) | 34.31 GB | 3.95 GB | 24.5 t/s | **20.3 t/s** | 83.8% |
| **128k (131072)**| 10% GPU / 90% CPU (30/42) | 34.66 GB | 3.85 GB | 20.8 t/s | **17.8 t/s** | 84.9% |
| **160k (163840)**| 10% GPU / 90% CPU (26/42) | 35.12 GB | 3.81 GB | 19.9 t/s | **16.8 t/s** | 84.9% |
| **192k (196608)**| 10% GPU / 90% CPU (22/42) | 35.65 GB | 3.77 GB | 22.9 t/s | **14.8 t/s** | 81.3% |
| **256k (262144)**| 9% GPU / 91% CPU (15/42)  | 36.61 GB | 3.73 GB | 18.8 t/s | **13.7 t/s** | 75.6% |

*Key finding: **Maximum reasoning precision and fidelity** (near-lossless FP16 output). Because this machine is equipped with 64GB host RAM, the 34–37 GB memory footprint fits without paging or system memory pressure. It sustains the ≥ 20 t/s threshold up to 112k context (`20.3 t/s`) and remains perfectly usable for deep reasoning and code generation at 160k (`16.8 t/s`).*

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
| **Qwen3.5-4B UD-Q4_K_XL (MTP)** | 2.79 GB | **100% GPU** (3.76 GB VRAM) | **112.1 t/s** | **191.5 t/s** | **Max Speed**: Full GPU offload up to 78k with MTP speculative decoding. |
| **Qwen3.5-4B UD-Q6_K_XL (MTP)** | 4.26 GB | 69% GPU / 31% CPU (4.20 GB VRAM) | **43.4 t/s** | 51.5 t/s | **High Precision 4B with MTP**: Meets ≥ 20 t/s target all the way to 128k (22.9 t/s at 128k, 43.4 t/s at 64k). |
| **Qwen3.5-4B UD-Q8_K_XL** | 5.54 GB | 61% GPU / 39% CPU (5.28 GB VRAM) | 17.0 t/s | 49.3 t/s | ❌ Fails ≥ 20 t/s goal on 8GB GPU due to DDR5 bandwidth on dense layers. |
| **Qwen3.5-9B UD-Q4_K_XL** | 5.71 GB | 58% GPU / 42% CPU (4.34 GB VRAM) | **23.2 t/s** | 39.8 t/s | **Best Dense Quality**: Meets ≥ 20 t/s goal up to 88k context. |
| **Qwen3.6-35B-A3B UD-IQ3_S** | 14.3 GB | 28% GPU / 72% CPU (4.25 GB VRAM) | **39.7 t/s** | 38.0 t/s | **Max Speed MoE**: Scaling up to 160k (31 t/s) and 256k (22 t/s), but lower precision. |
| **Qwen3.6-35B-A3B UD-Q4_K_XL** | 21.3 GB | 19% GPU / 81% CPU (4.29 GB VRAM) | **35.8 t/s** | 39.5 t/s | **Sweet Spot (Speed + Quality)**: Fixes i3 quality degradation, maintains >30 t/s up to 128k and ≥ 20 t/s up to 192k context. |
| **Qwen3.6-35B-A3B UD-Q8_K_XL** | 36.4 GB | 11% GPU / 89% CPU (4.25 GB VRAM) | **23.0 t/s** | 26.9 t/s | **Max Precision**: Highest reasoning/code quality, maintains ≥ 20 t/s up to 112k context on 64GB RAM. |
| **Qwen3.8-27B UD-IQ3_S** | 11.2 GB | 37% GPU / 63% CPU (5.02 GB VRAM) | 2.36 t/s (48k) | 0.58 t/s | ❌ Dense RAM bottleneck on 8GB GPU. |