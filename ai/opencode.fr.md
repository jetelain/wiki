# 📘 OpenCode

[OpenCode](https://opencode.ai/) est un agent de codage IA open source. Il est disponible sous forme d’interface basée sur un terminal, d’application de bureau, d'application web ou d’extension IDE.

Même si OpenCode peut fonctionner directement sur Windows, il est fortement recommandé d’utiliser Windows Subsystem for Linux (WSL) pour une meilleure expérience. WSL fournit un environnement Linux qui s’intègre parfaitement aux fonctionnalités d’OpenCode.

## 🔹 Étape 1 : Installation et configuration WSL2

Basé sur [Comment installer Linux sur Windows avec WSL](https://learn.microsoft.com/fr-fr/windows/wsl/install)

1. **Installer WSL** depuis un terminal en tant qu'administrateur

```pwsh
wsl.exe --install -d Ubuntu
```

2. **Vérifier le fonctionnement de WSL** depuis un terminal standard

La commande suivante devrait basculer le terminal dans WSL2

```pwsh
wsl.exe
```

3. **Vérifier que Ubuntu est à jour**

```bash
sudo apt update && sudo apt dist-upgrade -y
```

## 🔹 Étape 2 : Installation OpenCode

1. **Installer OpenCode** depuis un terminal WSL2

```bash
curl -fsSL https://opencode.ai/install | bash
```

2. **Modifier le fichier de configuration**

```bash
nano ~/.config/opencode/opencode.json
```

Contenu à coller (penser à préciser l'adresse IP du serveur ou son [nom de domaine Tailscale](tailscale.fr.md))
```json
{
    "$schema": "https://opencode.ai/config.json",
    "model": "ollama/Qwen3.6-35B-A3B-Code:latest",
    "provider": {
        "ollama": {
            "models": {
                "Qwen3.6-35B-A3B-Code:latest": {
                    "modalities": {
                        "input": [
                            "text",
                            "image"
                        ],
                        "output": [
                            "text"
                        ]
                    },
                    "limit": {
                        "context": 262144,
                        "output": 65536
                    },
                    "reasoning": true,
                    "name": "Qwen 3.6"
                },
                "Qwen3.8-27B:latest": {
                    "modalities": {
                        "input": [
                            "text",
                            "image"
                        ],
                        "output": [
                            "text"
                        ]
                    },
                    "limit": {
                        "context": 262144,
                        "output": 65536
                    },
                    "reasoning": true,
                    "name": "Qwen 3.8"
                }
            },
            "name": "Ollama",
            "npm": "@ai-sdk/openai-compatible",
            "options": {
                "baseURL": "http://[ip du serveur ou nom de domaine tailscale vers le serveur]:11434/v1"
            }
        }
    }
}
```