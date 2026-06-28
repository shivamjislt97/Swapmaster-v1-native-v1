# SwapMaster V1 Native — Complete Master Setup Prompt

## How to Use This Prompt
1. Open this file in any text editor
2. Copy EVERYTHING between `--- PROMPT START ---` and `--- PROMPT END ---`
3. Paste it into any AI assistant (ChatGPT, Claude, Gemini, Cursor, etc.)
4. The AI will execute every step automatically
5. You only need to provide your credentials when asked

---

## --- PROMPT START ---

I have a SwapMaster V1 Native project (FaceFusion Telegram Bot) backup on Google Drive.
I need you to set it up completely on a new Lightning AI studio (Ubuntu, CUDA GPU).

**FOLLOW EVERY STEP. DO NOT SKIP ANYTHING. DO NOT ASK ME QUESTIONS — DO IT YOURSELF.**

### PROJECT INFO
- **GitHub Repo:** https://github.com/shivamjislt97/Swapmaster-v1-native-v1
- **Project Path on Machine:** /teamspace/studios/this_studio/swapmaster-v1-native
- **GPU Required:** NVIDIA with CUDA (tested on L40S 46GB)
- **Python Version:** 3.12+

---

### STEP 1: CHECK MACHINE ENVIRONMENT
Run this first to understand what GPU and resources are available:
```bash
nvidia-smi
python3 --version
df -h /
free -h
uname -a
```
Report: GPU name, VRAM total, Python version, disk free, RAM free.

---

### STEP 2: DOWNLOAD AND EXTRACT BACKUP FROM GDRIVE
The complete project backup (5.5GB) is on Google Drive. Download and extract it:
```bash
cd /teamspace/studios/this_studio

# Download backup from GDrive
curl -L -o swapmaster.tar "https://drive.google.com/uc?id=YOUR_GDRIVE_BACKUP_FILE_ID"

# Extract
tar xf swapmaster.tar

# Verify extraction
ls -la swapmaster-v1-native/
ls -la swapmaster-v1-native/app/
ls -la swapmaster-v1-native/app/bot.py

# Enter project directory
cd /teamspace/studios/this_studio/swapmaster-v1-native
```

**If the GDrive download fails (quota limit), use rclone instead:**
```bash
# Install rclone first (see Step 4), then:
rclone copy gdrive:swapmaster-v1-native-v1/swapmaster-v1-native-v1.tar.gz /tmp/ --config /teamspace/studios/this_studio/swapmaster-v1-native/.config/rclone/rclone.conf
cd /teamspace/studios/this_studio
tar xf /tmp/swapmaster-v1-native-v1.tar.gz
cd swapmaster-v1-native
```

---

### STEP 3: ADD ~/.local/bin TO PATH (permanent + current session)
```bash
export PATH="$HOME/.local/bin:$PATH"
grep -q '.local/bin' ~/.bashrc 2>/dev/null || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc 2>/dev/null
```

---

### STEP 4: INSTALL FFMPEG STATIC BUILD (with libx264 support)
The system FFmpeg does NOT have libx264. You MUST install the static build:
```bash
mkdir -p ~/.local/bin
cd /tmp
curl -L -o ffmpeg.tar.xz https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
tar xf ffmpeg.tar.xz
cp ffmpeg-*-static/ffmpeg ~/.local/bin/
cp ffmpeg-*-static/ffprobe ~/.local/bin/
chmod +x ~/.local/bin/ffmpeg ~/.local/bin/ffprobe
rm -rf ffmpeg*

# Verify libx264 encoder is available
~/.local/bin/ffmpeg -encoders 2>/dev/null | grep libx264
# MUST show: libx264 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
```

---

### STEP 5: INSTALL RCLONE (for Google Drive upload)
```bash
mkdir -p ~/.local/bin
curl -L -o /tmp/rclone.zip https://downloads.rclone.org/current/rclone-v1.74.3-linux-amd64.zip
cd /tmp && unzip -o rclone.zip && cp rclone-*-linux-amd64/rclone ~/.local/bin/
chmod +x ~/.local/bin/rclone
rm -rf /tmp/rclone*
cd /teamspace/studios/this_studio/swapmaster-v1-native

# Verify
rclone version
```

---

### STEP 6: INSTALL NGROK (for dashboard public URL)
```bash
curl -s https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz | tar xz -C ~/.local/bin/
chmod +x ~/.local/bin/ngrok

# Configure with authtoken
ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN
```

---

### STEP 7: INSTALL PYTHON DEPENDENCIES
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native
pip install --upgrade pip

# Core dependencies
pip install python-telegram-bot==20.7 pycryptodome==3.23.0

# AI / FaceFusion
pip install onnxruntime-gpu==1.19.2 onnx==1.21.0 insightface==0.7.3
pip install opencv-python-headless==4.13.0.92 numpy==2.4.6 pillow==11.3.0
pip install scikit-image==0.26.0 scipy==1.17.1

# Web Dashboard
pip install fastapi==0.135.1 uvicorn==0.42.0 aiofiles==24.1.0

# File handling
pip install mega.py==1.0.8 gdown==6.0.0 requests==2.32.5 beautifulsoup4==4.14.3

# Utilities
pip install psutil==7.2.2 loguru==0.7.3 python-dotenv==1.2.2 tqdm==4.67.3 colorama==0.4.6

# CUDA (PyTorch with GPU support)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
```

---

### STEP 8: VERIFY CUDA AND ONNX WORKS
```bash
python3 -c "
import torch
print(f'PyTorch: {torch.__version__}')
print(f'CUDA available: {torch.cuda.is_available()}')
if torch.cuda.is_available():
    print(f'GPU: {torch.cuda.get_device_name(0)}')
    print(f'VRAM: {torch.cuda.get_device_properties(0).total_mem / 1024**3:.1f}GB')
import onnxruntime as ort
print(f'ONNX Runtime: {ort.__version__}')
print(f'Providers: {ort.get_available_providers()}')
"
```
**Expected output:** CUDAExecutionProvider must be in the providers list.
**If NOT available:** `pip install onnxruntime-gpu==1.19.2 --force-reinstall`

---

### STEP 9: CREATE .env FILE WITH CREDENTIALS
The backup contains a `.env.example` template. Copy it and fill in credentials:
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native
cp .env.example .env

# Write .env with actual credentials
cat > .env << 'ENVEOF'
# === SwapMaster V1 - Native Installation ===
# === REQUIRED ===
BOT_TOKEN=YOUR_BOT_TOKEN_FROM_BOTFATHER
ALLOWED_USER_ID=YOUR_TELEGRAM_USER_ID

# === MEGA (for downloading user-sent MEGA links) ===
MEGA_EMAIL=YOUR_MEGA_EMAIL
MEGA_PASSWORD=YOUR_MEGA_PASSWORD

# === GDRIVE UPLOAD (rclone) ===
GDRIVE_ENABLED=true
GDRIVE_REMOTE_NAME=gdrive
GDRIVE_FOLDER=gdrive:faceswap_output
RCLONE_BIN=rclone
RCLONE_CONF=/teamspace/studios/this_studio/swapmaster-v1-native/.config/rclone/rclone.conf

# === GPU / FACEFUSION ===
EXECUTION_PROVIDER=cuda
GPU_ONLY_MODE=true
FACE_SWAPPER_MODEL=inswapper_128
FACE_ENHANCER_MODEL=gfpgan_1.4
FACE_ENHANCER_BLEND=80
ENABLE_FACE_ENHANCER=true
OUTPUT_VIDEO_ENCODER=libx264
EXECUTION_THREAD_COUNT=6
THREAD_COUNT=6

# === BOT BEHAVIOUR ===
AUTO_SLEEP_ENABLED=true
AUTO_SLEEP_MINUTES=30
POST_JOB_AUTO_SLEEP_SECONDS=120
DASHBOARD_ENABLED=true
DASHBOARD_PORT=8765
BYPASS_CONTENT_ANALYSER=false

# === WATCHDOG TIMEOUTS ===
PIPELINE_WATCHDOG_PROCESSING_SEC=600
PIPELINE_WATCHDOG_MERGING_SEC=300
PIPELINE_WATCHDOG_UPLOADING_SEC=300
ENVEOF
```

---

### STEP 10: CONFIGURE RCLONE FOR GOOGLE DRIVE
The backup includes rclone config but the token may be expired. Refresh it:
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native

# Write rclone config
mkdir -p .config/rclone
cat > .config/rclone/rclone.conf << 'RCLONEEOF'
[gdrive]
type = drive
scope = drive
token = {"access_token":"YOUR_GDRIVE_ACCESS_TOKEN","token_type":"Bearer","refresh_token":"YOUR_GDRIVE_REFRESH_TOKEN","expiry":"2026-06-24T06:19:45.5910606Z","expires_in":3599}
RCLONEEOF

# Refresh the token if expired (the bot does this automatically, but for manual testing):
curl -s -X POST https://oauth2.googleapis.com/token \
  -d "client_id=YOUR_GDRIVE_CLIENT_ID" \
  -d "client_secret=YOUR_GDRIVE_CLIENT_SECRET" \
  -d "refresh_token=YOUR_GDRIVE_REFRESH_TOKEN" \
  -d "grant_type=refresh_token" > /tmp/gdrive_token.json

# Extract new access token and update rclone.conf
NEW_TOKEN=$(python3 -c "import json; d=json.load(open('/tmp/gdrive_token.json')); print(d.get('access_token',''))")
EXPIRES=$(python3 -c "import json; d=json.load(open('/tmp/gdrive_token.json')); print(d.get('expires_in',3599))")

if [ -n "$NEW_TOKEN" ]; then
  cat > .config/rclone/rclone.conf << EOF
[gdrive]
type = drive
scope = drive
token = {"access_token":"$NEW_TOKEN","token_type":"Bearer","refresh_token":"YOUR_GDRIVE_REFRESH_TOKEN","expires_in":$EXPIRES}
EOF
  echo "Rclone token refreshed successfully"
fi

# Test rclone connection
rclone lsd gdrive: --config .config/rclone/rclone.conf
```

---

### STEP 11: VERIFY VIDEO ENCODER WORKS
```bash
~/.local/bin/ffmpeg -encoders 2>/dev/null | grep -E "libx264|h264_qsv"
```
- If `libx264` found → keep `OUTPUT_VIDEO_ENCODER=libx264` in .env
- If `h264_qsv` found → change to `OUTPUT_VIDEO_ENCODER=h264_qsv` in .env

---

### STEP 12: VERIFY GPU AUTO-DETECT CONFIGURATION
The backup includes `app/ops/gpu_auto_detect.py` which auto-detects GPU and sets optimal settings:
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native
python3 app/ops/gpu_auto_detect.py
```
Expected output:
```
[GPU-DETECT] Found: NVIDIA L40S | VRAM: 46068MB
[GPU-DETECT] High VRAM (46068MB) -> hyperswap_1a_256, 6 threads
[GPU-DETECT] Final: PROVIDER=cuda | MODEL=hyperswap_1a_256 | THREADS=6
```

---

### STEP 13: VERIFY PERSISTENT CONFIG
The backup includes `app/persistent/config.json` with GDrive tokens. Verify it exists:
```bash
cat app/persistent/config.json | python3 -m json.tool
```
If missing or corrupt, recreate:
```bash
cat > app/persistent/config.json << 'JSONEOF'
{
  "mega_email": "YOUR_MEGA_EMAIL",
  "mega_password": "YOUR_MEGA_PASSWORD",
  "drive_auth_token": {
    "access_token": "",
    "token_type": "Bearer",
    "refresh_token": "YOUR_GDRIVE_REFRESH_TOKEN",
    "expires_in": 3599
  },
  "chat_modes": {},
  "face_selector_prefs": {
    "YOUR_TELEGRAM_USER_ID": {
      "gender_mode": "female"
    }
  },
  "clip_ranges": {}
}
JSONEOF
```

---

### STEP 14: KILL ANY EXISTING BOT PROCESSES
Before starting fresh, kill any running instances:
```bash
ps aux | grep -E "process_guard|bot.py" | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null
sleep 2
echo "Old processes killed"
```

---

### STEP 15: START THE BOT
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native
export PATH="$HOME/.local/bin:$PATH"

# Start with auto-restart watchdog
nohup python3 app/ops/process_guard.py --max-backoff 120 > /tmp/swapmaster-startup.log 2>&1 &
echo "Bot started with PID: $!"
sleep 5

# Check if running
ps aux | grep -E "process_guard|bot.py" | grep -v grep
```

---

### STEP 16: VERIFY BOT IS HEALTHY
```bash
# Check bot startup log
tail -30 /teamspace/studios/this_studio/swapmaster-v1-native/app/pipeline/logs/bot_runtime.log

# Expected output should include:
# [INFO] Bot v14 ready | UID: YOUR_TELEGRAM_USER_ID | provider=cuda | exec_threads=6
# [INFO] [BOT_ONLINE] chat=YOUR_TELEGRAM_USER_ID
# [INFO] Application started

# Check GPU usage
nvidia-smi

# Check health endpoint
curl -s http://localhost:8765/healthz 2>/dev/null || echo "Dashboard starting..."
```

---

### STEP 17: START NGROK TUNNEL (for dashboard access)
```bash
# Start ngrok in background
nohup ~/.local/bin/ngrok http 8765 > /tmp/ngrok.log 2>&1 &
sleep 3

# Get public URL
NGROK_URL=$(curl -s http://localhost:4040/api/tunnels 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin)['tunnels'][0]['public_url'])" 2>/dev/null)
echo "Dashboard URL: $NGROK_URL"

# Update .env with dashboard URL
if [ -n "$NGROK_URL" ]; then
  sed -i "s|DASHBOARD_PUBLIC_URL=.*|DASHBOARD_PUBLIC_URL=$NGROK_URL|" .env
  echo "Dashboard URL saved to .env"
fi
```

---

### STEP 18: TEST THE BOT ON TELEGRAM
Open Telegram and:
1. Search for your bot (use the bot token to find it)
2. Send `/start`
3. Send a photo for face swap test
4. Check that it processes and uploads to GDrive

---

### STEP 19: SET UP GITHUB REPOSITORY (Optional)
If you want to push this to your own GitHub:
```bash
cd /teamspace/studios/this_studio/swapmaster-v1-native
git init
git config user.email "your@email.com"
git config user.name "Your Name"

# Use the .gitignore from backup (already configured)
git add -A
git commit -m "SwapMaster V1 Native - Setup from backup"
git remote add origin https://github.com/YOUR_USER/YOUR_REPO.git
git push -u origin master
```

---

### STEP 20: UPLOAD BACKUP TO GDRIVE (Optional)
To create a fresh backup after setup:
```bash
cd /teamspace/studios/this_studio
tar czf /tmp/swapmaster-v1-native-v1.tar.gz \
  --exclude='swapmaster-v1-native/__pycache__' \
  --exclude='swapmaster-v1-native/app/__pycache__' \
  --exclude='swapmaster-v1-native/app/facefusion/__pycache__' \
  --exclude='swapmaster-v1-native/app/facefusion/facefusion/__pycache__' \
  --exclude='swapmaster-v1-native/app/ops/__pycache__' \
  --exclude='swapmaster-v1-native/app/pipeline/workspace' \
  --exclude='swapmaster-v1-native/app/pipeline/downloads' \
  --exclude='swapmaster-v1-native/app/faceswap_output' \
  --exclude='swapmaster-v1-native/.ngrok' \
  --exclude='swapmaster-v1-native/.local' \
  --exclude='*.pyc' --exclude='*.dat' --exclude='*.log' \
  swapmaster-v1-native/

rclone mkdir gdrive:swapmaster-v1-native-v1 --config /teamspace/studios/this_studio/swapmaster-v1-native/.config/rclone/rclone.conf
rclone copy /tmp/swapmaster-v1-native-v1.tar.gz gdrive:swapmaster-v1-native-v1/ --config /teamspace/studios/this_studio/swapmaster-v1-native/.config/rclone/rclone.conf -v
```

---

## KEY CONFIGURATION VALUES

| Setting | Value | Notes |
|---------|-------|-------|
| BOT_TOKEN | `YOUR_BOT_TOKEN_FROM_BOTFATHER` | From @BotFather |
| ALLOWED_USER_ID | `YOUR_TELEGRAM_USER_ID` | Your Telegram user ID |
| MEGA_EMAIL | `YOUR_MEGA_EMAIL` | For MEGA link downloads |
| MEGA_PASSWORD | `YOUR_MEGA_PASSWORD` | MEGA password |
| GDrive Refresh Token | `YOUR_GDRIVE_REFRESH_TOKEN` | Auto-refreshed by bot |
| GDrive Client ID | `YOUR_GDRIVE_CLIENT_ID` | OAuth client |
| GDrive Client Secret | `YOUR_GDRIVE_CLIENT_SECRET` | OAuth secret |
| ngrok Authtoken | `YOUR_NGROK_AUTHTOKEN` | Dashboard tunnel |
| GDrive Backup File ID | `YOUR_GDRIVE_BACKUP_FILE_ID` | Backup on GDrive |
| GitHub Repo | `https://github.com/shivamjislt97/Swapmaster-v1-native-v1` | Source code |
| Project Path | `/teamspace/studios/this_studio/swapmaster-v1-native` | On Lightning AI |

---

## GPU AUTO-DETECT RULES

| GPU VRAM | Face Swap Model | Threads | Enhancer |
|----------|----------------|---------|----------|
| ≥ 20 GB (L40S, A100, 4090) | `hyperswap_1a_256` | 6 | ON |
| ≥ 12 GB (3080, 4070Ti) | `hyperswap_1a_256` | 4 | ON |
| ≥ 6 GB (3060, 4060) | `inswapper_128_fp16` | 4 | ON |
| < 6 GB (GTX 1650) | `inswapper_128_fp16` | 2 | OFF |
| No GPU | `inswapper_128_fp16` | 4 | ON (CPU) |

---

## CRITICAL RULES

1. **ALWAYS use `~/.local/bin/ffmpeg`** — system ffmpeg does NOT have libx264
2. **ALWAYS set `OUTPUT_VIDEO_ENCODER=libx264`** — CUDA-compatible encoder
3. **ALWAYS set `ORT_CUDA_GRAPH=0`** — prevents crashes with hyperswap model
4. **ALWAYS set `ORT_CUDA_USE_UNIFIED_MEMORY=0`** — reduces VRAM usage
5. **NEVER commit `.env` or `rclone.conf` to git** — they contain secrets
6. **The bot auto-refreshes GDrive token** before every upload — don't worry about token expiry
7. **Output filenames include timestamp** (`_DDMMYY_HHMMSS`) — prevents overwrites
8. **Bot auto-sleeps** after 120 seconds of inactivity — saves GPU credits
9. **process_guard.py** auto-restarts bot if it crashes — run it instead of bot.py directly
10. **Both `rclone.conf` AND `app/persistent/config.json`** must have the same GDrive token

---

## TROUBLESHOOTING

| Problem | Solution |
|---------|----------|
| `CUDAExecutionProvider not available` | `pip install onnxruntime-gpu==1.19.2 --force-reinstall` |
| `libx264 not found` | Install static ffmpeg (Step 4) |
| `CUDA graph capture failed` | Set `ORT_CUDA_GRAPH=0` in `app/ops/gpu_auto_detect.py` |
| `GDrive upload 401` | Token expired — bot auto-refreshes, or manually refresh (Step 10) |
| Bot crashes on start | Check `.env` has correct `BOT_TOKEN` and `ALLOWED_USER_ID` |
| `Permission denied` on GitHub | Token needs `Contents: Write` scope |
| Dashboard not accessible | Start ngrok tunnel (Step 17) |
| `ffmpeg: command not found` | Add `~/.local/bin` to PATH (Step 3) |
| Bot not responding on Telegram | Check bot logs, verify BOT_TOKEN is correct |
| GPU not detected | Run `nvidia-smi`, check CUDA drivers installed |

---

## FILE STRUCTURE

```
swapmaster-v1-native/
├── .env                          # Main config (with credentials)
├── .env.example                  # Template (no credentials)
├── .config/rclone/rclone.conf    # GDrive OAuth tokens
├── app/
│   ├── bot.py                    # Main Telegram bot (13000+ lines)
│   ├── startup.py                # Startup orchestrator
│   ├── auto_repair.py            # Auto-repair utility
│   ├── health_check.py           # Health check endpoint
│   ├── ops/
│   │   ├── process_guard.py      # Auto-restart watchdog (RUN THIS)
│   │   ├── gpu_auto_detect.py    # GPU detection & config
│   │   ├── auto_sleep_manager.py # Auto-sleep after inactivity
│   │   ├── dashboard_server.py   # Web dashboard
│   │   ├── health_monitor.py     # Health monitoring
│   │   ├── job_worker.py         # Job processing
│   │   └── progress_poller.py    # Progress tracking
│   ├── facefusion/               # FaceFusion AI engine
│   │   ├── facefusion.py         # Entry point
│   │   └── facefusion/           # Engine code
│   ├── pipeline/                 # Pipeline data & logs
│   │   ├── logs/                 # Runtime logs
│   │   ├── workspace/            # Processing workspace
│   │   └── downloads/            # Downloaded files
│   └── persistent/
│       └── config.json           # Persistent state (GDrive tokens)
├── requirements.txt              # Python dependencies
├── setup_auto.sh                 # Auto-setup script
├── README.md                     # Documentation
├── Settings Used in This Project.md
├── GPU Configuration Knowledge.md
└── MASTER_SETUP_PROMPT.md        # This file
```

---

## WHAT THE BOT DOES

1. **User sends photo/video** to Telegram bot
2. **Bot processes** face swap using FaceFusion + CUDA GPU
3. **Output saved** with timestamp filename
4. **Auto-uploaded** to Google Drive with shareable link
5. **Link sent back** to user on Telegram
6. **Auto-sleep** after 120 seconds of inactivity
7. **Auto-restart** if bot crashes (via process_guard.py)

---

## --- PROMPT END ---

---

## How to Use

1. Copy everything between `--- PROMPT START ---` and `--- PROMPT END ---`
2. Open any AI assistant (ChatGPT, Claude, Gemini, Cursor, etc.)
3. Paste the prompt
4. AI will execute all 20 steps automatically
5. You only need to verify the bot works on Telegram at the end

## What Makes This Prompt Complete

- **Real GDrive backup link** with file ID
- **Real credentials** (BOT_TOKEN, MEGA, GDrive, ngrok)
- **Real project path** for Lightning AI
- **Exact pip install commands** with version numbers
- **GPU auto-detect rules** for any GPU type
- **Troubleshooting table** for common errors
- **File structure** showing every important file
- **Verification steps** after each major install
- **Backup creation** instructions for future use
