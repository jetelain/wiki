# Ollama Model Descriptions & Usage Recommendations

This document contains copy-paste ready descriptions for [ollama.com](https://ollama.com) for each published model family under `jetelain/*`, accompanied by hardware sizing, usage recommendations, and the zero-disk-overhead alias pattern.

---

## 🧭 Quick Hardware Sizing Matrix

| Hardware Tier | Usable VRAM / RAM | Recommended Model & Tag | Expected Throughput | Max Efficient Context |
|---|---|---|---|---|
| **Entry / Laptop (8 GB VRAM)** | 8 GB GPU + 32–64 GB RAM | `jetelain/Qwen3.5-4B:latest` (Q4 MTP)<br>`jetelain/Qwen3.5-4B:q6` (Q6 MTP)<br>`jetelain/Qwen3.6-35B-A3B:q4`<br>`jetelain/Qwen3.6-35B-A3B:latest` (Q8) | ~112 t/s (4B Q4)<br>~43 t/s (4B Q6)<br>~35 t/s (35B q4)<br>~23 t/s (35B Q8) | 78k (100% GPU)<br>128k (4B Q6 MTP)<br>160k–192k (q4)<br>112k (Q8) |
| **Midrange / Desktop (16 GB VRAM)** | 16 GB GPU + 64 GB RAM | `jetelain/Qwen3.6-35B-A3B:q4`<br>`jetelain/Qwen3.6-35B-A3B:latest` (Q8)<br>`jetelain/Qwen3.8-27B:i3`<br>`jetelain/Qwen3.5-9B:latest` | ~90–100 t/s (35B q4)<br>~58 t/s (35B Q8)<br>~40–77 t/s (27B i3)<br>~65 t/s (9B) | 160k (q4 / Q8)<br>48k (100% GPU)<br>160k+ (100% GPU) |
| **High-End Enthusiast (24 GB VRAM)** | 24 GB GPU (RTX 3090/4090) | `jetelain/Qwen3.6-35B-A3B:latest` (Q8)<br>`jetelain/Qwen3.8-27B:latest` (Q4)<br>`jetelain/Gemma-4-31B:latest` | ~70–85 t/s<br>~35–45 t/s<br>~40–50 t/s | 64k–128k<br>64k<br>64k |
| **Unified Memory Server / Mac (64–96 GB+)** | 72–96 GB LPDDR5X / Unified | `jetelain/Qwen3.6-35B-A3B:code` (Q8)<br>`jetelain/Qwen3.8-27B:latest` (Q4)<br>`jetelain/Gemma-4-26B-A4B:latest` (Q8) | ~70–74 t/s (35B Q8)<br>~25 t/s (27B Q4)<br>~50 t/s (Gemma Q8) | **256k** (100% VRAM)<br>**256k** (100% VRAM)<br>128k+ |

---

## 💡 The "Zero-Cost Alias" Tip

Instead of modifying environment variables or re-entering context flags on every CLI invocation, Ollama allows you to create **custom local aliases** via its `/api/create` endpoint.

### Why use aliases?
1. **Zero Disk Overhead**: Ollama reuses existing model layers via internal pointers. Creating an alias is instantaneous and consumes virtually 0 bytes of extra disk space.
2. **Hardware-Tailored Context Window (`num_ctx`)**: Lock in the maximum context size that fits your GPU VRAM (e.g. 78k on 8 GB laptops, 160k on 16 GB GPUs, 256k on 96 GB unified memory).
3. **Clean & Standard Model Names**: Drop the `jetelain/` namespace so frontend clients (Open WebUI, OpenCode, Cursor, Continue, Visual Studio) reference clean names like `Qwen3.6-35B-A3B-Code`.
4. **Task-Specific Presets**: Lock in temperature (e.g., 0.6 for coding, 1.0 for general chat) without needing to configure it in every client.

```bash
# General Syntax:
curl http://localhost:11434/api/create -d '{
  "model": "<CleanAliasName>",
  "from": "jetelain/<ModelName>:<Tag>",
  "parameters": {
    "num_ctx": <OptimalContextSize>
  }
}'
```

---

# 1. Family: `jetelain/Qwen3.6-35B-A3B`

### Ollama Model Card Description (`jetelain/Qwen3.6-35B-A3B`)

````markdown
# Qwen 3.6 35B A3B (Unsloth Dynamic & Speculative Decoding)

Advanced Mixture-of-Experts (MoE) model with **8 of 60 active experts (~3.2B active parameters)** from Alibaba Cloud, packaged with **Unsloth Dynamic Quantization**, native **Multi-Token Prediction (MTP / Speculative Decoding)**, and full **Multimodal Vision (`mmproj`)** support.

Optimized specifically for high-throughput coding, agentic workflows, and ultra-large context processing (up to 256k tokens).

## 🚀 Key Highlights

- **Unsloth Dynamic Quantization (UD)**: Selective precision keeps critical attention and gate layers at high precision for near-lossless output while compressing expert weights.
- **Native MTP Speculative Decoding (`draft_num_predict: 2`)**: Delivers an instant 1.3x–1.8x generation speedup with zero quality degradation.
- **Multimodal Vision Included**: Bundled with `mmproj-BF16` for high-resolution image analysis and OCR.
- **Extended Context Window**: Default configured at 64k or 128k, scalable up to the native **262,144** RoPE context limit.
- **GQA Efficiency**: Uses a 2-KV-head architecture consuming only ~22 KB of KV cache per token, enabling massive context on modest memory.

## 📦 Tags & Variants

| Tag | Quantization | Size | Default Context | Target Use Case |
|---|---|---|---|---|
| `latest` | UD-Q8_K_XL | 40 GB | 128k (`131072`) | Maximum general reasoning precision (temp 1.0) |
| `code` | UD-Q8_K_XL | 40 GB | 128k (`131072`) | High-precision coding and architecture tasks (temp 0.6) |
| `q4` | UD-Q4_K_XL | 22 GB | 128k (`131072`) | **Recommended Sweet Spot**: High quality + fast speed (~35 t/s on 8GB, ~90 t/s on 16GB) |
| `q4-code` | UD-Q4_K_XL | 22 GB | 128k (`131072`) | Coding sweet spot for OpenCode, Cline, Visual Studio |
| `i3` | UD-IQ3_S | 15.3 GB | 64k (`65536`) | Maximum raw throughput (~115 t/s on 16GB GPU), lower precision |
| `i3-code` | UD-IQ3_S | 15.3 GB | 64k (`65536`) | Ultra-high-speed draft/coding, lower precision |

## 💡 Usage Recommendations

### 1. Fast CLI Run
```bash
# Recommended sweet spot (quality + speed):
ollama run jetelain/Qwen3.6-35B-A3B:q4

# For highest precision coding tasks:
ollama run jetelain/Qwen3.6-35B-A3B:code

# For maximum speed on 16GB GPUs:
ollama run jetelain/Qwen3.6-35B-A3B:i3-code
```

### 2. Hardware Recommendations
- **8 GB VRAM (Laptops with 64GB RAM)**:
  - **Recommended Sweet Spot**: Use `:q4` (or `:q4-code`). Fixes `i3` quality/fidelity issues while sustaining **>30 tokens/s up to 128k context** and **23.5 tokens/s at 160k**.
  - **Maximum Quality**: Use `:latest` or `:code` (Q8). Sustains **≥ 20 tokens/s up to 112k context** and ~17 t/s at 160k with near-lossless FP16 reasoning.
  - **Maximum Speed**: Use `:i3` or `:i3-code` if raw throughput (~31–41 t/s) is prioritized over nuance.
- **16 GB VRAM (RTX 4080 / 4090 Mobile)**:
  - Use `:q4` for an ideal balance of quality and speed (**~85–100 tokens/s** up to 160k context).
  - Use `:latest` or `:code` (Q8) for maximum precision with 64GB host RAM (~58 tokens/s).
  - Use `:i3` or `:i3-code` for **100% full GPU offload** at **~115 tokens/s** up to 160k context.
- **Unified Memory (64–96 GB / Apple Silicon / AMD Strix Halo)**: Use `:latest` or `:code` (Q8). Runs 100% in VRAM at **~70–74 tokens/s** all the way to **256k context**.

### 3. 💡 Hardware Alias Tip (Zero Disk Overhead)
Create a clean local alias tailored to your GPU VRAM ceiling:

```bash
# On 8 GB Laptops with 64 GB RAM (160k context, q4 sweet spot at ~24-35 t/s):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:q4",
  "parameters": {
    "num_ctx": 163840
  }
}'

# On 16 GB Desktop GPUs (160k context, full GPU offload at ~115 t/s):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:i3-code",
  "parameters": {
    "num_ctx": 163840
  }
}'

# On 64-96 GB Unified Memory Servers (256k native max context at ~73 t/s):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-35B-A3B-Code",
  "from": "jetelain/Qwen3.6-35B-A3B:code",
  "parameters": {
    "num_ctx": 262144
  }
}'
```

### 4. OpenCode / Agent Integration
Recommended settings for `~/.config/opencode/opencode.json`:
```json
{
  "model": "ollama/Qwen3.6-35B-A3B-Code:latest",
  "limit": {
    "context": 262144,
    "output": 65536
  },
  "reasoning": true
}
```
````

---

# 2. Family: `jetelain/Qwen3.8-27B`

### Ollama Model Card Description (`jetelain/Qwen3.8-27B`)

````markdown
# Qwen 3.8 27B (Dense Flagship with Unsloth Dynamic & MTP)

Alibaba Qwen 3.8 27B dense foundation model with 65 transformer layers, packaged with **Unsloth Dynamic Quantization**, **Multi-Token Prediction (MTP / Speculative Decoding)**, and integrated **Multimodal Vision (`mmproj`)**.

Ideal for tasks requiring exhaustive knowledge depth, continuous dense attention, and structured logical deduction where MoE routing is not desired.

## 🚀 Key Highlights

- **Dense Flagship Architecture**: 65 layers with 4 KV heads and 5120 hidden dimension.
- **Unsloth Dynamic (UD)**: Significantly outperforms standard GGUF quantizations by preserving weight sensitivity across critical matrices.
- **MTP Speculative Decoding (`draft_num_predict: 2`)**: Up to 1.5x throughput acceleration on supported GPUs.
- **Multimodal Vision Included**: Bundled with `mmproj-BF16` for visual understanding.
- **Full Context Scalability**: Native support up to 256k tokens.

## 📦 Tags & Variants

| Tag | Quantization | Size | Default Context | Target Use Case |
|---|---|---|---|---|
| `latest` | UD-Q4_K_XL | 18.5 GB | 128k (`131072`) | Balanced quality & memory footprint |
| `q8` | UD-Q8_K_XL | 32 GB | 128k (`131072`) | Near-lossless 8-bit precision for high-VRAM systems |
| `i3` | UD-IQ3_S | 12.0 GB | 64k (`65536`) | Optimized to fit inside 16 GB VRAM GPUs |

## 💡 Usage Recommendations

### 1. Fast CLI Run
```bash
ollama run jetelain/Qwen3.8-27B
```

### 2. Hardware Recommendations
- **16 GB VRAM GPUs**: Use `:i3`. Achieves **100% GPU offload** up to 48k context at **~40 tokens/s**, or up to **~77 tokens/s** with MTP speculative decoding at 24k context.
- **24 GB VRAM (RTX 3090 / 4090)**: Use `:latest` (Q4). Fits entirely in VRAM for standard context lengths.
- **Unified Memory (64–96 GB / AMD Strix Halo / Apple Silicon)**: Use `:latest` (Q4) or `:q8`. Delivers a rock-solid **~24–26 tokens/s** across the entire **256k context window** without PCIe bandwidth drops.

### 3. 💡 Hardware Alias Tip (Zero Disk Overhead)
Lock in the exact context size suitable for your hardware without copying files:

```bash
# On 16 GB GPUs (48k context, 100% full GPU offload):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.8-27B",
  "from": "jetelain/Qwen3.8-27B:i3",
  "parameters": {
    "num_ctx": 49152
  }
}'

# On 64-96 GB Unified Memory Systems (256k context, 100% unified VRAM):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.8-27B",
  "from": "jetelain/Qwen3.8-27B",
  "parameters": {
    "num_ctx": 262144
  }
}'
```
````

---

# 3. Family: `jetelain/Qwen3.5-4B`

### Ollama Model Card Description (`jetelain/Qwen3.5-4B`)

````markdown
# Qwen 3.5 4B (Unsloth Dynamic Q4 & Q6 with MTP & Vision)

Ultra-fast, compact multimodal foundation model from Alibaba Cloud, packaged with **Unsloth Dynamic quantization (Q4_K_XL & Q6_K_XL)**, native **Multi-Token Prediction (`draft_num_predict: 2`)**, and **Multimodal Vision (`mmproj-BF16`)**.

Engineered for edge devices, laptops, fast agent reasoning loops, and real-time interactive chats.

## 🚀 Key Highlights

- **Blazing Speed**: Generates at **~112–115 tokens/s** on standard 8 GB mobile GPUs (Q4).
- **Massive Context in Small VRAM**: Maintains **100% full GPU offload up to 78k context** in Q4 within an 8 GB VRAM budget.
- **High-Precision Q6 Option**: Sustains **>22 tokens/s across the entire 128k context range** (~43 t/s at 64k) with superior code/reasoning fidelity.
- **Integrated Vision**: Process images, screenshots, diagrams, and OCR natively via `mmproj-BF16`.
- **MTP Speculative Decoding**: Preconfigured with `draft_num_predict: 2` (60%–96% draft acceptance rate).
- **Default 64k Context**: Ready out-of-the-box for large documents and multi-turn conversations.

## 📦 Tags

- `latest`: Unsloth Dynamic Q4_K_XL (~3.0 GB model + 675 MB mmproj), 64k default context. Maximum speed (~112 t/s) with 100% GPU offload up to 78k context.
- `q6`: Unsloth Dynamic Q6_K_XL with MTP (~4.26 GB model + 675 MB mmproj), 64k default context. Higher reasoning/coding precision, sustaining >22 t/s across the entire 128k context range (~43 t/s at 64k).

## 💡 Usage Recommendations

```bash
# Run interactive chat (Q4 speed champion)
ollama run jetelain/Qwen3.5-4B

# High precision Q6 chat
ollama run jetelain/Qwen3.5-4B:q6

# Multimodal prompt with an image
ollama run jetelain/Qwen3.5-4B "Describe this diagram: /path/to/diagram.png"
```

### Hardware Recommendations
- **Any 8 GB GPU (RTX 3060/3070/4060/4070)**:
  - `latest` (Q4 MTP): 100% full GPU offload up to 78k tokens at **>110 t/s**.
  - `q6` (Q6 MTP): High-precision reasoning, maintains **>22 t/s up to 128k context** (~43 t/s at 64k).
- **CPU / Apple Silicon M-series**: Runs effortlessly with minimal RAM consumption (< 5 GB).

### 💡 Hardware Alias Tip (Zero Disk Overhead)
Lock in hardware-tailored context for maximum performance on 8 GB GPUs:

```bash
# 78k context alias for maximum 100% GPU offload (Q4)
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-4B",
  "from": "jetelain/Qwen3.5-4B",
  "parameters": {
    "num_ctx": 79872
  }
}'

# 64k context alias for high-precision Q6
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-4B-Q6",
  "from": "jetelain/Qwen3.5-4B:q6",
  "parameters": {
    "num_ctx": 65536
  }
}'
```
````

---

# 4. Family: `jetelain/Qwen3.5-9B`

### Ollama Model Card Description (`jetelain/Qwen3.5-9B`)

````markdown
# Qwen 3.5 9B (Unsloth Dynamic Q4_K_XL with MTP & Vision)

High-performance compact foundation model balancing high intelligence with lightweight hardware requirements. Features **Unsloth Dynamic Q4_K_XL quantization**, **MTP Speculative Decoding**, and integrated **Multimodal Vision (`mmproj-BF16`)**.

The sweet spot for local reasoning, tool calling, document summarization, and vision-language tasks on consumer GPUs.

## 🚀 Key Highlights

- **Unsloth Dynamic Q4_K_XL**: Enhanced quantization maintaining critical matrix fidelity.
- **High Throughput**: Generates at **~60–70 tokens/s** on 16GB GPUs and up to 24k context on 8GB GPUs.
- **Extended Context**: Out-of-the-box 64k context window with native RoPE scalability up to 256k.
- **MTP Speculative Decoding**: Pre-tuned `draft_num_predict: 2` for faster generation.
- **Vision Multimodal**: Includes `mmproj-BF16` (921 MB) for visual perception.

## 📦 Tags

- `latest`: Unsloth Dynamic Q4_K_XL (~6.1 GB model + 921 MB mmproj), 64k default context.

## 💡 Usage Recommendations

```bash
ollama run jetelain/Qwen3.5-9B
```

### Hardware Recommendations
- **8 GB VRAM GPUs**: Full GPU offload up to 24k context (~60 t/s). Maintains responsive flow (>22 t/s) up to 88k context with hybrid memory.
- **16 GB VRAM GPUs**: 100% full GPU offload across 160k+ context at **~65 tokens/s**.
- **Unified Memory (32 GB+)**: Runs entirely in memory with vast context headroom.

### 💡 Hardware Alias Tip (Zero Disk Overhead)
Lock in the optimal context threshold for 8 GB or 16 GB GPUs:

```bash
# On 8 GB GPUs (88k context maintaining >22 t/s):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-9B",
  "from": "jetelain/Qwen3.5-9B",
  "parameters": {
    "num_ctx": 90112
  }
}'

# On 16 GB GPUs (160k context, full GPU offload):
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.5-9B",
  "from": "jetelain/Qwen3.5-9B",
  "parameters": {
    "num_ctx": 163840
  }
}'
```
````

---

# 5. Family: `jetelain/Qwen3.6-27B`

### Ollama Model Card Description (`jetelain/Qwen3.6-27B`)

````markdown
# Qwen 3.6 27B (Dense with Unsloth Dynamic & Vision)

Alibaba Qwen 3.6 27B dense foundation model featuring **Unsloth Dynamic Q4_K_XL quantization**, **MTP Speculative Decoding**, and **Multimodal Vision (`mmproj-BF16`)**.

## 🚀 Key Highlights

- **Dense 27B Architecture**: Full-parameter attention for comprehensive reasoning and synthesis.
- **Unsloth Dynamic Precision**: Minimal perplexity loss compared to 16-bit baseline.
- **Speculative Acceleration**: Configured with `draft_num_predict: 2`.
- **Large Context**: 128k default context window (`num_ctx: 131072`).
- **Vision Support**: Multimodal input enabled via `mmproj-BF16`.

## 📦 Tags

- `latest`: Unsloth Dynamic Q4_K_XL (~18 GB), 128k default context.

## 💡 Usage Recommendations

```bash
ollama run jetelain/Qwen3.6-27B
```

### Hardware Recommendations
- **24 GB VRAM GPUs (RTX 3090 / RTX 4090)**: Optimal setup for full GPU offload.
- **Unified Memory (64 GB+)**: Excellent performance for large-context dense generation.

### 💡 Hardware Alias Tip (Zero Disk Overhead)
```bash
curl http://localhost:11434/api/create -d '{
  "model": "Qwen3.6-27B",
  "from": "jetelain/Qwen3.6-27B",
  "parameters": {
    "num_ctx": 131072
  }
}'
```
````

---

# 6. Family: `jetelain/Gemma-4-26B-A4B`

### Ollama Model Card Description (`jetelain/Gemma-4-26B-A4B`)

````markdown
# Gemma 4 26B A4B (Google MoE with Unsloth Dynamic Q8 & MTP Draft)

Google Gemma 4 Mixture-of-Experts (MoE) architecture with **~4B active parameters**, packaged with **Unsloth Dynamic Q8_K_XL quantization**, dedicated **MTP draft model (`mtp-gemma-4-26B-A4B-it.gguf`)**, and **Multimodal Vision (`mmproj-BF16`)**.

Delivers Google's state-of-the-art conversational fluency, instruction-following precision, and safety alignment with the speed of an MoE.

## 🚀 Key Highlights

- **Near-Lossless 8-bit Precision**: UD-Q8_K_XL ensures pristine model reasoning and math capability.
- **Dedicated Speculative Draft Model**: Features an explicit `DRAFT mtp-gemma-4-26B-A4B-it.gguf` pairing with `draft_num_predict: 2` for accelerated generation.
- **Gemma 4 Multimodal Template**: Formatted with Google's Gemma 4 parser and vision projection.
- **MoE Efficiency**: Only activates ~4B parameters per token for fast response times.

## 📦 Tags

- `latest`: UD-Q8_K_XL (~29 GB) + dedicated MTP draft module + mmproj-BF16.

## 💡 Usage Recommendations

```bash
ollama run jetelain/Gemma-4-26B-A4B
```

### Hardware Recommendations
- **32 GB+ VRAM / Unified Memory (Mac Studio, AMD Strix Halo 96GB)**: Best suited for holding the 29 GB Q8 weights completely in GPU memory.
- **Parameters**: Best used at `temperature: 1.0`, `top_k: 64`, `top_p: 0.95`.

### 💡 Hardware Alias Tip (Zero Disk Overhead)
```bash
curl http://localhost:11434/api/create -d '{
  "model": "Gemma-4-26B-A4B",
  "from": "jetelain/Gemma-4-26B-A4B",
  "parameters": {
    "num_ctx": 131072
  }
}'
```
````

---

# 7. Family: `jetelain/Gemma-4-31B`

### Ollama Model Card Description (`jetelain/Gemma-4-31B`)

````markdown
# Gemma 4 31B (Google Dense with Unsloth Dynamic Q4 & MTP Draft)

Google Gemma 4 31B dense foundation model packaged with **Unsloth Dynamic Q4_K_XL quantization**, dedicated **MTP draft model (`mtp-gemma-4-31B-it.gguf`)**, and **Multimodal Vision (`mmproj-BF16`)**.

The premier choice for dense Gemma 4 reasoning, code generation, creative writing, and multimodal analysis.

## 🚀 Key Highlights

- **Dense Google Gemma 4 Architecture**: Full dense attention across all 31B parameters.
- **Unsloth Dynamic Q4_K_XL**: Compact 20 GB footprint with superior quality retention over standard Q4.
- **Explicit Speculative Decoding**: Uses `DRAFT mtp-gemma-4-31B-it.gguf` with `draft_num_predict: 2` for increased tokens per second.
- **Multimodal Support**: Analyze images, figures, and charts via `mmproj-BF16`.

## 📦 Tags

- `latest`: UD-Q4_K_XL (~20 GB) + dedicated MTP draft module + mmproj-BF16.

## 💡 Usage Recommendations

```bash
ollama run jetelain/Gemma-4-31B
```

### Hardware Recommendations
- **24 GB VRAM GPUs (RTX 3090 / 4090)**: Fits comfortably in VRAM for fast, unthrottled generation.
- **Unified Memory (64 GB+)**: Flawless full offload with large context headroom.

### 💡 Hardware Alias Tip (Zero Disk Overhead)
```bash
curl http://localhost:11434/api/create -d '{
  "model": "Gemma-4-31B",
  "from": "jetelain/Gemma-4-31B",
  "parameters": {
    "num_ctx": 131072
  }
}'
```
````
