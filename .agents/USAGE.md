1. RUST HEADLESS BROWSER: https://github.com/h4ckf0r0day/obscura 
2. COMPRESS TOKEN HEADROOM: https://github.com/headroomlabs-ai/headroom
3. REDUCE TOKEN ON DEV COMMANDS: https://github.com/rtk-ai/rtk

# INSTALLATION
npm install -g @agentmemory/agentmemory
npm install -g gitnexus
npm install -g agent-browser

docker run -d --name obscura -p 127.0.0.1:9222:9222 h4ckf0r0day/obscura
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

# RUNTIME

agentmemory

headroom wrap claude

gitnexus analyze --embeddings --skip-git

export ANTHROPIC_TARGET_API_URL=https://api.deepseek.com/anthropic
headroom proxy --port 8787
export ANTHROPIC_BASE_URL=http://127.0.0.1:8787