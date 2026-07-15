
# 🎮 Virtual Steering Wheel — MediaPipe + Python

Control any car game using your hands as a steering wheel — no hardware needed. Just your webcam.

Uses Google's MediaPipe hand-tracking model to detect both your hands in real time, calculates the tilt angle between them, and translates it into keyboard input (arrow keys) that any keyboard-controlled game can read.

---

## ✨ Features

| Gesture | Action |
|---|---|
| Hold both hands up, level | Straight (no steering key pressed) |
| Tilt hands **left** | `←` LEFT arrow key pressed |
| Tilt hands **right** | `→` RIGHT arrow key pressed |
| Both hands open & visible | `↑` UP arrow key auto-pressed (accelerate) |
| Close **both hands into fists** | `↓` DOWN arrow key pressed (brake) — UP is released |
| Remove hands from frame | All keys released instantly (after a short grace period) |

On-screen HUD also shows a live steering wheel graphic, tilt angle in degrees, FPS counter, hand-detection status, and current action (ACCELERATING / BRAKE).

---

## How It Works

1. **MediaPipe Hands** detects 21 landmarks per hand from the webcam feed.
2. The **wrist landmarks** of both hands are used to calculate the tilt angle (`atan2`) between them — this drives steering.
3. **Finger tip vs. finger-joint (PIP) landmark positions** are compared to detect whether a hand is open or curled into a fist — this drives the brake.
4. Detected gestures are mapped to keyboard events using `pynput`, which simulates real arrow-key presses that any application (browser game, PC game, emulator) can receive.

```
Both hands level, open   →  Straight + Accelerate (↑)
Tilt LEFT                →  ← LEFT arrow key
Tilt RIGHT                →  → RIGHT arrow key
Both hands closed (fist)  →  ↓ DOWN arrow key (brake), ↑ released
Remove hands               →  All keys released instantly
```

---

## Requirements

- Python **3.9 – 3.12** (MediaPipe does not yet support 3.13+/3.14 reliably as of mid-2026 — use 3.12 if unsure)
- A webcam
- Windows, macOS, or Linux

---

## Installation

### 1. Get the project files
Make sure you have these three files in one folder:
```
steering_wheel.py
requirements.txt
README.md
```

### 2. Create a virtual environment (recommended)

**Windows:**
```bash
py -3.12 -m venv venv
venv\Scripts\activate
```

**macOS / Linux:**
```bash
python3.12 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

`requirements.txt` contains:
```
mediapipe==0.10.18
opencv-python>=4.8.0
pynput>=1.7.6
numpy<2
```

> **Note:** Newer MediaPipe releases (0.10.31+) currently have a known bug where `mp.solutions` is missing (`AttributeError: module 'mediapipe' has no attribute 'solutions'`). Version `0.10.18` with `numpy<2` is confirmed working.

### 4. Run
```bash
python steering_wheel.py
```
Press **Q** in the camera window to quit at any time.

---

## macOS Setup ⚠️

1. Go to **System Settings → Privacy & Security → Camera**
2. Enable access for **Terminal** (or your Python launcher / VS Code)
3. Run the script again

---

## Windows Setup

The script auto-detects your OS via:
```python
backend = cv2.CAP_AVFOUNDATION if platform.system() == "Darwin" else cv2.CAP_ANY
```
No manual change needed — on Windows it automatically uses `cv2.CAP_ANY`.

**If you see a "running scripts is disabled" error** when activating the venv in PowerShell:
```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Confirm with `Y`, then try activating the venv again.

**If the camera window shows a permission prompt**, click **Allow**.

**If `py`/`python` isn't recognized**, make sure "Add python.exe to PATH" was checked during install, and check **Settings → Apps → Advanced app settings → App execution aliases** to ensure Python entries are enabled.

---

## Usage Tips

- Hold both hands up in front of the camera like you're gripping a real steering wheel.
- Keep hands **open** (fingers extended) to accelerate.
- **Close both hands into fists** to brake — you'll see "BRAKE (FIST)" appear on screen.
- Tilt your hands (not just wrists) to steer — the wheel graphic in the corner mirrors your tilt.
- Works with any game or app that reads arrow-key input.

---

## Config (top of `steering_wheel.py`)

| Setting | Default | Description |
|---|---|---|
| `CAMERA_INDEX` | `0` | `0` = built-in webcam, `1`/`2` = external USB camera |
| `DEAD_ZONE_DEG` | `12` | Degrees of tilt to ignore at center (prevents jitter) |
| `RELEASE_ZONE_DEG` | `6` | Hysteresis zone — angle must cross back past this before releasing a steering key |
| `SOFT_ZONE_DEG` | `25` | Angle at which steering strength reaches 100% |
| `FLIP_CAMERA` | `True` | Mirror the feed (selfie view). Set `False` for some external cameras |
| `SHOW_ANGLE` | `True` | Show live tilt angle in degrees on screen |
| `MIN_DETECTION_CONF` | `0.7` | MediaPipe hand-detection confidence threshold |
| `MIN_TRACKING_CONF` | `0.5` | MediaPipe hand-tracking confidence threshold |
| `GRACE_FRAMES` | `8` | Frames to wait before releasing all keys when hands disappear |

### Fist-brake sensitivity
In the `is_fist()` function, `curled >= 3` means at least 3 of 4 fingers must be curled to count as a fist. Increase to `4` for a stricter (must fully close) brake, or decrease to `2` for an easier trigger.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `[ERROR] Cannot open camera` | Try changing `CAMERA_INDEX` to `0`, `1`, or `2` |
| `AttributeError: module 'mediapipe' has no attribute 'solutions'` | Downgrade with `pip install mediapipe==0.10.18` and `pip install "numpy<2"` |
| Steering is reversed | Toggle `FLIP_CAMERA = False` in config |
| Keys stuck after removing hands | Hands must be fully out of frame for ~8 frames (`GRACE_FRAMES`) |
| Brake doesn't trigger / triggers too easily | Adjust `curled >= 3` threshold in `is_fist()` |
| Low FPS / laggy | Lower camera resolution in the script, close other apps, or ensure `model_complexity=0` is set |
| `ModuleNotFoundError: No module named 'cv2'` | You're not in the activated venv — run `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (Mac/Linux) first |
| PowerShell "scripts disabled" error | Run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` |

---

## Works With Any Game That Uses Arrow Keys

- Google Chrome Dinosaur game
- Trackmania
- TORCS
- Hill Climb Racing (browser)
- Any browser/PC racing game using arrow keys for steer/accelerate/brake

---

## Project Structure
```
.
├── steering_wheel.py     # Main script — run this
├── requirements.txt      # Python dependencies
└── README.md             # This file
```

---

## Tech Stack

- **[MediaPipe](https://ai.google.dev/edge/mediapipe)** — real-time hand landmark detection
- **[OpenCV](https://opencv.org/)** — camera capture, frame processing, drawing HUD overlays
- **[pynput](https://pynput.readthedocs.io/)** — simulated keyboard input
- **[NumPy](https://numpy.org/)** — angle smoothing / math

---

## License / Disclaimer

For personal/educational use. Simulated key presses may not work with games that use raw input APIs or anti-cheat systems that block synthetic input.
