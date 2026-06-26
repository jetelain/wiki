# LLM Model Creation

Models have been uploaded to ollama.com, there's no need to recreate them. This page documents their creation.

## Qwen 3.6 35B A3B

1. Retrieve the [Qwen3.6-35B-A3B.txt](models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B.txt) and [Qwen3.6-35B-A3B-Code.txt](models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B-Code.txt) files

2. Download Unsloth gguf files
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF/resolve/main/Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF/resolve/main/mmproj-BF16.gguf`

3. Verify the gguf files

`sha256sum *.gguf`

| File                            | SHA256                                                           |
|---------------------------------|------------------------------------------------------------------|
| Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf | 6c6b816537abad90b250a0972b345466028d861ddfe316d5f0de31ca6440f781 |
| mmproj-BF16.gguf                | da63cb47a76763c712393f8a017070188a304fa39f8aeea6edc629ed7b975cfa |

4. Create models with ollama
- `ollama create jetelain/Qwen3.6-35B-A3B -f Qwen3.6-35B-A3B.txt`
- `ollama create jetelain/Qwen3.6-35B-A3B-Code -f Qwen3.6-35B-A3B-Code.txt`

5. Publish the models
- `ollama push jetelain/Qwen3.6-35B-A3B`
- `ollama push jetelain/Qwen3.6-35B-A3B-Code`

## Gemma 4 26B A4B

1. Retrieve the [Gemma-4-26B-A4B.txt](models/Gemma-4-26B-A4B/Gemma-4-26B-A4B.txt) file

2. Download Unsloth gguf files
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/gemma-4-26B-A4B-it-UD-Q8_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/mtp-gemma-4-26B-A4B-it.gguf`

3. Verify the gguf files

`sha256sum *.gguf`

| File                               | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| gemma-4-26B-A4B-it-UD-Q8_K_XL.gguf | 50e180d69641e017d7e08a6f602988effde8232ff6bc0231e839636fdcc03d8f |
| mmproj-BF16.gguf                   | 41926ed5f1403cf5add23b0684992805ea6f97253096132e769e65646b8cef9d |
| mtp-gemma-4-26B-A4B-it.gguf        | 6326fb9f5e487aa8dcdd313a091e3c67724cb2a666ec3b7d2895b5b26d93ed1b |


4. Create the model with ollama
- `ollama create jetelain/Gemma-4-26B-A4B -f Gemma-4-26B-A4B.txt`

5. Publish the model
- `ollama push jetelain/Gemma-4-26B-A4B`

## Qwen3.6 27B

1. Retrieve the [Qwen3.6-27B.txt](models/Qwen3.6-27B/Qwen3.6-27B.txt) file

2. Download Unsloth gguf files
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF/resolve/main/Qwen3.6-27B-UD-Q4_K_XL.gguf`

3. Verify the gguf files

`sha256sum *.gguf`

| File                               | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| Qwen3.6-27B-UD-Q4_K_XL.gguf        | 4085665ee36d82a672a238a43f0e5643f2f0e39f2d7bd5d373f0ef10ecf53095 |
| mmproj-BF16.gguf                   | 05353347512982ee62317b9d8c89372bc815f4b4043580e7ef3ad411ec1a1cd3 |


4. Create the model with ollama
- `ollama create jetelain/Qwen3.6-27B -f Qwen3.6-27B.txt`

5. Publish the model
- `ollama push jetelain/Qwen3.6-27B`


## Gemma 4 31B

1. Retrieve the [Gemma-4-31B.txt](models/Gemma-4-31B/Gemma-4-31B.txt) file

2. Download Unsloth gguf files
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/gemma-4-31B-it-UD-Q4_K_XL.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/mmproj-BF16.gguf`
- `curl -LO https://huggingface.co/unsloth/gemma-4-31B-it-GGUF/resolve/main/mtp-gemma-4-31B-it.gguf `

3. Verify the gguf files

`sha256sum *.gguf`

| File                               | SHA256                                                           |
|------------------------------------|------------------------------------------------------------------|
| gemma-4-31B-it-UD-Q4_K_XL.gguf     | dd69c1566b0d559f303424e2b00e830837bb6c8b25e5baa08c75f11709bd41db |
| mmproj-BF16.gguf                   | 7a4601b12ec680f706a7e0c7e1f78f579b1db64d485a9e352ee87d4b9daa45e4 |
| mtp-gemma-4-31B-it.gguf            | 5ae8b0117bed601e8924c6305bd5b0585de361d51f0e77091bcb4252cf1f27de |


4. Create the model with ollama
- `ollama create jetelain/Gemma-4-31B -f Gemma-4-31B.txt`

5. Publish the model
- `ollama push jetelain/Gemma-4-31B`
