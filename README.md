# Happy-Horse-1.0
Happy Horse 1.0 is an open-source AI video generation model with synchronized audio, multilingual lip-sync, and 1080p output. Learn how Happy Horse delivers cinematic quality in seconds.

# Happy Horse 1.0 — Open-Source AI Video Generation Model

> **The first open-source SOTA AI video model with native joint audio-video generation.**
> 15B-parameter unified self-attention Transformer · 1080p in ~38 seconds · 6-language lip-sync · Single-pass dialogue, Foley & ambient audio.

<p align="center">
  <a href="#-what-is-happy-horse-10">What is Happy Horse 1.0</a> •
  <a href="#-key-features">Features</a> •
  <a href="#-model-architecture">Architecture</a> •
  <a href="#-benchmarks">Benchmarks</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-comparison">Comparison</a> •
  <a href="#-faq">FAQ</a>
</p>

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/status-coming%20soon-orange">
  <img alt="License" src="https://img.shields.io/badge/license-open%20source-blue">
  <img alt="Parameters" src="https://img.shields.io/badge/params-15B-blueviolet">
  <img alt="Output" src="https://img.shields.io/badge/output-1080p%20%2B%20audio-green">
  <img alt="Speed" src="https://img.shields.io/badge/speed-~38s%20%2F%201080p-brightgreen">
  <img alt="Steps" src="https://img.shields.io/badge/sampling-8%20steps%20no%20CFG-success">
</p>

<p align="center">
  <strong>🌐 Official site: <a href="https://happyhorses.io/">happyhorses.io</a></strong>
</p>

---

## 🐎 What is Happy Horse 1.0?

**Happy Horse 1.0** is an open-source state-of-the-art AI video generation model that produces 1080p cinematic video **and** synchronized audio in a single forward pass. It is one of the first open-weights models to natively generate dialogue, ambient sound, and Foley effects jointly with video — eliminating the multi-stage pipeline of generating silent video, then dubbing, then lip-syncing that every other open video model relies on.

The model is built around a **15-billion-parameter unified self-attention Transformer** that processes text, image, video, and audio tokens in a single token sequence. There is no cross-attention, no separate audio branch, and no dedicated conditioning network. Combined with **DMD-2 distillation**, the model generates high-quality video in just **8 denoising steps without classifier-free guidance**, producing a 1080p clip in roughly 38 seconds on an H100.

Happy Horse 1.0 first surfaced publicly as a "mystery model" on the Artificial Analysis Video Arena leaderboard, where it competed anonymously against frontier closed models from ByteDance, Kling, and Google. Independent observers in the AI video community noted that it appeared to come from Asia, that it included native audio generation, and that its motion quality placed it in the same tier as the most recent Kling releases.

This repository will host the **base model**, the **distilled 8-step model**, the **super-resolution module**, and **inference code** under an open license that permits commercial use and fine-tuning.

> ⚠️ **Repository status:** The model weights and inference code are marked **"coming soon."** This README documents the announced architecture, training, and benchmark results. **Star and watch this repo** to be notified the moment the weights are published. In the meantime, you can try the model and read the latest updates on [happyhorses.io](https://happyhorses.io/).

---

## ✨ Key Features

### 🎬 1080p Cinematic Output
Happy Horse 1.0 generates HD video at up to 1080p resolution. Default clip length is 5–10 seconds, with multiple aspect ratios supported (16:9, 9:16, 4:3, 21:9, 1:1). Output is tuned for cinematic motion: stable camera moves, coherent physics, and minimal "morphing" or "glitching" — the failure modes that plague most diffusion video models.

### 🔊 Native Joint Audio-Video Generation
This is the headline feature. A single unified Transformer denoises **video tokens and audio tokens together** in the same sequence. The result: dialogue is naturally aligned to mouth shapes at the phoneme level, footsteps land on the right frames, and ambient noise responds to camera cuts — all without a single line of post-production code.

### 🗣️ 6-Language Native Lip-Sync
Native lip-sync support for **English, Mandarin Chinese, Japanese, Korean, German, and French** with industry-leading low Word Error Rate (WER). Speech timing, prosody, and mouth shapes are learned jointly with the video, not bolted on afterward.

### ⚡ ~38s for 1080p — Only 8 Denoising Steps
DMD-2 (Distribution Matching Distillation v2) reduces the sampling loop from the typical 25–50 steps down to just **8 steps with no classifier-free guidance required**. Combined with the MagiCompiler full-graph compilation runtime — which fuses operators across Transformer layers for an additional ~1.2× speedup — the model produces a 1080p clip in roughly 38 seconds on a single H100, and a 256p preview in around 2 seconds.

### 🧩 Unified Text-to-Video and Image-to-Video
A single set of weights handles both **text-to-video** and **image-to-video** workflows. Style, subject identity, and physical realism stay consistent regardless of whether the input is a text prompt, a reference image, or both.

### 🔓 Fully Open Source
The full release includes the **base 15B model**, the **distilled 8-step model**, the **super-resolution module**, and **inference code**. Self-host on your own GPUs, fine-tune on your own data, and use the outputs commercially.

---

## 🏗️ Model Architecture

Happy Horse 1.0 makes a deliberately minimalist architectural choice: instead of stacking multiple specialized networks for video, audio, and conditioning, it puts everything into one unified Transformer. The simplicity is the point — fewer moving parts, fewer alignment failures.

### Core specifications

| Component | Specification |
|---|---|
| Total parameters | ~15B |
| Architecture type | Unified self-attention Transformer (no cross-attention) |
| Total layers | 40 |
| Layer layout | Sandwich: first 4 + last 4 layers use modality-specific projections; middle 32 layers share parameters across all modalities |
| Modalities | Text, image, video, and audio tokens — processed in a single token sequence |
| Multimodal fusion | Per-attention-head learned scalar gates with sigmoid activation (for training stability) |
| Conditioning | Reference image + denoising signals routed through one minimal unified interface — no dedicated conditioning branches |
| Timestep handling | **No explicit timestep embeddings** — the model infers the denoising state directly from the noise level of input latents |
| Distillation method | DMD-2 (Distribution Matching Distillation v2) |
| Sampling steps | 8 |
| Classifier-free guidance | Not required |
| Inference runtime | MagiCompiler (full-graph compilation, ~1.2× end-to-end speedup) |
| Default GPU | NVIDIA H100 80GB |

### Why a unified self-attention Transformer?

Most open-source video models — Wan 2.2, HunyuanVideo, LTX-2, CogVideoX — use a Diffusion Transformer (DiT) backbone where text conditioning is injected via **cross-attention** from a separate text encoder, and audio (when present) is generated by an entirely separate model afterward. This works, but it means the audio model has no idea what the video model "saw" at each frame, and lip-sync becomes a downstream alignment problem.

Happy Horse 1.0 takes the opposite approach. **Text tokens, a reference-image latent, and noisy video and audio tokens are concatenated into a single sequence and jointly denoised by self-attention.** Every attention layer can look at every modality at every layer. The model is forced to learn alignment as part of denoising, not as a separate stage.

### Why a sandwich layout?

The first 4 and last 4 layers use **modality-specific projections** so that each input type can be embedded into and decoded from the shared latent space cleanly. The middle 32 layers share parameters across all modalities — this is where the actual cross-modal reasoning happens. The result is parameter efficiency: 32 of the 40 layers are doing double duty as the video transformer, the audio transformer, *and* the text-to-video aligner all at once.

### Why per-head gating?

Joint multimodal training is notoriously unstable — gradients from the audio loss can dominate or be dominated by gradients from the video loss. Happy Horse 1.0 inserts a **learned scalar gate with sigmoid activation on each attention head**. During training, the model can effectively turn off heads that are producing destructive gradients in a particular modality and amplify heads that are helping. This is a small architectural detail that pays off as a much smoother loss curve.

### Why no explicit timestep embeddings?

Diffusion models traditionally feed the current denoising timestep into every layer as an explicit embedding. Happy Horse 1.0 omits this entirely. Because the noise level is already encoded in the noisy latents themselves, the model learns to read it directly from its inputs. This **timestep-free denoising** simplifies the architecture and is one of the prerequisites for the aggressive 8-step DMD-2 distillation.

### Why DMD-2 distillation?

Standard video diffusion needs 25–50 denoising steps with classifier-free guidance, both of which double or triple inference cost. **DMD-2 (Distribution Matching Distillation v2)** trains a student model to match the teacher's full output distribution in just **8 steps without CFG**. The quality drop is minimal but the wall-clock speedup is roughly an order of magnitude. This is what makes "1080p in 38 seconds" possible at all.

---

## 📊 Benchmarks

Happy Horse 1.0 is evaluated on the **Artificial Analysis Video Arena**, the industry-standard public leaderboard for AI video models. The arena uses **blind head-to-head comparisons** — users see two videos generated from the same prompt without knowing which model produced which, then vote on which they prefer. Rankings are computed using an Elo rating system.

Two leaderboards track text-to-video and image-to-video performance separately:

- 📈 [Artificial Analysis Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video)
- 🖼️ [Artificial Analysis Image-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/image-to-video)
- 🥊 [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena) — vote for yourself

### How to interpret arena Elo scores

For context on what the numbers on the leaderboard mean, here is the current top tier of the Artificial Analysis Text-to-Video Leaderboard:

| Tier | Models |
|---|---|
| **Frontier closed models** (Elo 1,200–1,275) | Dreamina Seedance 2.0, SkyReels V4, Kling 3.0, PixVerse V6, Veo 3.1, Runway Gen-4.5 |
| **Mid-tier closed models** (Elo 1,150–1,200) | Sora 2 Pro, Hailuo 2.3, Wan 2.6, Vidu Q2 |
| **Top open-weights models** (Elo 1,100–1,135) | LTX-2 Pro, LTX-2 Fast, Wan 2.2 A14B |
| **Earlier open-weights** (Elo 950–1,020) | HunyuanVideo-1.5, Wan 2.1 14B, Wan 2.2 5B, Hunyuan Video |

Anything that places at or above the LTX-2 line is currently the open-source state of the art. Anything that places in the frontier closed tier is competing directly with the best paid APIs in the world.

### Try Happy Horse 1.0 on the leaderboard

The model first appeared on the Artificial Analysis Video Arena under its codename and has been generating community discussion about its motion quality, audio sync, and prompt adherence. To see its current placement and to vote on its outputs against other top models, visit the official leaderboards linked above. You can also try the model directly on [happyhorses.io](https://happyhorses.io/) and judge it against your own prompts.

---

## 🚀 Quick Start

> The repository is currently in pre-release. Code and weights will be published here at launch. Star and watch the repo, or follow updates on [happyhorses.io](https://happyhorses.io/).

### Planned installation

```bash
git clone https://github.com/<org>/happy-horse.git
cd happy-horse
pip install -r requirements.txt
```

### Planned text-to-video usage

```python
from happy_horse import HappyHorsePipeline

pipe = HappyHorsePipeline.from_pretrained("happy-horse/hh-1.0-distilled")
pipe.to("cuda")

video = pipe(
    prompt=(
        "A young desert survivor in a hooded cloak runs across golden sand dunes "
        "under intense sunlight, cinematic tracking shot, golden hour lighting"
    ),
    negative_prompt="blurry, low quality, distorted, morphing",
    resolution=(1920, 1080),
    duration_seconds=5,
    audio_language="en",
    num_inference_steps=8,   # DMD-2 distilled — no CFG needed
)

video.save("output.mp4")
```

### Planned image-to-video usage

```python
from PIL import Image
from happy_horse import HappyHorsePipeline

pipe = HappyHorsePipeline.from_pretrained("happy-horse/hh-1.0-distilled")
pipe.to("cuda")

init_image = Image.open("portrait.png")

video = pipe(
    image=init_image,
    prompt="The subject begins speaking warmly, soft natural lighting, gentle camera push-in",
    audio_language="en",
    duration_seconds=8,
)

video.save("output.mp4")
```

### Hardware requirements

| Tier | GPU | VRAM | Notes |
|---|---|---|---|
| **Recommended** | NVIDIA H100 80GB | 80 GB | ~38s per 1080p clip — published reference number |
| **Workable** | NVIDIA A100 80GB | 80 GB | Full quality, slower wall-clock |
| **Consumer (TBC)** | RTX 4090 / 6000 Ada | 24–48 GB | Distilled model + lower resolution recommended; benchmarks at release |

Memory-saving options including model offloading and quantization will be documented in this repository at launch.

### Try it now without installing

While weights are pending, you can try Happy Horse 1.0 in the browser and read the latest model updates at **[happyhorses.io](https://happyhorses.io/)** — no installation, no GPU, free credits to test text-to-video, image-to-video, and audio-video generation.

---

## 🆚 Comparison with Other Open-Source Video Models

| Feature | **Happy Horse 1.0** | LTX-2 Pro | Wan 2.2 A14B | HunyuanVideo-1.5 | CogVideoX-5B |
|---|---|---|---|---|---|
| Parameters | **15B** | ~13B | 14B | ~13B | 5B |
| Architecture | **Unified self-attention Transformer** | DiT | DiT | DiT | DiT |
| Native audio generation | ✅ **Yes (joint)** | ❌ | ❌ | ❌ | ❌ |
| Native lip-sync languages | **6** | 0 | 0 | 0 | 0 |
| Sampling steps | **8 (no CFG)** | ~25 | ~50 | ~50 | ~50 |
| 1080p generation time | **~38s on H100** | ~minutes | ~minutes | ~minutes | ~minutes |
| Text-to-video | ✅ | ✅ | ✅ | ✅ | ✅ |
| Image-to-video | ✅ (unified) | ✅ | ✅ | ✅ | ✅ |
| Open weights | ✅ | ✅ | ✅ | ✅ | ✅ |
| Commercial use | ✅ | ✅ (license-dep.) | ✅ (license-dep.) | ✅ (license-dep.) | ✅ (license-dep.) |

### How is Happy Horse 1.0 different from closed models like Sora, Veo, and Kling?

Sora 2, Veo 3.1, Kling 3.0, and Dreamina Seedance 2.0 are **closed, API-only services**. You pay per-minute, you cannot self-host, you cannot fine-tune, and you cannot inspect the model. Happy Horse 1.0 is being released as **open weights** that you download once and run forever on your own infrastructure. It also generates audio jointly with video in a single forward pass — most closed models still treat audio as a separate feature (Veo 3) or don't generate it at all (Kling, Runway).

For a complete side-by-side, including pricing and quality benchmarks, see the [Artificial Analysis Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video).

---

## 🎯 Use Cases

### Short-form social video
Generate scroll-stopping vertical video for **TikTok, Instagram Reels, YouTube Shorts**, and launch-day social content — with native sound — in under a minute per clip. No separate dubbing, no lip-sync tooling, no third-party music licensing for ambient audio.

### Marketing and ad creative
Build **launch trailers, product teasers, and high-converting ad creatives** with cinematic motion that looks directed rather than synthesized. Iterate quickly to A/B test multiple creative angles on tight launch timelines.

### Multilingual campaigns
Native lip-sync in six languages means you can produce a single creative concept and ship it across English, Chinese, Japanese, Korean, German, and French markets — without re-shooting and without paying for human dubbing studios.

### B-roll, establishing shots, and previz
Create **B-roll, concept shots, establishing footage, and stylized sequences** for film, TV, and YouTube production. Useful as drop-in footage for editors and as previz/animatics for directors before a shoot.

### E-commerce product video
Turn product photos into **photorealistic motion demos** with image-to-video. Show packaging reveals, device demos, and lifestyle scenes with stable camera movement and accurate physics — before any physical shoot.

### Indie filmmaking and storytelling
Multi-shot scenes with dialogue, music, and Foley generated together. The unified architecture maintains better subject and lighting consistency across shots than pipelines that stitch separate models together.

### AI research
Open weights for studying **joint audio-video diffusion**, **distillation methods (DMD-2)**, **unified multimodal Transformers**, and **timestep-free denoising**. The architecture is unusually clean — a useful reference implementation for researchers working on the next generation of generative video models.

---

## ❓ FAQ

### What is Happy Horse 1.0?
Happy Horse 1.0 is an open-source 15B-parameter AI video generation model that produces 1080p video with synchronized audio in a single forward pass. It uses a unified self-attention Transformer architecture and generates a clip in roughly 38 seconds on an H100. Visit [happyhorses.io](https://happyhorses.io/) for the live demo and the latest news.

### How is Happy Horse 1.0 different from Sora, Veo, and Kling?
Sora, Veo, and Kling are closed API services. Happy Horse 1.0 is **open source** — you can download the weights, self-host, fine-tune, and use commercially. Architecturally, it generates audio and video **jointly** in one pass, while most closed models either don't generate audio at all or generate it in a separate stage.

### How is it different from other open-source video models like Wan, Hunyuan, and LTX-2?
The biggest difference is **native joint audio-video generation**. Wan, HunyuanVideo, and LTX-2 generate silent video and require completely separate models for sound and lip-sync. Happy Horse 1.0 generates both modalities in the same Transformer at the same time. It also uses a pure self-attention architecture (no cross-attention) and ships with an aggressive DMD-2 distillation that gets it to 8 steps without classifier-free guidance.

### How fast is Happy Horse 1.0?
Roughly **38 seconds for a 1080p clip** on a single NVIDIA H100, and around 2 seconds for a 5-second 256p preview. Speed comes from three sources: DMD-2 distillation (8 steps instead of 50), no classifier-free guidance (cuts compute roughly in half), and MagiCompiler full-graph compilation (~1.2× additional speedup).

### What languages does the lip-sync support?
Native lip-sync is trained on **English, Mandarin Chinese, Japanese, Korean, German, and French**. The model can accept text prompts in other languages, but lip-sync accuracy is only guaranteed for the six trained languages.

### Is Happy Horse 1.0 really open source?
Yes. The release includes the base model, the distilled 8-step model, the super-resolution module, and inference code. The license permits commercial use and fine-tuning. Final license terms will be published in this repository at release.

### Can I run it on a consumer GPU like an RTX 4090?
The published 38-second figure is measured on an H100. A path for 24 GB consumer GPUs is expected at release with the distilled model and adjusted resolution settings. Final consumer hardware benchmarks will be added to this README at launch.

### When will the weights be released?
The repository is currently marked "coming soon." **Watch this repository** to be notified, or subscribe for updates at happyhorses.

### Can I fine-tune Happy Horse 1.0?
Yes. Fine-tuning is part of the announced release scope. Training and fine-tuning documentation will be published alongside the model.

### Can I use Happy Horse 1.0 for commercial work?
Yes. The release is announced as fully open source with commercial-use rights, including for advertising, e-commerce, YouTube monetization, client work, and enterprise production pipelines.

### Where can I see Happy Horse 1.0 ranked against other models?
On the [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena) and the [Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video). These are the most-cited public benchmarks for AI video model quality.

### Where can I try the model right now?
At **[happyhorses.io](https://happyhorses.io/)**. Free credits, no installation, no GPU required.

---

## 📜 Citation

A technical report will be published with the model release. Placeholder citation:

```bibtex
@misc{happyhorse2026,
  title  = {Happy Horse 1.0: An Open-Source Unified Self-Attention Transformer for Joint Audio-Video Generation},
  author = {Happy Horse Team},
  year   = {2026},
  url    = {https://happyhorses.io/},
  note   = {Repository: https://github.com/<org>/happy-horse}
}
```

---

## 🤝 Contributing

Contribution guidelines, issue templates, and a public roadmap will be published with the initial weight release. In the meantime:

- ⭐ **Star this repository** to be notified at launch
- 👀 **Watch releases** for the model drop
- 🌐 **Follow updates** at [happyhorses.io](https://happyhorses.io/)
- 🥊 **Vote in the arena** at the [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena)

---

## 📝 License

The model and code will be released under an open-source license that permits commercial use and fine-tuning. Exact license terms will be confirmed in this repository at the public release.

---

## ⚠️ Responsible Use

Happy Horse 1.0 is capable of generating photorealistic video and synchronized speech in six languages. Capability of this kind comes with real responsibility:

- **Do not** generate non-consensual likenesses of real people, including public figures, private individuals, or deceased persons.
- **Do not** generate deceptive political content, fabricated news footage, or impersonations intended to deceive.
- **Do not** generate sexual content involving minors, or any other content prohibited by applicable law.
- **Do** clearly disclose AI-generated content in journalism, advertising, education, and other contexts where the audience has a reasonable expectation of authenticity.
- **Do** respect copyright, trademarks, and likeness rights in your prompts and outputs.

A complete acceptable-use policy will accompany the model release.

---

<p align="center">
  <strong>🐎 Happy Horse 1.0 — open-source video generation, with sound, in seconds.</strong><br>
  <a href="https://happyhorses.io/">happyhorses.io</a>
</p>
