# Création des modèles LLM

Les modèles ont été envoyés sur ollama.com, il n'est pas nécessaire de les recréer. Cette page retrace leur création.

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

5. Publier les modèles
- `ollama push jetelain/Qwen3.6-35B-A3B`
- `ollama push jetelain/Qwen3.6-35B-A3B-Code`

## Gemma 4 26B A4B

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


4. Créer le modèle avec ollama
- `ollama create jetelain/Gemma-4-26B-A4B -f Gemma-4-26B-A4B.txt`

5. Publier le modèle
- `ollama push jetelain/Gemma-4-26B-A4B`

## Qwen3.6 27B

1. Récupérer le fichier [Qwen3.6-27B.txt](models/Qwen3.6-27B/Qwen3.6-27B.txt)

2. Télécharger les fichiers gguf de Unsloth
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF/resolve/main/Qwen3.6-27B-UD-Q4_K_XL.gguf`

3. Vérifier les gguf

`sha256sum *.gguf`

| Fichier                            | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| Qwen3.6-27B-UD-Q4_K_XL.gguf        | 4085665ee36d82a672a238a43f0e5643f2f0e39f2d7bd5d373f0ef10ecf53095 |
| mmproj-BF16.gguf                   | 05353347512982ee62317b9d8c89372bc815f4b4043580e7ef3ad411ec1a1cd3 |


4. Créer le modèle avec ollama
- `ollama create jetelain/Qwen3.6-27B -f Qwen3.6-27B.txt`

5. Publier le modèle
- `ollama push jetelain/Qwen3.6-27B`


## Gemma 4 31B

1. Récupérer le fichier [Gemma-4-31B.txt](models/Gemma-4-31B/Gemma-4-31B.txt)

2. Télécharger les fichiers gguf de Unsloth
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/gemma-4-31B-it-UD-Q4_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/mtp-gemma-4-31B-it.gguf `

3. Vérifier les gguf

`sha256sum *.gguf`

| Fichier                            | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| gemma-4-31B-it-UD-Q4_K_XL.gguf     | dd69c1566b0d559f303424e2b00e830837bb6c8b25e5baa08c75f11709bd41db |
| mmproj-BF16.gguf                   | 7a4601b12ec680f706a7e0c7e1f78f579b1db64d485a9e352ee87d4b9daa45e4 |
| mtp-gemma-4-31B-it.gguf            | 5ae8b0117bed601e8924c6305bd5b0585de361d51f0e77091bcb4252cf1f27de |


4. Créer le modèle avec ollama
- `ollama create jetelain/Gemma-4-31B -f Gemma-4-31B.txt`

5. Publier le modèle
- `ollama push jetelain/Gemma-4-31B`


## Qwen3.8 27B

1. Récupérer le fichier [Qwen3.8-27B.txt](models/Qwen3.8-27B/Qwen3.8-27B.txt)

2. Télécharger les fichiers gguf de Unsloth
- `curl -LO https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main/Qwen3.8-27B-UD-Q4_K_XL.gguf`

3. Vérifier les gguf

`sha256sum *.gguf`

| Fichier                            | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| Qwen3.8-27B-UD-Q4_K_XL.gguf        | 3f227079003add2511437e5b1e94812e363385225bf6a9b47b0054a72bc8b01e |
| mmproj-BF16.gguf                   | 83ee4f4f205fa514161778c41df1ea14144faa0f713510893b63c2395f5c2d53 |


4. Créer le modèle avec ollama
- `ollama create jetelain/Qwen3.8-27B -f Qwen3.8-27B.txt`

5. Publier le modèle
- `ollama push jetelain/Qwen3.8-27B`