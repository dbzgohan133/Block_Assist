# Block_Assist
This works for me for a specific gpu 3090 from vast ai and cuda 12.8 

# STEP:1 Installation
```
sudo apt update
sudo apt install -y \
  make build-essential gcc \
  libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
  libsqlite3-dev libncursesw5-dev xz-utils tk-dev \
  libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev \
  curl git unzip \
  libxi6 libxrender1 libxtst6 libxrandr2 libglu1-mesa libopenal1
```

# Step 2: Clone the repo and enter the directory
```
git clone https://github.com/gensyn-ai/blockassist.git
cd blockassist
```

# Step 3: Install Node.js
```
# Verify version
# Skip the whole step if version 20
node --version
npm --version

# Remove existing Node.js
sudo apt remove -y nodejs npm

# Install NodeSource repository for Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js 20
sudo apt install -y nodejs
```

# Step 4: Install Yarn
```
curl -o- -L https://yarnpkg.com/install.sh | bash
export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
```

# Step 5: Install Java
```
./setup.sh
exec $SHELL
```

# Step 6: Install pyenv
```
curl https://pyenv.run | bash
```

```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

# Step 7: Install Python
```
pyenv install 3.10
pyenv global 3.10
```

# Step 8: Install project dependecies
```
pip install --upgrade pip

pip install -e . --no-cache-dir
pip install "mbag-gensyn[malmo]" --no-cache-dir

pip install psutil readchar
```

# Step 9: Run BlockAssist
```
cd blockassist

export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"

python run.py
```

# Problem 1 – Node.js version mismatch
Error text
error js-cookie@3.0.5: The engine "node" is incompatible … Got "12.22.9"
Cause Ubuntu image ships with Node 12; BlockAssist packages require ≥14.
Fix
```
sudo apt remove -y nodejs npm          # purge old v12
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs             # installs v20+
node -v && npm -v
# reinstall yarn
curl -o- -L https://yarnpkg.com/install.sh | bash
export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"

```


#  Problem 2 – Script name typo
Error text
/usr/bin/python: can't open file 'run.py#': [Errno 2] No such file or directory
Cause An accidental trailing # made Python look for a non-existent file.
Fix
```
cd ~/blockassist
python run.py          # run without the hash

```

# Problem 3 – Java missing (Malmo cannot launch)
Error text
FileNotFoundError: ... java or Command 'java' not found
Cause Java Runtime not installed; Minecraft needs it.
Fix
```
sudo apt update
sudo apt install -y openjdk-8-jdk openjdk-8-jre   # Java 8 recommended
JAVA_PATH=$(readlink -f /usr/bin/java | sed 's:/bin/java::')
echo "export JAVA_HOME=$JAVA_PATH" >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
java -version

```

# Problem 4 – 401 Unauthorized during model upload
Error text
requests.exceptions.HTTPError: 401 Client Error: Unauthorized in blockassist-train.log
Cause No valid write-scoped Hugging Face token in the environment.
Fix
```
# 1. Create a Write token at https://huggingface.co/settings/tokens
export HF_TOKEN=hf_your_write_token_here

# 2. Log in and verify
huggingface-cli login --token "$HF_TOKEN"
huggingface-cli whoami

# 3. Let BlockAssist inherit it
echo "HF_TOKEN=$HF_TOKEN" > ~/blockassist/.env

```

# Problem 5 – Stuck at “START MINECRAFT … press ENTER when two windows have opened”
Error text No windows appear after several minutes.
Cause First-run download or display not ready (normal, not a crash).
Fix / diagnostics
```
# Watch real-time Malmo logs
tail -f ~/blockassist/logs/malmo.log
# If you see heap-size errors, relaunch with less RAM
export _JAVA_OPTIONS='-Xmx2G'
python run.py
# After both Minecraft windows are stable, return to the main terminal and press ENTER.

```


# CUDNNN ERROR
```
wget https://developer.download.nvidia.com/compute/cudnn/9.11.0/local_installers/cudnn-local-repo-ubuntu2204-9.11.0_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2204-9.11.0_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2204-9.11.0/cudnn-local-4EC753EA-keyring.gpg /usr/share/keyrings/
echo "deb [signed-by=/usr/share/keyrings/cudnn-local-4EC753EA-keyring.gpg] file:///var/cudnn-local-repo-ubuntu2204-9.11.0 /" | sudo tee /etc/apt/sources.list.d/cudnn-local.list
sudo apt update
sudo apt install -y libcudnn9 libcudnn9-dev
```

# If Instance Crashing
```
# 2.1 Make sure broken episodes can’t crash the trainer
rm -rf data/evaluation/*            # wipe any old junk
mkdir -p data/evaluation            # folder must exist

# 2.2 Supply a WRITE-scoped Hugging Face token
echo 'HF_TOKEN=hf_your_write_token_here' > .env
grep HF_TOKEN .env                  # should print the token line

```

# Environment prep (your usual five lines)
```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

# Launch BlockAssist
```
python run.py
```
