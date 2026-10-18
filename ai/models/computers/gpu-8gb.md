# Laptop

NVIDIA RTX 4070 8GB Mobile with Intel Core Ultra 9 185H

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
```