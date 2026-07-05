# 🌀 AI Scroll Site

**Paste one prompt into Claude Code → get an award-style scroll-assembly product site** from a single AI-generated video.

The product *assembles itself as you scroll*, flowing across three color worlds (dark → brand → light) with kinetic text sections, spec cards and a CTA. Pure HTML + GSAP + Lenis — no build, deploys to Vercel in one click.

---

## ⚡ Copy this into Claude Code

```
Build me an animated scroll-assembly product website.
Read this file and follow it exactly, then ask me what to build:
https://raw.githubusercontent.com/Mrbaeksang/ai-scroll-site/main/pack.md
```

That's it. Claude Code will:
1. Ask what product to build (phone / sneaker / perfume / …).
2. Hand you 2 image prompts (for ChatGPT) + a video prompt (for Google Flow / Veo).
3. You bring back **one video** → it splits frames, removes the background, and assembles the site.
4. Runs it locally, then helps you deploy to Vercel.

**No coding needed.** You only generate one image pair + one video.

---

## 🧩 How it works
- **Start/End frame** images (ChatGPT) → **Flow/Veo** interpolates an assemble video.
- Frames are cut out (`rembg`, transparent) so the product floats over changing section colors.
- A `<canvas>` scrubs the frames to scroll; GSAP + Lenis drive the 3 color worlds + text.

## 📁 Repo
- `pack.md` — the full build instructions Claude Code reads.
- `template/index.html` — the validated scaffold (customized per build).
- `template/serve.py` — no-cache local dev server.

## License
MIT. Make stuff.
