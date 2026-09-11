# 📘 OpenCode

[OpenCode](https://opencode.ai/) is an open source AI coding agent. It is available as a terminal-based interface, desktop application, web application, or IDE extension.

While OpenCode can run directly on Windows, it is highly recommended to use Windows Subsystem for Linux (WSL) for the best experience. WSL provides a Linux environment that integrates seamlessly with OpenCode's features.

## 🔹 Step 1: WSL2 Installation and Configuration

Based on [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)

1. **Install WSL** from an administrator terminal

```pwsh
wsl.exe --install -d Ubuntu
```

2. **Verify WSL is working** from a standard terminal

The following command should switch the terminal to WSL2

```pwsh
wsl.exe
```

3. **Ensure Ubuntu is up to date**

```bash
sudo apt update && sudo apt dist-upgrade -y
```

## 🔹 Step 2: OpenCode Installation

1. **Install OpenCode** from a WSL2 terminal

```bash
curl -fsSL https://opencode.ai/install | bash
```

2. **Edit the configuration file**

```bash
nano ~/.config/opencode/opencode.json
```

Content to paste (make sure to specify the server's IP address or its [Tailscale domain name](tailscale.md))
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
                        "context": 196608,
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
                        "context": 196608,
                        "output": 65536
                    },
                    "reasoning": true,
                    "name": "Qwen 3.8"
                }
            },
            "name": "Ollama",
            "npm": "@ai-sdk/openai-compatible",
            "options": {
                "baseURL": "http://[server IP or Tailscale domain name]:11434/v1"
            }
        }
    }
}
```
