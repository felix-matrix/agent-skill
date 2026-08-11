1. RUST HEADLESS BROWSER: https://github.com/h4ckf0r0day/obscura 
2. COMPRESS TOKEN HEADROOM: https://github.com/headroomlabs-ai/headroom
3. REDUCE TOKEN ON DEV COMMANDS: https://github.com/rtk-ai/rtk

# INSTALLATION
npm install -g @agentmemory/agentmemory
npm install -g gitnexus
npm install -g @playwright/mcp

npm install -g agent-browser
agent-browser install --with-deps

npm install -g @playwright/cli
playwright-cli install-browser --with-deps

curl -fsSL https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.sh | bash

docker run -d --name obscura -p 127.0.0.1:9222:9222 h4ckf0r0day/obscura
docker run -d --name lightpanda -p 127.0.0.1:9222:9222 lightpanda/browser:nightly

curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

# RUNTIME

codebase-memory-mcp config set auto_index true
codebase-memory-mcp --ui=true --port=9749 (http://localhost:9749)

agentmemory

gitnexus analyze --embeddings --skip-git

headroom wrap claude

export ANTHROPIC_TARGET_API_URL=https://api.deepseek.com/anthropic
headroom proxy --port 8787
export ANTHROPIC_BASE_URL=http://127.0.0.1:8787  

ssh -L 3118:localhost:3118 fxbite@alienware-fxbite (forward port)

