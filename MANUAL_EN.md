# 🐾 VRAMaPyon — User Manual

**English** ｜ [📖 マニュアル（日本語）](MANUAL_JA.md)

---

## What is VRAMaPyon?

When you run local AI (LLMs, image generation), VRAM fills up fast — but on **GeForce + Windows**, `nvidia-smi` only returns `[N/A]` for per-process VRAM (a WDDM driver limitation). So you can see the *total*, but not *who's eating it*.

VRAMaPyon reads Windows' performance counter `\GPU Process Memory(*)\Dedicated Usage` to show **per-process VRAM** and **which GPU** each process is running on. Then it lets you release **just that** — an Ollama model, image-gen VRAM, or a single process.

---

## Launch

```bash
python VRAMaPyon.py
```

- On first launch it auto-installs `customtkinter` if it's missing.
- For console-only data (no GUI): `python VRAMaPyon.py --probe`.

---

## Reading the screen

![VRAMaPyon main window](完成_実機フル表示.png)

### The two GPU gauges (top)

Whole-memory usage for your NVIDIA card and the integrated GPU (the same idea as Task Manager's "GPU memory").

| Display | Meaning |
|---|---|
| 🟢 Plenty / 🟡 Caution / 🟠 Warning / 🔴 Critical | Usage color mark |
| Dedicated X/Y GB ・ Shared X/Y GB | Dedicated VRAM vs shared (system RAM) breakdown |

> The NVIDIA total comes from `nvidia-smi`; the integrated GPU (AMD/Intel) comes from the `\GPU Adapter Memory` counters. Even if the iGPU's dedicated VRAM is full, overflow goes to shared (RAM) — it never moves onto the NVIDIA card.

### Process list (most VRAM first)

Each row shows the process name, PID, VRAM usage, and action buttons.

| Display | Meaning |
|---|---|
| 🤖 | AI-related apps (Forge / ComfyUI / Ollama / llama.cpp / KoboldCpp …) |
| 🖥5070 / 🍃iGPU | The GPU the process is actually on (split by GB when it spans both) |
| 🔒 Protected | Critical Windows processes (`dwm` / `explorer` / `csrss` …) — no End button |
| Auto / fixed→XX | Current 🎛GPU setting, shown without pressing anything |

Use the **AI only** switch to narrow the list to AI-related processes.

### 🐾 Paw-print sparkline

After each name is a tiny graph of that process's VRAM over time (with a 🐾 paw at the tip). The segmented control at the list header switches two modes:

| Mode | Look |
|---|---|
| 📊 GPU share | Height = share of total GPU VRAM (sticks to the top when pinned, with a ceiling dotted line) |
| 📈 Fine detail | Auto-scaled to the process's own min–max (small ups & downs show clearly) |

Line, paw and number color reflect state: **🔥 pinned (red) / ▲ rising (orange) / stable (pink)**. Text has a white halo so it stays readable over the graph.

---

## 🦙 Gently release Ollama models

The 🦙 panel lists currently-loaded Ollama models.

- **Release** — sends `keep_alive:0` to drop just that model from VRAM. **The Ollama server stays alive**, so it comes right back on your next chat (no restart).
- **Release all** — drops all loaded models at once.
- Each model gets a **VRAM-fit badge** (✅ all on GPU / ⚠️ overflowing to RAM / 🔴 mostly RAM).

If Ollama isn't running, it shows "not connected" and the buttons are greyed out (they enable automatically once it's reachable).

---

## ❌ Ending a process

Any process — AI app or browser — can be ended from its row's End button (with a confirm dialog).

- **🔒 Protection**: critical Windows processes (`dwm` / `explorer` / `csrss` …) don't even show an End button.
- If it couldn't be ended, it may be a permissions issue (try running as administrator).

---

## 🎨 Release image-gen (SD) VRAM

| Target | Action | Setup |
|---|---|---|
| **Forge** | Unload the checkpoint | Launch with `--api` (default `http://127.0.0.1:7860`) |
| **ComfyUI** | Free models & memory | Auto-detected if running (default `http://127.0.0.1:8188`) |

Not running / not installed → "not connected" and the button stays grey. It enables automatically once started.

---

## 🎛 Pick a GPU per app (send non-AI to the iGPU)

Each process row's **🎛GPU** button lets you choose the GPU that app uses.

| Option | Meaning |
|---|---|
| 🚀 RTX5070 (high performance) | Render on the NVIDIA card |
| 🍃 iGPU (power-saving / VRAM-saving) | Send it to the integrated GPU |
| 🪟 Auto | Let Windows decide |

This is the same as Windows' *Graphics settings* (it writes `HKCU\…\DirectX\UserGpuPreferences`). The point: push browsers and GUI apps onto the iGPU so your NVIDIA card's VRAM stays free for LLM / SD.

**Good to know**
- It only moves **rendering (D3D) VRAM**. **CUDA memory (LLM/SD) always stays on the NVIDIA card** and can't be moved this way.
- Takes effect **after the target app restarts**; set **per exe path** (apps sharing one `python.exe` can't be split).
- The setting persists across reboots (it *is* the Windows graphics setting).

> 💡 **The real "all-in" is to move your monitor cable to the motherboard (iGPU) output.** The whole desktop, `dwm` included, moves off the NVIDIA card, turning it into a compute-only card.

---

## 🐾 Mini window

The **🐾 Mini** button shrinks into a small, frameless, always-on-top widget (live VRAM gauge).

- Drag to move ・ **double-click to restore** ・ right-click for an Open / Quit menu.
- Border color tracks usage: <80% = pink / 80–89% = amber / 90%+ = a **soft pulsing red** (a stylish warning meter).

Park it in a corner and keep half an eye on it.

---

## 📸 Logging (snapshot) + trend charts

The **📸 Log** button saves the current state to Excel (`.xlsx`).

- One row per process (timestamp, GPU info, process name, VRAM, which GPU, monitor …) — sorting / filtering / pivots work out of the box.
- Each save rebuilds a **"Trends" sheet** with three line charts placed at the **top**:
  - Total GPU VRAM over time (used / total)
  - GPU usage % over time
  - Per-process VRAM over time (top processes by peak)
- The data sheet's process-VRAM column gets **in-cell data bars (pink)** for at-a-glance size.
- The default filename is **date-stamped** (`VRAMaPyon_log_YYYYMMDD.xlsx`), so a day's records collect into one file. Pick an existing file to append.
- After saving, it asks **"Open this file now?"**

> 💡 A line needs **at least two snapshots** in one file. A single record is just one point and can't draw a line — record a few times, or use Auto-log below.

---

## ▶️ Auto-log

The easy way to capture a few minutes of trend. Use the **▶️ Auto-log** button and the **interval menu (15 / 30 / 60s)** next to it.

1. Press **▶️ Auto-log** and pick a save location once.
2. It appends to the same file at your chosen interval (the button turns red: "⏹ Stop! (count)").
3. Press **⏹ Stop!** to finish.
4. Open the **"Trends" tab** in the resulting Excel — the lines have grown 📈.

- Samples are buffered in memory; the actual file write & chart rebuild happen only every few samples + on stop, so **it stays light even over long sessions**.
- If a save fails (e.g. the log file is open in Excel), it stops automatically and tells you.

---

## 🔍 Diagnosis tool

The **"🔍 Diagnose"** button in the 🦙 panel launches the companion "Ollama diagnosis" GUI (one-shot "is this model fast? why is it slow?" diagnosis). It's a separate tool, called only when you need it.

---

## ⚙️ Settings (top of `VRAMaPyon.py`)

Tweak the constants at the top of the file.

| Constant | Description | Default |
|---|---|---|
| `OLLAMA_URL` | Ollama address | `http://localhost:11434` |
| `FORGE_URL` | Forge (needs `--api`) | `http://127.0.0.1:7860` |
| `COMFY_URL` | ComfyUI | `http://127.0.0.1:8188` |
| `REFRESH_MS` | Auto-refresh interval (ms) | `2500` |

The auto-log interval can be changed any time from the in-app menu (15 / 30 / 60s).

---

## 🌐 Japanese / English

The **🌐** button in the toolbar switches the whole UI between Japanese and English (it rebuilds everything on the spot). Log headers and chart labels switch too.

---

## FAQ

**Q. The process list is empty.**
→ Try running as administrator (some processes' counters need it).

**Q. The Ollama / image-gen buttons stay grey.**
→ They just aren't reachable yet. Start Ollama / Forge (`--api`) / ComfyUI and they enable automatically.

**Q. No line shows / the graph is empty.**
→ (1) Are you on the **"Trends" tab** at the bottom? (it opens there by default). (2) A line needs **two or more snapshots** (a single record is one point and can't form a line).

**Q. Auto-log stopped with an error.**
→ Saving fails if the log file is open in Excel. Close it and start again.

**Q. The 🎛GPU setting doesn't move my LLM's VRAM to the iGPU.**
→ That's expected. It only moves rendering VRAM; CUDA memory (LLM/SD) always stays on the NVIDIA card. To move the *display* itself, plug your monitor into the motherboard output.

---

*VRAMaPyon 🐾*
