# Common Dev Environment Setup — MacBook M1 Pro
> Shared across all 4 projects. Do this ONCE before starting any project.

---

## System Prerequisites

```bash
# 1. Xcode Command Line Tools
xcode-select --install

# 2. Homebrew (M1 native — installs to /opt/homebrew)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

---

## Terminal

```bash
brew install --cask iterm2
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

## Git + GitHub SSH

```bash
brew install git
git config --global user.name "Amisha Joshipura"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"

# SSH key
ssh-keygen -t ed25519 -C "your@email.com"   # press Enter 3 times
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
pbcopy < ~/.ssh/id_ed25519.pub
# → Paste at: github.com → Settings → SSH and GPG Keys → New SSH Key

ssh -T git@github.com   # verify: "Hi Amisha! You've successfully authenticated."
```

---

## Python (via pyenv)

```bash
brew install pyenv

# Add to ~/.zshrc
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc

# Install Python 3.11
pyenv install 3.11.9
pyenv global 3.11.9
python --version   # Python 3.11.9
```

**Virtual env — do this inside EVERY project folder:**
```bash
python -m venv venv
source venv/bin/activate   # (venv) appears in prompt
pip install --upgrade pip
```

---

## VS Code + Extensions

```bash
brew install --cask visual-studio-code
# Enable 'code' command: Cmd+Shift+P → "Shell Command: Install 'code' command"

code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.black-formatter
code --install-extension charliermarsh.ruff
code --install-extension ms-toolsai.jupyter
code --install-extension eamodio.gitlens
code --install-extension mhutchie.git-graph
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode-remote.remote-containers
code --install-extension humao.rest-client
code --install-extension PKief.material-icon-theme
```

**VS Code settings (Cmd+Shift+P → Open User Settings JSON):**
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "ms-python.black-formatter",
  "editor.fontSize": 14,
  "editor.tabSize": 4,
  "editor.wordWrap": "on",
  "files.autoSave": "afterDelay",
  "git.autofetch": true
}
```

---

## Docker Desktop

```bash
brew install --cask docker
# Open Docker Desktop from Applications and complete setup
docker --version   # verify
```

---

## Node.js (via nvm)

```bash
brew install nvm
mkdir ~/.nvm
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
source ~/.zshrc

nvm install 20
nvm use 20
nvm alias default 20
node --version   # v20.x.x
```

---

## PostgreSQL 16

```bash
brew install postgresql@16
brew services start postgresql@16
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
createdb $(whoami)   # create default DB for your user
psql --version       # PostgreSQL 16.x
```

---

## Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
# Launch inside any project: claude
# Requires Claude Pro or Max subscription
```

---

## AWS CLI + Terraform

```bash
brew install awscli
aws configure   # enter Access Key, Secret, region: ap-south-1, format: json

brew install tfenv
tfenv install 1.7.5
tfenv use 1.7.5
terraform --version
```

---

## Global .gitignore (run once)

```bash
echo ".env"             >> ~/.gitignore_global
echo ".env.local"       >> ~/.gitignore_global
echo "venv/"            >> ~/.gitignore_global
echo "__pycache__/"     >> ~/.gitignore_global
echo "*.pyc"            >> ~/.gitignore_global
echo ".DS_Store"        >> ~/.gitignore_global
echo ".ipynb_checkpoints/" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

---

## Folder Structure

```bash
mkdir -p ~/dev/{interview-prep,learning/{dsa,system-design,ai-ml/{fastai,huggingface}}}
mkdir -p ~/dev/projects/{ai-rag-assistant,devflow,ai-pm-copilot,manupilot}
```

```
~/dev/
├── interview-prep/         # DSA solutions, SD notes, AI/ML notes
│   ├── dsa/
│   ├── system-design/
│   ├── ai-ml/
│   └── setup/              # ← this file + all project setup docs
├── projects/               # The 4 live projects
│   ├── ai-rag-assistant/
│   ├── devflow/
│   ├── ai-pm-copilot/
│   └── manupilot/
└── learning/               # Course notebooks (fast.ai, HuggingFace)
    └── ai-ml/
        ├── fastai/
        └── huggingface/
```

---

## Verification — Run This Before Starting Any Project

```bash
git --version && python --version && node --version && \
docker --version && aws --version && terraform --version && \
claude --version && code --version && \
echo "✅ All tools ready"
```
