HandConnect AR — Feature Enhancement PR
Overview
Major feature expansion of the original HandConnect hand-tracking AR experience. This PR adds a full settings system, 8 redesigned themes, five toggleable skeleton styles, 11 camera filters (including a reworked Matrix filter with live parameter tuning), 5 new creative modes, draw mode, comprehensive audio controls, and a new README that documents how to use the expanded feature set — all with no new dependencies.

Changes
⚙️ Settings Panel
Added a slide-out settings panel triggered by a gear button (top-right)
Panel organizes all toggles into labeled sections: Display, Effects, Creative, Draw Mode, Audio, Camera Filter
Gear button rotates 90° when panel is open
All settings take effect instantly with no reload required
🎨 Theme System — Full Redesign
Replaced the original 5 themes (Rainbow, Cyberpunk, Lava, Ocean, Galaxy) with 8 new ones. Each theme uses a time-based HSL function for smooth animated color cycling and updates the entire UI accent color dynamically via a CSS variable.

Theme	Character
Neon	Cycling purple → cyan, high energy
Vapor	Pastel lavender and rose, vaporwave
Ocean	Blue-cyan shifting waves
Synthwave	Hot pink ↔ electric blue alternating
Toxic	Acid green and chartreuse
Inferno	Deep orange to near-white-hot
Ice	Cool blues and whites, breathing pulse
Cyberpunk	Hard red/cyan high contrast
Theme bar is now toggleable via settings (Show/Hide)
Each theme button shows a colored gradient dot indicator
Theme bar is horizontally scrollable on narrow screens
Switching themes updates --accent CSS variable, re-coloring toggles, labels, and panel accents in real time
🎛️ All UI Elements Now Toggleable
Every visual element can be independently shown or hidden from the settings panel:

Display

Stats HUD (hands detected, FPS, gesture, spread %)
Theme Bar
Effects

Matrix Rain background
Hand Skeleton overlay
Fingertip Sparks particles
Lightning Arcs (two-hand proximity)
Mandala pattern (two hands)
🦴 Skeleton Styles — 5 Toggleable Looks
Added a new Skeleton Styles settings group underneath Effects
Styles can be enabled individually or stacked while testing combinations
Hologram is enabled by default for a more cinematic baseline hand render
New looks: Hologram, Constellation, Circuit, Mystic Sigil, X-Ray Plasma
📷 Camera Filters — 11 Total
Added a camera filter system with two implementation modes:

CSS-based (instant, no GPU overhead): Normal, Grayscale, Sepia, Invert, Hyper Color, Noir
Canvas pixel-processed (per-frame GPU readback): Thermal, Pixelate, Glitch, Comic, Matrix
New filters:

Thermal — Luminance remapped to a blue→green→yellow→red heat palette
Pixelate — 14×14px block averaging
Glitch — RGB channel split with sinusoidal shift and random horizontal tears
Comic — High-contrast saturation boost with halftone Ben-Day dot overlay
Matrix — See section below
⬛ Matrix Filter — Luminance-Driven Character Mosaic
Complete rework of what was originally a simple CSS green-tint overlay.

How it works:

Each frame, the raw video is sampled at 50% resolution into an off-DOM canvas
The screen is divided into a character grid (default 14×14px cells)
Each cell's luminance is inverted — dark areas of the video (the person) become high-luminance, bright areas (background) become low-luminance
Characters are rendered with brightness proportional to inverted luminance
A hard cutoff eliminates the background entirely
Probabilistic sparseness creates a soft edge transition at the subject silhouette
A three-tier glow system (3px / 7px / 16px shadowBlur) adds bloom to bright areas
A persistent off-DOM canvas layers falling katakana rain leaders on top via screen compositing
Characters are stable per cell (only ~3% chance of refresh per frame) so the figure is readable
Matrix Tuner panel (appears automatically when Matrix filter is selected):
A live parameter panel (bottom-left) with 6 sliders:

Slider	Range	Effect
Dark Cut	0.20 – 0.70	Background elimination threshold
Edge Fade	0.45 – 0.82	Width of silhouette transition zone
Contrast	0.25 – 1.50	Gamma curve — lower = more punch
Char Size	8 – 26px	Character grid density
Rain Trails	0.02 – 0.22	Rain persistence (lower = longer trails)
Rain Density	2% – 40%	Falling leader character frequency
Reset Defaults button snaps all parameters back to baseline. Panel auto-hides when switching to any other filter.

🎨 Draw Mode
Index finger on either hand paints persistent glowing trails onto a dedicated canvas layer (z-index above background, below hand skeleton)
Trails use the current theme color
Draw canvas persists across frames independently of the main render loop
"Clear Canvas" button in settings wipes all trails
"✦ DRAW MODE" badge appears at bottom of screen when active
🔊 Audio Controls
Three-level independent audio control:

Master Audio — Global mute/unmute (ramps gain smoothly)
Ambient Hum — Sine oscillator pitch-modulated by distance between two hands
Pinch Zap — Sawtooth burst triggered on pinch gesture detection
🎵 Creative Features (5 new, all off by default)
Finger Piano

Each fingertip maps to a pentatonic note: C4 (thumb), E4 (index), G4 (middle), A4 (ring), C5 (pinky)
Triggers on downward y-velocity > 0.02 threshold (tap gesture)
Plays triangle wave oscillator via Web Audio API
Floating note label rises from the triggering fingertip
Large note name flashes center-screen with theme-colored glow
Ghost Trails

Stores last 12 hand landmark snapshots in a ring buffer
Draws fading skeleton echoes behind current hands using drawConnectors
Opacity scales linearly from 0 → 38% across the history buffer
Fireworks

Detects open hand (spread > 22%) with upward palm velocity (Δy < −0.025 per frame)
Launches a physics firework projectile from the palm center
Projectile arcs upward with gravity, explodes into 80–120 sparks at apex
Sparks have individual hue variation, gravity, and velocity drag
1.0 second cooldown prevents spam
Renders in screen blend mode for natural glow compositing
Aura Field

Soft radial gradient centered on each palm landmark (index 0)
Radius pulses with hand velocity: 90 + handVel×600 + sin(time×3)×22
Uses theme colors at 30% opacity in screen blend mode
Kaleidoscope

After each frame renders, copies mainCanvas to an off-DOM buffer
Draws the buffer back with three mirror transforms (H-flip, V-flip, both-flip) at 60% opacity in screen blend
Creates 4-way symmetric patterns from hand effects
Auto-skips frames when FPS drops below 26 to protect performance
🏗️ Architecture
Canvas layer stack (z-index order):

Canvas	Z	Purpose
<video>	0	Raw camera feed (CSS scaleX(-1))
#filterCanvas	1	Camera filter output
#bgCanvas	2	Matrix rain background
#drawCanvas	4	Persistent finger paint layer
#mainCanvas	5	Hand skeleton, effects, physics
Off-DOM canvases:

kaleidoCvs — Kaleidoscope frame buffer
mfCvs — Matrix filter persistent rain character layer
matSampleCvs — Half-resolution video pixel sampler (willReadFrequently)
Fonts: Space Mono (monospace HUD/labels) + Syne (display/UI)

Files Changed
index.html — Core AR experience enhancements in the single-file app
README.md — Usage guide for the new settings, filters, creative modes, draw mode, audio controls, and skeleton styles
Dependencies
No new dependencies. Uses the same CDN scripts as the original:

@mediapipe/camera_utils
@mediapipe/drawing_utils
@mediapipe/hands
Google Fonts (Space Mono, Syne)
