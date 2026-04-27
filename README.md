# Happy Horse 1.0 — AI Video Generator Information Collection

> ## ⚠️ Disclaimer — Please Read First
>
> **This is a personal information-collection repository, not an official Happy Horse repository and not a public release.**
>
> Happy Horse 1.0 has **not yet been officially open-sourced**. No model weights, no inference code, and no official GitHub repository have been published yet. Everything in this README is gathered from publicly circulating sources — community-compiled architecture notes, alleged technical leaks, the Artificial Analysis Video Arena, project landing pages, and independent discussion threads — and is collected here for personal reference and learning.
>
> I am not affiliated with the Happy Horse team. **The information below may be incomplete, inconsistent across sources, or change at any time.** I am watching for updates and will sync this README the moment the official model, repository, or technical report is released.
>

## Latest HappyHorse 1.0 information(2026.4.27)

HappyHorse 1.0, officially translated as **“快乐小马”**, is Alibaba ATH Innovation Unit’s latest video generation and editing model. It started grayscale testing in China on April 27, 2026.

### Official Websites

- China official website: `www.happyhorse.cn`
- Try HappyHorse now: [happyhorse](happyhorses.io)

### Available Platforms

HappyHorse 1.0 can be experienced through:

- Alibaba Cloud Bailian
- HappyHorse official website
- Qwen App
- Alibaba Wukong
- MuleRun
- JVS Claw

### Benchmark Performance

On Arena.ai, HappyHorse 1.0 ranked second in:

- Text-to-video
- Image-to-video
- Video editing

It ranked behind ByteDance Seedance 2.0.

### Capabilities

- Text-to-video generation
- Image-to-video generation
- Video editing
- 3s–15s video generation
- Multi-shot generation
- Coherent short-form storytelling
- Up to 1080p resolution
- Up to 4 videos generated at the same time

### Pricing

| Resolution | Listed Price | Discounted Pro Price |
|---|---:|---:|
| 720p | RMB 0.9 / second | RMB 0.44 / second |
| 1080p | RMB 1.6 / second | RMB 0.78 / second |

### Strengths

- Fast generation speed: around 2–5 minutes per video
- Strong prompt-following ability
- Good control over camera movement, composition, style, and atmosphere
- High reference fidelity in image-to-video generation
- Supports multi-shot and coherent short video storytelling

### Limitations

- Audio-video synchronization still needs improvement in complex scenes
- Longer videos may contain physical inconsistencies
- Text rendering may produce garbled or incorrect characters

---

## 📖 What is Happy Horse 1.0?

**Happy Horse 1.0** is described as an open-source state-of-the-art **AI video generator** with **native joint audio-video generation** — meaning the Happy Horse AI video model produces video frames and the corresponding audio track (dialogue, ambient sound, Foley) together in a single forward pass, rather than generating silent video and dubbing it afterward.

According to community-compiled architecture notes, the model is built around a **15-billion-parameter unified self-attention Transformer** that processes text, image, video, and audio tokens within a single token sequence. It is reportedly built without dedicated cross-attention branches and without a separate audio module. Combined with **DMD-2 distillation**, the distilled variant is reported to generate 1080p video in roughly **38 seconds on an NVIDIA H100**, using only **8 denoising steps without classifier-free guidance**.

The Happy Horse AI video generator first surfaced publicly as a "mystery model" on the [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena), where it appeared anonymously alongside frontier closed models from ByteDance, Kling, and Google. Independent community observers noted that it appeared to come from Asia and that — unusually for a mystery entry — it included native audio output.

As of this writing, the announcement promises that the public release will include the **base model**, the **distilled 8-step model**, the **super-resolution module**, and **inference code**. None of these have been published yet. You can follow the [official Happy Horse platform](https://happyhorses.io/) for release news.

---

## 🧱 Reported Architecture

> The technical details below come from community-compiled architecture notes and alleged technical leaks circulating about Happy Horse 1.0. None of this has been confirmed by an official technical report or peer review. Treat as plausible but unverified until the model is officially released.

| Component | Reported Specification |
|---|---|
| Total parameters | ~15B |
| Architecture | Unified self-attention Transformer (reportedly without dedicated cross-attention branches) |
| Total layers | 40 |
| Layer layout | Sandwich — first 4 + last 4 layers use modality-specific projections; middle 32 layers reportedly share parameters across all modalities |
| Modalities processed | Text, image, video, and audio tokens — concatenated into a single token sequence |
| Multimodal fusion | Per-attention-head learned scalar gates with sigmoid activation, used for training stability |
| Conditioning | Reference image and denoising signals routed through one minimal unified interface — reportedly no dedicated conditioning branches |
| Timestep handling | Reportedly no explicit timestep embeddings — the model is said to infer the denoising state directly from the noise level of input latents |
| Distillation | DMD-2 (Distribution Matching Distillation v2) |
| Sampling steps | 8 |
| Classifier-free guidance | Reportedly not required |
| Inference runtime | MagiCompiler — full-graph compilation with operator fusion across Transformer layers, reported to give ~1.2× end-to-end speedup |
| Reference GPU | NVIDIA H100 80GB |

### Why these reported design choices matter

**Unified self-attention instead of cross-attention.** Most existing open-source video models — Wan 2.2, HunyuanVideo, LTX-2, CogVideoX — use a Diffusion Transformer (DiT) backbone where text conditioning is injected via cross-attention from a separate text encoder, and audio (when present) is generated by a different model entirely. Happy Horse 1.0 is reported to rely entirely on unified self-attention, putting text, image, video, and audio into the same sequence and letting attention handle all of it. The claim is that this lets the model learn audio-video alignment as a fundamental part of denoising rather than as a downstream fix-up step.

**Sandwich layer layout.** The first 4 and last 4 layers reportedly handle modality-specific embedding and decoding; the middle 32 layers share parameters across all modalities. The reported benefit is parameter efficiency — most of the network does cross-modal reasoning instead of being split into siloed sub-networks.

**Per-head sigmoid gating.** Joint multimodal training is notoriously unstable, because gradients from the audio loss can dominate or be dominated by gradients from the video loss. The reported solution: a learned scalar gate on each attention head, allowing the model to selectively dampen heads producing destructive gradients in a particular modality.

**Timestep-free denoising.** Most diffusion models feed the current denoising timestep into every layer as an explicit embedding. Happy Horse 1.0 is reported to omit this entirely, on the basis that the noise level is already encoded in the noisy latents themselves. This is described as one of the prerequisites for the aggressive 8-step DMD-2 distillation.

**DMD-2 distillation.** Standard video diffusion typically needs 25–50 sampling steps with classifier-free guidance, both of which double or triple inference cost. DMD-2 (Distribution Matching Distillation v2) trains a student model to match the teacher's output distribution in just 8 steps with no CFG. This is what the reported "1080p in ~38 seconds" figure rests on.

---

## ✨ Reported Features of the Happy Horse AI Video Generator

### Native joint audio-video generation
This is the defining reported feature. A single unified Transformer is said to denoise video tokens and audio tokens together in the same sequence. Dialogue, Foley, and ambient sound are produced in one pass and are inherently aligned to the visuals — no separate dubbing or lip-sync model needed.

### 1080p output
Reported to generate up to 1080p video at clip lengths of roughly 5–10 seconds, with multiple aspect ratios.

### Native lip-sync in 6 languages
According to the technical project description, Happy Horse 1.0 lip-sync is natively trained for **English, Mandarin Chinese, Japanese, Korean, German, and French** with low Word Error Rate. (One marketing-oriented page lists 7 languages including Cantonese — the official count should be confirmed at release.)

### ~38 seconds for 1080p
Reported wall-clock generation time on an NVIDIA H100 using the distilled Happy Horse model. A 5-second 256p preview is reported at around 2 seconds.

### Unified text-to-video and image-to-video
A single set of weights is reported to handle both **text-to-video** and **image-to-video** tasks within the same Happy Horse pipeline.

### Open-source release scope
The announced release scope includes the base Happy Horse 1.0 model, the distilled 8-step model, the super-resolution module, and the inference code. License terms have not been published.

---

## 📊 Public Benchmarks

The most-cited public benchmark for AI video models is the **Artificial Analysis Video Arena**, which uses blind head-to-head voting to compute Elo ratings. Happy Horse 1.0 has been observed competing in the arena under its codename, but as of this writing it is not yet listed in the official public leaderboard tables.

Authoritative leaderboards to watch for the model's eventual public placement:

- 🥊 [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena) — vote on blind comparisons
- 📈 [Artificial Analysis Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video)
- 🖼️ [Artificial Analysis Image-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/image-to-video)

> **Note on rankings:** Elo ratings on the Artificial Analysis arena are recomputed continuously as new votes come in, and a model's rank can move several places within a single day. The tier table below is a **point-in-time snapshot** of the Text-to-Video leaderboard as of the date this README was last updated. For the most current ordering, always check the live leaderboards linked above.

### Current top tier of the Text-to-Video leaderboard (snapshot, for context)

Where Happy Horse 1.0 eventually places will be most meaningful when read against the existing field. At the time of writing, top entries on the Artificial Analysis Text-to-Video Leaderboard fall roughly into these tiers:

| Tier | Models |
|---|---|
| **Frontier closed models** (Elo ~1,200–1,275) | Dreamina Seedance 2.0, SkyReels V4, Kling 3.0, PixVerse V6, Veo 3.1, Runway Gen-4.5 |
| **Mid-tier closed models** (Elo ~1,150–1,200) | Sora 2 Pro, Hailuo 2.3, Wan 2.6, Vidu Q2 |
| **Top open-weights models** (Elo ~1,100–1,135) | LTX-2 Pro, LTX-2 Fast, Wan 2.2 A14B |
| **Earlier open-weights** (Elo ~950–1,020) | HunyuanVideo-1.5, Wan 2.1 14B, Wan 2.2 5B, Hunyuan Video |

In this landscape, anything that places at or above the LTX-2 line is considered the open-source state of the art. Anything in the frontier closed tier is competing directly with the best paid APIs. I will update this section with Happy Horse 1.0's verified placement once it appears on the official leaderboard.

---

## 🆚 Happy Horse 1.0 vs. Wan 2.2, LTX-2, and Other Open Video Models

A frequently asked question in the AI video community is how Happy Horse 1.0 stacks up against the current open-source leaders. Based on reported specifications, here is the comparison:

| Feature | **Happy Horse 1.0 (reported)** | LTX-2 Pro | Wan 2.2 A14B | HunyuanVideo-1.5 | CogVideoX-5B |
|---|---|---|---|---|---|
| Parameters | ~15B | ~13B | 14B | ~13B | 5B |
| Backbone | Unified self-attention Transformer | DiT | DiT | DiT | DiT |
| Native audio generation | ✅ joint with video | ❌ | ❌ | ❌ | ❌ |
| Native lip-sync languages | 6 (reported) | 0 | 0 | 0 | 0 |
| Sampling steps | 8 (no CFG) | ~25 | ~50 | ~50 | ~50 |
| Reported 1080p time | ~38s on H100 | minutes | minutes | minutes | minutes |
| Text-to-video | ✅ | ✅ | ✅ | ✅ | ✅ |
| Image-to-video | ✅ unified | ✅ | ✅ | ✅ | ✅ |
| Open weights available *today* | ❌ not yet | ✅ | ✅ | ✅ | ✅ |

The headline differentiator on paper is **native joint audio-video generation**. Wan 2.2, HunyuanVideo, LTX-2, and CogVideoX all generate silent video and require entirely separate models for sound and lip-sync. The headline disclaimer is the last row: **all of those competitors have already released weights, while Happy Horse 1.0 has not.**

---

## 🎯 Possible Use Cases

Based on the announced capabilities, the Happy Horse AI video generator — once released — would be relevant for the following workflows. None of this has been verified by hands-on testing yet.

- **Short-form social video** — TikTok, Reels, and Shorts content with native sound, no separate dubbing pipeline.
- **Marketing and ad creative** — launch trailers, product teasers, and high-converting ad creatives with cinematic motion.
- **Multilingual campaigns** — running a single creative concept across English, Chinese, Japanese, Korean, German, and French markets without re-shooting.
- **B-roll and previz** — establishing shots, concept footage, and animatics for film, TV, and YouTube production.
- **E-commerce product video** — turning product photos into motion demos via image-to-video.
- **AI research** — open weights for studying joint audio-video diffusion, unified multimodal Transformers, DMD-2 distillation, and timestep-free denoising, once published.

---

## ❓ FAQ

### Is Happy Horse 1.0 open source right now?
**No.** As of this README, the Happy Horse model weights, inference code, and an official repository have not been published. The release is announced as "coming soon." This repository is a personal information collection while waiting for that release.

### Is this the official Happy Horse repository?
**No.** This is a personal collection of public information. I am not affiliated with the Happy Horse team. For anything authoritative, refer to the [official Happy Horse platform](https://happyhorses.io/).

### When will the Happy Horse model weights be available to download?
No public release date has been announced yet. The Happy Horse 1.0 model weights, the distilled 8-step variant, the super-resolution module, and the inference code are all listed in the announced release scope but have not yet been uploaded. I will sync this README the moment any of them go live, and you can also watch the [official Happy Horse update channel](https://happyhorses.io/) for release news.

### What architecture does the Happy Horse AI video generator use?
According to community-compiled architecture notes, a 15B-parameter unified self-attention Transformer with 40 layers, sandwich layout (4 modality-specific layers at each end + 32 shared middle layers), per-head sigmoid gating, no dedicated cross-attention branches, no explicit timestep embeddings, and DMD-2 distillation to 8 sampling steps without CFG.

### How fast is Happy Horse 1.0 supposed to be?
The reported figure is ~38 seconds for a 1080p clip and around 2 seconds for a 5-second 256p preview, both on a single NVIDIA H100. These numbers have not been independently reproduced.

### What languages will Happy Horse 1.0 support for lip-sync?
The technical description lists 6 languages: English, Mandarin Chinese, Japanese, Korean, German, and French. A separate marketing page lists 7 (adding Cantonese). The official count should be confirmed at release.

### How does Happy Horse 1.0 compare to Wan 2.2 and LTX-2?
On paper, Happy Horse 1.0 has roughly the same parameter count as the current top open-weights models (Wan 2.2 A14B at 14B, LTX-2 Pro at ~13B), but adds native joint audio-video generation that none of those competitors have. In practice, Wan 2.2 and LTX-2 are downloadable today and Happy Horse is not — so any direct comparison is theoretical until the weights ship.

### Where can I see the Happy Horse AI video generator in the meantime?
On the [official Happy Horse demo platform](https://happyhorses.io/), and — once it appears in the public tables — on the [Artificial Analysis Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video).

### What license will the Happy Horse model use?
Not yet announced. The release is described as fully open source with commercial use permitted, but the exact license has not been published.

### Will this README be updated?
Yes. I am tracking the official site, the Artificial Analysis arena, and community discussion. As soon as anything official changes — repository goes live, weights are uploaded, a paper is posted, the leaderboard updates with verified Elo numbers — this README will be synced.

---

## 📚 Sources & Updates

This README is compiled from publicly circulating information about Happy Horse 1.0 as of the date below. Two categories of sources are referenced:

- **Official information channel** — the [official Happy Horse AI video generator site](https://happyhorses.io/) is the source of record for live demos and updates. All other promotional pages discovered during research have been intentionally **not linked** here, since they are third-party and their claims occasionally contradict each other.
- **Authoritative third-party benchmark** — the [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena) and its [text-to-video](https://artificialanalysis.ai/video/leaderboard/text-to-video) and [image-to-video](https://artificialanalysis.ai/video/leaderboard/image-to-video) leaderboards are the only third-party sources linked in this document, because they are widely cited as the standard public benchmark for AI video models.

**Last updated:** April 8, 2026
**Status:** Pre-release · awaiting official open-source publication
**Will be updated when:** the official repository goes live, the model weights are published, a technical report is released, or the model appears in the public Artificial Analysis leaderboard tables.

---

<p align="center">
  <em>This is a personal information-collection repository.<br>
  Happy Horse 1.0 has not yet been officially open-sourced — this README will be synced as soon as it is.</em>
</p>
