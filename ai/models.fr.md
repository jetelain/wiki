# Création des modèles LLM

## Qwen 3.6 35B A3B

1. Récupérer les fichiers [Qwen3.6-35B-A3B.txt](models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B.txt) et [Qwen3.6-35B-A3B-Code.txt](models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B-Code.txt)

2. Télécharger les fichiers gguf de Unsloth
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF/resolve/main/Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF/resolve/main/mmproj-BF16.gguf`

3. Vérifier les gguf

`sha256sum *.gguf`

| Fichier                         | SHA256                                                           |
|---------------------------------|------------------------------------------------------------------|
| Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf | 6c6b816537abad90b250a0972b345466028d861ddfe316d5f0de31ca6440f781 |
| mmproj-BF16.gguf                | da63cb47a76763c712393f8a017070188a304fa39f8aeea6edc629ed7b975cfa |

4. Créer les modèles avec ollama
- `ollama create jetelain/Qwen3.6-35B-A3B -f Qwen3.6-35B-A3B.txt`
- `ollama create jetelain/Qwen3.6-35B-A3B-Code -f Qwen3.6-35B-A3B-Code.txt`

5. Publier les modèless
- `ollama push jetelain/Qwen3.6-35B-A3B`
- `ollama push jetelain/Qwen3.6-35B-A3B-Code`

## Gemma 4

1. Récupérer le fichier [Gemma-4-26B-A4B.txt](models/Gemma-4-26B-A4B/Gemma-4-26B-A4B.txt)

2. Télécharger les fichiers gguf de Unsloth
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/gemma-4-26B-A4B-it-UD-Q8_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/mtp-gemma-4-26B-A4B-it.gguf`

3. Vérifier les gguf

`sha256sum *.gguf`

| Fichier                            | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| gemma-4-26B-A4B-it-UD-Q8_K_XL.gguf | 50e180d69641e017d7e08a6f602988effde8232ff6bc0231e839636fdcc03d8f |
| mmproj-BF16.gguf                   | 41926ed5f1403cf5add23b0684992805ea6f97253096132e769e65646b8cef9d |
| mtp-gemma-4-26B-A4B-it.gguf        | 6326fb9f5e487aa8dcdd313a091e3c67724cb2a666ec3b7d2895b5b26d93ed1b |


4. Créer les modèles avec ollama
- `ollama create jetelain/Gemma-4-26B-A4B -f Gemma-4-26B-A4B.txt`

5. Publier les modèless
- `ollama push jetelain/Gemma-4-26B-A4B`