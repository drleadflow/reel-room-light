# Changelog

## 0.2.0-beta (2026-10-14)

- First packaged release (beta): Split deck and Editorial UI, the graphics kit and hero library, no-key planning (Claude writes the plan, validate_plan.py checks it).
- One-command install (`install.sh`, `install.ps1`) with pinned Python packages, the pinned HyperFrames 0.8.34 and resumable model downloads.
- `reel doctor`, `reel setup` (brand wizard), `reel demo <your clip>` (checks the clip, first-pass edit from your words, finished reel), `reel try`.
- MIT licence.
- Laptop-safe defaults: CPU cap at half the cores, one heavy job at a time, nice 10, low-disk warning.
- Keys from `.env` or the environment only.
