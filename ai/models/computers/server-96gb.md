# AI Server

GMKtec Evo-X2 AI Mini-PC AMD Ryzen AI Max+ 395 (96GB LPDDR5X 8000MHz Unified Memory)

> System running Ubuntu Server with AMD ROCm, configured with 72 GB VRAM pool allocated via TTM (`options amdttm pages_limit=18874368`), 256-bit memory bus bandwidth, and native speculative decoding (`draft_num_predict: 2`).

## Models installation

```bash
# MoE Model: Alibaba Qwen 3.6 35B A3B (Unsloth Dynamic Q8)
ollama pull jetelain/Qwen3.6-35B-A3B
ollama pull jetelain/Qwen3.6-35B-A3B:code

# Dense Model: Alibaba Qwen 3.8 27B (Unsloth Dynamic Q4)
ollama pull jetelain/Qwen3.8-27B
```

## Aliases with optimal context length

```bash
# 256k context: native maximum context for general tasks (~73 tokens/s, 100% unified VRAM)
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B",
  "from": "jetelain/Qwen3.6-35B-A3B",
  "parameters": {
    "num_ctx": 262144
  }
}'

# 256k context: coding tasks alias for OpenCode and Visual Studio
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:code",
  "parameters": {
    "num_ctx": 262144
  }
}'

# 256k context: dense model maintaining 100% unified VRAM at ~25 t/s
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.8-27B",
  "from": "jetelain/Qwen3.8-27B",
  "parameters": {
    "num_ctx": 262144
  }
}'
```

## Benchmark results (Tokens / second)

```
Qwen 3.6 35B A3B (Q8): [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k ── 192k ── 224k ── 256k]
                       100% Unified VRAM (~64 - 74 t/s, Prompt: 87 - 104 t/s, Draft Acc: 73 - 91%)
                       ▲ 100% VRAM across entire 256k context window (39.5 GB to 44.3 GB)

Qwen 3.8 27B (Q4):     [48k ── 64k ── 80k ── 96k ── 112k ── 128k ── 160k ── 192k ── 224k ── 256k]
                       100% Unified VRAM (~23 - 26 t/s, Prompt: 60 - 75 t/s, Draft Acc: 76 - 88%)
                       ▲ 100% VRAM across entire 256k context window (21.0 GB to 35.2 GB)
```

---

### Detailed Measurement Data

#### 1. Qwen3.6-35B-A3B (UD-Q8_K_XL, 40.0 GB, MTP Draft=2)

*Architecture: 41 layers, 2 KV heads, head dim 128, MoE 8/60 active experts (~3.2B active params).*

On the GMKtec Evo-X2 AI Mini-PC, the 40.0 GB Q8 model weights fit entirely inside the 72 GB ROCm VRAM allocation. Because Qwen3.6-35B-A3B uses a compact 2-head GQA design, each context token consumes only ~22 KB of KV cache, leaving the remaining ~30 GB of VRAM ample headroom for scaling context up to 256k tokens.

| Context Size | GPU Offload | Total VRAM Used | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|
| **48k (49152)** | **100% Unified VRAM** | 39.51 GB | 70.1 t/s | **72.6 t/s** | 87.5% |
| **64k (65536)** | **100% Unified VRAM** | 39.98 GB | 100.1 t/s | **72.0 t/s** | 88.7% |
| **80k (81920)** | **100% Unified VRAM** | 40.44 GB | 98.2 t/s | **68.4 t/s** | 82.4% |
| **96k (98304)** | **100% Unified VRAM** | 40.90 GB | 96.3 t/s | **72.0 t/s** | 88.7% |
| **112k (114688)**| **100% Unified VRAM** | 41.37 GB | 90.8 t/s | **73.7 t/s** | 91.4% |
| **128k (131072)**| **100% Unified VRAM** | 41.83 GB | 92.0 t/s | **60.5 t/s** | 65.9% |
| **160k (163840)**| **100% Unified VRAM** | 42.77 GB | 87.7 t/s | **66.8 t/s** | 76.9% |
| **192k (196608)**| **100% Unified VRAM** | 42.67 GB | 103.6 t/s | **64.1 t/s** | 73.4% |
| **224k (229376)**| **100% Unified VRAM** | 43.48 GB | 102.7 t/s | **69.9 t/s** | 86.1% |
| **256k (262144)**| **100% Unified VRAM** | 44.30 GB | 100.5 t/s | **73.1 t/s** | 87.5% |

*Key finding: Qwen3.6-35B-A3B Q8 achieves a remarkable ~65 to 74 tokens/second generation throughput across the entire context range up to 256k tokens. Memory usage reaches only 44.3 GB at 256k context, leaving ~28 GB of VRAM unused. With near-lossless 8-bit precision and high MTP draft acceptance (73% - 91%), this is the optimal flagship model for the server.*

#### 2. Qwen3.8-27B (UD-Q4_K_XL, 18.5 GB, MTP Draft=2)

*Architecture: 65 layers, 4 KV heads, head dim 5120 embedding.*

While dense 27B models suffer severe memory bandwidth bottlenecks on standard consumer GPUs with 8GB or 16GB of discrete VRAM when partially offloaded to CPU host memory, the GMKtec Evo-X2's unified 256-bit LPDDR5X-8000 memory bus allows the entire 27B model and its massive KV cache to reside 100% in GPU memory.

| Context Size | GPU Offload | Total VRAM Used | Prompt Eval Speed | Generation Speed | MTP Acceptance Rate |
|---|---|---|---|---|---|
| **48k (49152)** | **100% Unified VRAM** | 21.02 GB | 75.5 t/s | **22.9 t/s** | 84.9% |
| **64k (65536)** | **100% Unified VRAM** | 23.07 GB | 62.8 t/s | **25.7 t/s** | 88.7% |
| **80k (81920)** | **100% Unified VRAM** | 24.26 GB | 61.0 t/s | **23.9 t/s** | 76.9% |
| **96k (98304)** | **100% Unified VRAM** | 25.45 GB | 60.4 t/s | **25.2 t/s** | 86.1% |
| **112k (114688)**| **100% Unified VRAM** | 26.63 GB | 59.7 t/s | **25.0 t/s** | 83.8% |
| **128k (131072)**| **100% Unified VRAM** | 26.91 GB | 66.5 t/s | **25.8 t/s** | 88.7% |
| **160k (163840)**| **100% Unified VRAM** | 29.16 GB | 65.5 t/s | **24.5 t/s** | 80.3% |
| **192k (196608)**| **100% Unified VRAM** | 30.83 GB | 72.9 t/s | **23.3 t/s** | 76.6% |
| **224k (229376)**| **100% Unified VRAM** | 33.02 GB | 72.0 t/s | **24.4 t/s** | 81.3% |
| **256k (262144)**| **100% Unified VRAM** | 35.21 GB | 70.0 t/s | **24.5 t/s** | 82.4% |

*Key finding: Unlike discrete GPU systems where dense 27B models drop to 2-10 t/s due to PCIe offload bottlenecks, the AMD Strix Halo APU maintains a stable ~23 to 26 tokens/second across all context lengths up to 256k. Even with a 17 GB KV cache at 256k context, total memory stays at 35.2 GB, comfortably within the 72 GB VRAM ceiling.*

---

### 3. Hardware Comparison Summary (at 64k & 256k Context)

| Model & Quantization | File Size | Memory (64k / 256k) | Gen Speed (64k / 256k) | Prompt Speed (64k / 256k) | Evaluation Summary |
|---|---|---|---|---|---|
| **Qwen3.6-35B-A3B UD-Q8_K_XL** | 40.0 GB | 40.0 GB / 44.3 GB | **72.0 t/s / 73.1 t/s** | **100.1 t/s / 100.5 t/s** | **Flagship MoE**: Ultra-fast, near-lossless 8-bit precision, sustains ~70 t/s up to 256k context. |
| **Qwen3.8-27B UD-Q4_K_XL** | 18.5 GB | 23.1 GB / 35.2 GB | **25.7 t/s / 24.5 t/s** | **62.8 t/s / 70.0 t/s** | **Flagship Dense**: Solid ~25 t/s throughput up to 256k context with no PCIe penalty. |
