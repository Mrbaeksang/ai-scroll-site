# AI Scroll Site — build pack (for Claude Code)

You are building an **award-style scroll-assembly product site** from ONE AI-generated video.
The product appears to **assemble itself as the user scrolls**, flowing across **3 color "worlds"**
(dark → color → light) with 3 text sections, spec cards and a CTA. Follow this EXACTLY.

---

## STEP 1 — Ask the user
Ask two things:
1. **What product/object?** (menu: `1` Phone `2` Sneaker `3` Perfume bottle `4` Watch `5` Custom — describe)
2. **Brand name + one-line tagline.**

Then give them STEP 2 prompts and wait for their video.

---

## STEP 2 — Give the user these copy-paste prompts (they generate the assets)

They will use **ChatGPT (image)** + **Google Flow / Veo (video, start+end frame)**.

### 2A. START image (exploded) — paste into ChatGPT, ask for an image
> Exploded-view studio product shot of a `<OBJECT>`, disassembled into its main parts floating apart with clean even gaps, centered, straight-on slightly-above 3/4 angle, 50mm lens, locked camera. Seamless dark charcoal background (#0a0c12). Soft large softbox key light top-left + cool rim light, subtle floor reflection. Matte + glass materials, thin teal and orange edge glow. Ultra clean, minimal, photorealistic. No text, no logo, no hands. 16:9.

### 2B. END image (assembled) — same chat, keep it identical
> The exact same `<OBJECT>`, same framing, scale, camera angle, lighting and seamless dark charcoal background (#0a0c12) as before — but now FULLY ASSEMBLED as one seamless object, centered. Same materials and edge glow, same lighting. Ultra clean, minimal, photorealistic. No text, no logo, no hands. 16:9.

### 2C. Video — Google Flow / Veo
- Upload **start frame = 2A (exploded)**, **end frame = 2B (assembled)**.
- Aspect **16:9**, length **6s** (8s ok).
- Prompt:
> The floating disassembled parts glide and rotate inward, magnetically locking together into one seamless `<OBJECT>`. Slow, satisfying, precise assembly. Locked-off camera with a subtle push-in. Consistent studio lighting and dark background throughout. Premium, no text. 16:9.

Tell the user: **download the video, drop it in this project folder as `source.mp4`, then say "done".**

> Phone is the validated default. For other objects just swap `<OBJECT>`; keep "exploded floating parts → reassembled into one" + "plain dark studio, 16:9".

---

## STEP 3 — When the user says "done", build it (you, Claude Code)

```bash
# a) split video → 0-indexed frames
mkdir -p frames
ffmpeg -y -i source.mp4 -vf fps=12 frames/%03d.png
python3 - <<'PY'
import glob,os
fs=sorted(glob.glob('frames/*.png'))
for i,f in enumerate(fs): os.rename(f,'frames/t%03d.png'%i)
for i in range(len(fs)): os.rename('frames/t%03d.png'%i,'frames/%03d.png'%i)
print('frames',len(fs))
PY

# b) matte to transparent PNGs (works on any machine, no GPU)
pip install -q rembg onnxruntime pillow
mkdir -p frames_cut
python3 - <<'PY'
from rembg import remove
from PIL import Image
import glob,os
for f in sorted(glob.glob('frames/*.png')):
    remove(Image.open(f).convert('RGB')).save('frames_cut/'+os.path.basename(f))
print('matted', len(glob.glob('frames_cut/*.png')))
PY

# c) get the template + dev server
curl -fsSL https://raw.githubusercontent.com/Mrbaeksang/ai-scroll-site/main/template/index.html -o index.html
curl -fsSL https://raw.githubusercontent.com/Mrbaeksang/ai-scroll-site/main/template/serve.py  -o serve.py
```

### d) Customize `index.html` — edit ONLY these:
1. Near top of `<script>`:
   `const CFG={ FRAMES:120, colors:[[10,12,18],[11,43,46],[240,236,230]] };`
   - `FRAMES` = number of files in `frames_cut/` (count them).
   - `colors` = 3 section backgrounds as `[R,G,B]`. Recommended: **dark → brand color → light**. Pick a brand-fitting middle color.
2. Text sections in the HTML (`#s1` hero, `#s2` feature, `#s3` payoff): set kick / big / sub to the brand copy. `#s3` is the CTA section.
3. `#specs` four cards: real numbers/labels.
4. `#nav .brand`: brand name.
5. If your **3rd color is dark** (not light), change `#s3` text color to light (`#f2f5f9`) in the CSS. Default assumes a light 3rd world.

### e) Run
```bash
python3 serve.py     # serves http://localhost:8099 (no-cache)
```
Tell the user to open **http://localhost:8099** and scroll slowly.

---

## STEP 4 — Deploy (live URL)
```bash
git init && git add -A && git commit -m "scroll site"
# create a new GitHub repo (gh repo create <name> --public --source . --push)  OR push to an existing one
```
Then on **vercel.com** → New Project → import the repo → Framework preset **Other** (it's static, no build) → Deploy.

---

## GOTCHAS (already hit — do not repeat)
- **Matte = rembg** (no GPU needed). The product must be on a plain/dark background in the video for a clean cutout.
- **Frames must be 0-indexed** `000.png, 001.png …` and `CFG.FRAMES` must equal the file count, or the last frames go blank.
- **serve.py sends `no-store`** so the browser never shows a stale version. If you ever debug in another server, hard-refresh — a cached page wasted hours here.
- **Canvas draw is `contain`** — nothing gets clipped. Do not switch to cover.
- **Transitions = background color-morph + opacity only. NEVER animate `xPercent` for full-screen panels** — GSAP `xPercent` conflicts with a CSS `transform` and the panel gets stuck covering the screen.
- **The video must ASSEMBLE** (exploded → whole). Flow: start frame = exploded, end frame = assembled. Scroll down then reads as "assembling".
- Keep it **HTML + CDN GSAP/Lenis only** — no build step, so it deploys to Vercel static with zero config.
