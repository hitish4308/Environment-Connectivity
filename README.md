## Prompt Engineering Lab — Experiment 1
## Environment Setup & LLM Connectivity 
A minimal Python project that establishes a working environment for prompt
engineering: API key configuration, library installation, and successful
communication with an LLM via a simple streaming script.

---

## 📁 Project Structure

```
EXP_1/
├── Connectvity.py       # Main script — sends prompt, streams response
├── .env                 # Secrets (NVIDIA_API_KEY=...) — DO NOT COMMIT
├── .gitignore           # Excludes .env and caches
├── requirements.txt     # Python dependencies
└── README.md            # This file
```

---

## 🎯 Objectives

- Load an API key from an environment variable (`.env` file)
- Initialize a connection to an LLM (NVIDIA NIM, OpenAI-compatible endpoint)
- Send a simple prompt and stream the model's response
- Handle errors: missing key, network issues, API errors, retired models
- Verify end-to-end connectivity

---

## 🛠 Requirements

| Requirement    | Version |
|----------------|---------|
| Python         | 3.8+    |
| openai         | 1.0+    |
| python-dotenv  | 1.0+    |
| Internet       | Required |
| NVIDIA API key | Free at https://build.nvidia.com |

---

## 🚀 Quick Start (5 Commands)

```powershell
cd "C:\Users\admin\Downloads\Lab Manual_PE\EXP_1"

python -m pip install -r requirements.txt

$key = "nvapi-YOUR-KEY-HERE"
[System.IO.File]::WriteAllText("$PWD\.env", "NVIDIA_API_KEY=$key")

python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('found:', bool(os.getenv('NVIDIA_API_KEY')))"

python Connectvity.py
```

---

## 📦 Installation

### 1. Go to project folder

```powershell
cd "C:\Users\admin\Downloads\Lab Manual_PE\EXP_1"
```

### 2. (Recommended) Create a virtual environment

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks activation:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

### 3. Install dependencies

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

**`requirements.txt`:**
```
openai>=1.0.0
python-dotenv>=1.0.0
```

---

## 🔑 Configuration

### 1. Get an NVIDIA API key

1. Sign up at **https://build.nvidia.com** (free)
2. Click **Get API Key** → **Generate Key**
3. Copy the key (starts with `nvapi-`)

### 2. Create the `.env` file

**Do not use `Out-File -Encoding utf8`** on Windows PowerShell 5.1 — it adds
a BOM that breaks `python-dotenv`. Use .NET I/O instead:

```powershell
$key = "nvapi-YOUR-KEY-HERE"
[System.IO.File]::WriteAllText("$PWD\.env", "NVIDIA_API_KEY=$key")
```

### 3. Verify `.env` (value hidden)

```powershell
(Get-Content .env -Encoding UTF8)[0] -replace '=.*', '=<hidden>'
```

Expected:
```
NVIDIA_API_KEY=<hidden>
```

### 4. Confirm Python sees it

```powershell
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('found:', bool(os.getenv('NVIDIA_API_KEY')))"
```

Expected:
```
found: True
```

---

## 📜 The Script — `Connectvity.py`

```python
import os
from pathlib import Path
from dotenv import load_dotenv
from openai import OpenAI

# Load .env from the same folder as this script
env_path = Path(__file__).parent / ".env"
load_dotenv(dotenv_path=env_path)

# Read API key from environment
api_key = os.environ.get("NVIDIA_API_KEY")
if not api_key:
    raise SystemExit("❌ NVIDIA_API_KEY not found in environment variables.")

print("✅ Key loaded.")

# Create OpenAI-compatible client pointed at NVIDIA NIM
client = OpenAI(
    base_url="https://integrate.api.nvidia.com/v1",
    api_key=api_key,
)

print("⏳ Sending prompt...\n")

# Send prompt with streaming enabled
completion = client.chat.completions.create(
    model="nvidia/nemotron-3-super-120b-a12b",
    messages=[
        {"role": "user", "content": "Write a limerick about the wonders of GPU computing."}
    ],
    temperature=1,
    top_p=0.95,
    max_tokens=2048,
    stream=True,
)

# Print each streamed chunk as it arrives
for chunk in completion:
    if not chunk.choices:
        continue
    delta = chunk.choices[0].delta
    if delta and delta.content:
        print(delta.content, end="", flush=True)

print("\n\n✅ Done.")
```

---

## ▶️ Usage

```powershell
python Connectvity.py
```

**Expected output:**
```
✅ Key loaded.
⏳ Sending prompt...

There once was a GPU so keen,
... (limerick continues)

✅ Done.
```

---

## 🔧 Troubleshooting

### `❌ NVIDIA_API_KEY not found`

| Cause | Fix |
|-------|-----|
| `.env` missing | Create it with the `WriteAllText` command above |
| Wrong variable name | Must be exactly `NVIDIA_API_KEY` |
| BOM in `.env` | Rewrite using `[System.IO.File]::WriteAllText(...)` |
| Running from wrong folder | `cd` into `EXP_1` before running |
| File is `.env.txt` | `Rename-Item .env.txt .env` |

### `Error code: 410 - Gone`

The model ID was retired by NVIDIA. Swap to a currently-served model:

```python
model="nvidia/nemotron-3-super-120b-a12b",   # current default
model="gpt-oss-20b",                         # fast / small
model="meta/llama-3.3-70b-instruct",         # stable mid-size
```

Check the live list at **https://build.nvidia.com/models** before assuming
your code is broken — a 410 is the model's obituary, not your bug.

### `Error code: 401 - Unauthorized`

Key is invalid or revoked. Regenerate at https://build.nvidia.com.

### `openai.APIConnectionError`

Network, firewall, or VPN blocking `integrate.api.nvidia.com`.

### `Error code: 429 - Rate Limit`

Free tier caps at ~40 requests/minute. Wait 60s and retry.

### Script hangs on first request

Free-tier models sometimes cold-start. Wait 60–120s, or pass
`timeout=120.0` to the `OpenAI(...)` constructor.

### Only "reasoning" text prints, no final answer

Some models (Nemotron reasoning variants, DeepSeek-R1) stream
`reasoning_content` **before** `content`. If `max_tokens` is too low, the
reasoning alone consumes the budget. Either raise `max_tokens` to 2048+,
or remove the reasoning-printing lines from your script.

---

## 🔒 Security

- **Never commit `.env`** to version control. It is listed in `.gitignore`.
- **Never paste your API key** into chat, screenshots, README, or logs.
- **Rotate immediately** if a key is exposed:
  - Revoke at https://build.nvidia.com
  - Generate a new key
  - Update `.env`

### `.gitignore` contents

```
.env
venv/
__pycache__/
*.pyc
```

---

## 🧪 Verification Checklist

| Check | Command | Expected |
|-------|---------|----------|
| Python version | `python --version` | 3.8+ |
| Packages installed | `python -c "import openai, dotenv; print('ok')"` | `ok` |
| `.env` exists | `Test-Path .env` | `True` |
| Key loads | `python -c "..."` (see above) | `found: True` |
| Script runs | `python Connectvity.py` | Streams limerick |

---

## 📚 References

- [OpenAI Python SDK](https://github.com/openai/openai-python)
- [NVIDIA NIM API reference](https://docs.api.nvidia.com/nim/reference/llm-apis)
- [python-dotenv docs](https://saurabh-kumar.com/python-dotenv/)
- [Live model catalog](https://build.nvidia.com/models)

---

## 📝 Lab Manual Alignment

| Lab Requirement | Status |
|-----------------|--------|
| Load API key from `.env` / environment | ✅ |
| Initialize LLM connection | ✅ |
| Send prompt and print response | ✅ |
| Error handling (missing key, network, API) | ✅ |
| Verify connectivity end-to-end | ✅ |
| Handle deprecated model (410 Gone) | ✅ bonus |

---

## 📄 License

For educational / lab use only.
