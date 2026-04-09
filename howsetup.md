OpenClaw Setup Guide
Part 1 — Install OpenClaw

Step 1. Open Terminal and run:

curl -fsSL https://openclaw.ai/install.sh | bash

This will:

    Detect your OS and Node.js version
    Install the latest OpenClaw via npm
    Run a setup check
    Try to open the dashboard in your browser

Requirements: Node.js 22+, macOS/Linux/WSL2

    Note (tested on macOS/zsh): After install you may see WARN openclaw is not discoverable on PATH in this shell. The installer may also silently fail to register the npm package. If rehash doesn't fix it, install manually. This is what I did in the tutorial:

    npm i -g openclaw

    Then verify:

    openclaw --version

    Then onboarding:

    openclaw onboard

Step 1b — Required: Set gateway mode

Before starting the gateway, you must set the mode or it will be blocked:

openclaw config set gateway.mode local

    Note (tested on macOS): If you skip this step, the gateway will fail to start with Gateway start blocked: set gateway.mode=local. This is not mentioned during install but is required.

Step 2. Start the gateway:

openclaw gateway install --force

    Note: The launchctl bootstrap command in the original docs may return Bootstrap failed: 5: Input/output error if the LaunchAgent is already registered. Use openclaw gateway install --force instead — it handles both install and start reliably.

Step 3. Verify it's running:

openclaw status

Look for: Gateway service: LaunchAgent installed · loaded · running

Step 4. Open the dashboard in your browser:

  openclaw dashboard

Bookmark this — it's your main control panel.
Part 2 — Keep OpenClaw Always Running

OpenClaw is registered as a macOS LaunchAgent, which means:

    It auto-starts on every login — no terminal needed
    It runs silently in the background
    It survives reboots automatically

You do not need to keep any window open. Just open the dashboard when you need it.

If it ever stops, restart it with:

openclaw gateway install --force

Part 3 — Set Up Ollama (Local AI, Free)

This lets you run AI models locally on your Mac — no internet, no API cost.

Step 1. Check that Ollama is installed:

ollama --version

If not installed, download it from https://ollama.ai

Step 2. Check what models you have:

ollama list

To pull a new model (example):

ollama pull llama3.2

Step 3. Open the OpenClaw config file:

~/.openclaw/openclaw.json

Step 4. Add the Ollama provider inside "models":

"models": {
  "providers": {
    "ollama": {
      "baseUrl": "http://127.0.0.1:11434",
      "apiKey": "ollama-local",
      "api": "ollama",
      "models": [
        {
          "id": "llama3.2:latest",
          "name": "llama3.2:latest",
          "reasoning": false,
          "input": ["text"],
          "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
          "contextWindow": 32768,
          "maxTokens": 32768
        }
      ]
    }
  }
}

    Note (tested on macOS): Set contextWindow to at least 16000 or OpenClaw will refuse to start the agent with: Model context window too small (8192 tokens). Minimum is 16000. Use 32768 for llama3.2 (3B) to keep it GPU-resident.

Add one block per model you want available.

Step 5. Also register each model under agents.defaults.models:

"agents": {
  "defaults": {
    "model": {
      "primary": "ollama/llama3.2:latest"
    }
  }
}

Step 6. Save the file, then restart the gateway:

openclaw gateway install --force

Step 7. Confirm it worked:

openclaw status

The Sessions line should now show your Ollama model if it's set as default.
Useful Commands
Command 	What it does
openclaw status 	Check gateway and active model
openclaw logs --follow 	Watch live logs
openclaw doctor 	Diagnose config issues
openclaw gateway install --force 	Restart the gateway
ollama list 	See available local models
ollama pull <model> 	Download a new local model
Troubleshooting
Problem 	Fix
Dashboard won't load 	Run openclaw gateway install --force
Config error on startup 	Run openclaw logs to see the bad field, fix the JSON, restart
Ollama models not available 	Run ollama list to confirm the model exists, then restart gateway
Gateway stopped after reboot 	It should auto-start; if not, run openclaw gateway install --force
Doctor failed (/dev/tty) 	Safe to ignore if gateway is reachable

Full docs: https://docs.openclaw.ai/troubleshooting
