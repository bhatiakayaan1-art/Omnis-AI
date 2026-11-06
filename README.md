# Omnis AI

Omnis AI is a local-first desktop web assistant built on a modular Python backend. It is not a model trained from scratch; it combines open-source models with deterministic tools and auditable controls.

## MVP capabilities

- Local conversational AI through Ollama
- Multilingual chat (including Hindi and Marathi)
- Safe arithmetic and Python simulation execution
- Real-time web search and weather reports
- Allowlisted Windows app launching
- Streaming chat replies and model warm-up for reduced wait times
- Installable, phone-friendly web app for Android and iPhone
- Local image and video generation through configurable ComfyUI workflows
- Voice input using the browser's speech API
- Code, science, math, art, and general assistance

## Run

On the configured PC, double-click `start-local.bat` or the **Omnis AI (Local)** desktop shortcut. The launcher uses the bundled Python runtime, starts Ollama and ComfyUI in the background when needed, and opens the Omnis page in your browser. You do not need to open their separate windows or install Python separately.

Ollama, the `qwen2.5:7b` chat model, ComfyUI, and its DreamShaper image files must still be installed on a new PC. Omnis checks for these components and reports clearly when a model is missing; the large AI models are not downloaded automatically. The configured PC already has them installed.

For manual development, run `python server.py` after configuring Ollama and any ComfyUI workflows. The server uses no third-party Python packages. Optional configuration:

The server uses no third-party Python package. Optional configuration:

```bat
set OMNIS_MODEL=llama3.2:3b
set OMNIS_OLLAMA_URL=http://127.0.0.1:11434
set OMNIS_OPEN_BROWSER=0
set PORT=8000
python server.py
```

For quicker chat, Omnis keeps the model loaded for 30 minutes, warms it in the background at startup, streams generated text immediately, and uses a 4K context with a bounded reply length. Actual speed still depends on the PC's RAM, CPU/GPU, and model. A smaller Ollama model such as `qwen2.5:3b` can be selected with `OMNIS_MODEL=qwen2.5:3b`.

## Run Omnis from the KLEVV USB drive

The portable Windows bundle includes Omnis, a standalone 64-bit Python runtime, Ollama, and the `qwen2.5:7b` model. On a 64-bit Windows 10 (22H2+) or Windows 11 PC, open the `Omnis-AI` folder on the USB drive and double-click `start-usb.bat`. Omnis opens in your browser on a free local port starting at http://127.0.0.1:8000; the launcher prints the exact address. This avoids clashing with another local app. Keep the launcher window open and the USB connected while using Omnis; closing the launcher stops its server. A previously installed Omnis app can show a cached screen even when the server is off, but chat will not work until `start-usb.bat` is running. On first use per PC, the launcher copies the model to that user's local app-data folder for faster startup, so allow about 5 GB of free internal storage.

The model uses several gigabytes of RAM, so a PC with at least 8 GB of memory is recommended. Chat and calculations work locally; live search and weather need internet. ComfyUI, image/video models, and any personal conversation history are not included. Omnis keeps each Windows user's memory database on that PC, not on the USB. This bundle is for 64-bit Windows and has not been tested on every PC or Windows configuration.

### Run from this PC without the USB

The `Omnis-AI` folder and `start-local.bat` are installed on the PC. Double-click `start-local.bat` to run Omnis using the locally installed Ollama model and `%USERPROFILE%\ComfyUI`; leave the launcher window open while you use it. It does not depend on the USB drive.

### Local ComfyUI image generation

When `start-usb.bat` runs, Omnis checks `%USERPROFILE%\ComfyUI` for a working ComfyUI CPU installation and the DreamShaper UNET, CLIP, and VAE model files. If present, it starts ComfyUI when needed and connects the USB app to its image workflow. ComfyUI and the image models are not stored on the USB; install and configure them separately on each PC. Image generation runs on the CPU here and may take several minutes. Omnis temporarily unloads its local chat model to free memory, then chat can load it again. Without the local ComfyUI setup and all three models, the launcher clearly reports that image generation is unavailable while the other Omnis features remain usable.

## Local creator mode

Use **Creator access** in the app to set up a new passcode on this PC and unlock a creator identity for the current browser. Choose a fresh passcode of at least 16 characters; enter it in the local dialog, never in the chat. Omnis stores a salted PBKDF2 verifier in `%LOCALAPPDATA%\OmnisAI\creator-auth.json`, not the plaintext passcode or the USB drive. The browser session expires after an hour or can be locked from the dialog. After five incorrect attempts, unlocking is paused for five minutes.

Creator mode lets Omnis recognize the signed-in user as its creator in replies. It does not grant administrator privileges, arbitrary command execution, security bypasses, or unrestricted access to PC files. The passcode previously shared in chat should be considered exposed; do not reuse it.

## Accuracy and live answers

Omnis uses deterministic local evaluation for recognized arithmetic instead of asking the language model to calculate. Questions that ask for current/latest/recent information, today's news or results, or citations automatically retrieve live web results. Omnis checks that result titles and descriptions match the question before using them; when search fails or results are unrelated, it says it could not verify the answer rather than letting the model guess. `/search your query` shows matching results with their source links. Search-result markup is removed and source text is treated as untrusted evidence. The model also gets instructions to cite relevant evidence, state uncertainty and assumptions, and avoid fabricated citations. Recent conversation history is bounded so long chats are less likely to crowd out the latest question, and a lower response temperature makes answers more consistent.

These are accuracy improvements, not a guarantee. Omnis currently uses the local `qwen2.5:7b` model; a small local model can still misunderstand a question or give incorrect answers, and search snippets can be incomplete. Check cited source pages for important decisions. Omnis does not train its model or independently verify every answer.

## Local conversation memory

When enabled, Omnis saves completed user/assistant turns in a local SQLite database at `%LOCALAPPDATA%\OmnisAI\memory.sqlite3` on Windows. It also picks up clearly stated preferences and personal details (such as “I prefer short answers” or “My favorite color is blue”) and supplies relevant saved context to later chats. Memory matching is local keyword matching; it is not model training or a change to the model weights.

Use the **Memory: On/Off** control to pause or resume saving and recall. Pausing keeps existing saved data but stops accessing and adding to it. **Forget all** erases saved conversations and extracted details, and starts a fresh chat. Memory stays on the PC; the text of the current conversation still goes to the configured Ollama model on that PC as usual. Anyone who can access the PC's local data may be able to read the memory database, so do not save sensitive information to a shared Windows account.

To choose another local database location, set `OMNIS_MEMORY_DB` before starting Omnis.

## Use Omnis from a phone

Omnis is an installable web app with a phone-sized chat layout, touch-friendly tools, and a home-screen icon. The AI model still runs on the PC; your phone is the remote screen and sends chat requests to that PC.

For private phone access, use a tailnet VPN rather than exposing the Omnis server or opening a router port:

1. Install Tailscale on the PC and phone and sign both into your private tailnet.
2. Start Omnis on the PC with `python server.py` (it stays bound to localhost).
3. In PowerShell on the PC, run `tailscale serve --bg 8000`.
4. Run `tailscale serve status` and open the HTTPS URL it shows on your phone.
5. Add it to the home screen: use the **Install app** button or browser menu on Android; in iPhone Safari use **Share → Add to Home Screen**.

The phone and PC must be online and connected to Tailscale for AI replies. The service worker caches only the app shell; it never caches chat/API responses. Do not bind Omnis to `0.0.0.0`, forward port 8000 from your router, or publish it to the public internet without adding a proper authentication and HTTPS deployment.

## Local tools

Use the Tools panel or chat commands:

- `/calc 12 * (3 + 4) + sqrt(16)`
- `10 plus 10`
- `What is 500 raised to the power of 2?`
- `two hundred and twenty-one times three`
- `/python print(sum(range(100)))`
- `/search current Python release`
- `/weather Pune`
- `/open calculator`
- `/open notepad`
- `/open chrome`
- `/open terminal`
- `/image A watercolor illustration of a mountain lake at sunrise`
- `/video A short cinematic shot of waves rolling onto a beach`

### Image and video generation setup

The [ComfyUI](https://github.com/Comfy-Org/ComfyUI) source, Python 3.12 environment, and CPU PyTorch are installed at `%USERPROFILE%\ComfyUI`. This computer's Radeon RX 550 is not a supported NVIDIA CUDA device; CPU mode is used and image generation will be slow. The Omnis image workflow uses Lykon's DreamShaper 8 FP16 files. Run `%USERPROFILE%\ComfyUI\download-model.bat` once to download the approximately 2.1 GB model files from a pinned Hugging Face revision with SHA-256 verification. Review the [model card and CreativeML OpenRAIL-M license](https://huggingface.co/Lykon/dreamshaper-8).

Start ComfyUI in CPU mode with `%USERPROFILE%\ComfyUI\start-cpu.bat`. Start Omnis with `start.bat`, which configures its localhost ComfyUI URL and checked-in API workflow. The Omnis UI reports image generation as available only after the required workflow and model files exist.

Video generation needs a separately installed compatible video model and API workflow and is not included in the CPU setup. Video diffusion is particularly demanding; this PC is not a practical local video-generation machine.

To use different workflows or a different ComfyUI install, configure these variables before starting Omnis:

```bat
set OMNIS_COMFY_URL=http://127.0.0.1:8188
set OMNIS_COMFY_DIR=C:\path\to\ComfyUI
set OMNIS_IMAGE_WORKFLOW=C:\path\to\image-api-workflow.json
set OMNIS_VIDEO_WORKFLOW=C:\path\to\video-api-workflow.json
python server.py
```

For image-only use, the video workflow variable can be omitted. Generation runs as a background job so you can keep chatting while it works. Video-generation latency is hardware- and workflow-dependent and can be substantial on CPU-only PCs.

## Safety

- App launching is restricted to an allowlist.
- HTTP binds to localhost only.
- Generated media is served only by proxying files requested from the configured ComfyUI instance; traversal paths are rejected.
- Python has a 12-second timeout and 20,000-character limit, and runs in a temporary working directory.
- Python is not an OS sandbox. Never execute untrusted code as an elevated process.
- External search and model providers may receive the text you send to them.
- For autonomous file, system, network, and destructive actions, the backend should require explicit confirmation before production use.
