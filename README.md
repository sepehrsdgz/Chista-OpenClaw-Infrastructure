# Chista: OpenClaw Infrastructure & AI Assistant 

> **Status:** Active / Production
> **Role:** High-availability, self-hosted multi-model AI executive assistant.

This repository outlines the architecture, deployment pipeline, and security posture of **Chista**, a highly customized instance of the OpenClaw AI assistant framework. 

*Note: For security reasons, this repository contains sanitized architecture documentation and deployment templates only. Active state files and environment variables are strictly excluded.*

## 🏗️ Infrastructure & Hardware
* **Host:** Oracle Cloud Infrastructure (OCI) - Ampere A1 Compute 
* **Compute:** 4 OCPUs, 24GB RAM
* **Environment:** Ubuntu Linux, Node.js, Docker

## 🧠 Multi-Model Fallback Chain
To ensure maximum uptime and intelligent routing, the agent utilizes a dynamic fallback chain across multiple providers:
1. **Primary:** `Google Gemini 3 Flash` (Speed & Context)
2. **Fallback 1:** `Moonshot Kimi K2.5` via Nvidia NIM 
3. **Fallback 2:** `Moonshot Kimi K2.5` via Direct API
4. **Fallback 3:** `Llama 3.3 70B Versatile` via Groq
5. **Fallback 4:** `Mixtral 8x7B` via Groq

## 🛡️ Security Posture (Hardened Baseline)
Chista operates under a strict, hardened security baseline designed to prevent unauthorized access and context bleeding:
* **Network Isolation:** The OpenClaw gateway is bound strictly to `loopback (127.0.0.1)`. 
* **Perimeter Defense:** The Control UI is completely inaccessible from the public internet. Access is granted exclusively via SSH Port Forwarding (`Port 18789`).
* **Secrets Management:** Zero hardcoded API keys. All credentials and tokens are dynamically injected into the container at runtime using **1Password CLI (`op`)**.
* **Sandboxing:** The agent runs inside a strict Docker sandbox (`mode: all`) with elevated tools disabled. Filesystem access is strictly limited to the designated workspace.
* **Access Control:** Deployed via Telegram with a strict `pairing` DM policy. Group access is disabled, and session scopes are set to `per-channel-peer` to maintain strict context isolation.

## 🚀 Deployment Pipeline
Updates to the blueprint (`openclaw.tpl.json`) are pushed to production bypassing file permission locks and injecting secure secrets via the following automated sequence:

```bash
cd ~/OpenClaw
op inject -i openclaw.tpl.json | sudo tee config/openclaw.json > /dev/null
sudo chown 1000:1000 config/openclaw.json
sudo chmod 600 config/openclaw.json
docker compose restart openclaw-gateway