# MentorConnect-Cross-College-Mentor-Student-Platform-
# MentorConnect — Cross-College Mentor-Student Platform

> A two-sided marketplace connecting students outside top-tier colleges with verified mentors — built end-to-end, from identity verification to live demand-based mentor matching.

---

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Identity Verification Pipeline](#identity-verification-pipeline)
- [Mentor Matching — Live Bidding](#mentor-matching--live-bidding)
- [My Role](#my-role)
- [Challenges & What I Learned](#challenges--what-i-learned)
- [Roadmap](#roadmap)
- [Status](#status)

---

## Overview

Most mentorship platforms assume mentors and students are already at top-tier institutions. MentorConnect is built for the opposite case — students outside the IIT/NIT bracket who still want access to verified, credible mentors, and mentors who want a fair, demand-driven way to price their time instead of a flat listing fee.

The platform covers the full loop: landing page → signup → identity verification → mentor discovery → live-bid matching.

## Key Features

- **10-step signup flow** for both mentors and students, with OTP verification across college email, personal mobile, and personal email — three independent checks before an account goes live.
- **Age-aware consent flow** — a dedicated parental-consent step is automatically inserted for users aged 16–17, and skipped for everyone else, without a separate signup path.
- **Document + face identity verification** — every mentor and student is verified against a real ID document before they can transact on the platform.
- **Live bidding mentor marketplace** — instead of a fixed hourly rate, mentors are discovered through a live bidding round with a price slider, so value is set by real-time demand rather than a static listing price.
- **Trust score engine** — a single composite score built from four independent checks, with hard override rules so a duplicate identity or a clearly failed check is rejected outright regardless of the aggregate score.
- **Abuse-prevention / rate-limiting layer** — session-aware request throttling to stop signup and verification abuse.

## Tech Stack

**Backend & ML Service**
`FastAPI` · `Uvicorn` · `Pydantic` · `OpenCV (cv2)` · `NumPy` · `Pillow` · `imagehash` · `python-dateutil`

**OCR & Computer Vision**
`PaddleOCR` · `PaddlePaddle` · `PyTorch` · `HuggingFace Transformers` · `InsightFace` · `ONNX Runtime` · `Qwen2.5-VL` (vision-language model, second-opinion verification)

**Realtime / Node Backend**
`TypeScript` · `ws` (WebSocket) · `ioredis` (Redis client) · `nanoid` · `dotenv` · `node-fetch` · `tsx`

**Frontend**
`React 19` · `TypeScript` · `TanStack Start` / `TanStack Router` / `TanStack Query` · `Zustand` · `React Hook Form` · `Zod` · `Tailwind CSS v4` · `Radix UI` · `shadcn/ui` · `Framer Motion`

**3D / Landing Page Visuals**
`three.js` · `@react-three/fiber` · `@react-three/drei` · `three-globe` · `cobe` · `react-globe.gl` · `canvas-confetti`

## System Architecture

```
┌─────────────┐      ┌──────────────────┐      ┌────────────────────┐
│   Landing   │ ───▶ │   Signup Flow     │ ───▶ │  Identity Verify    │
│  (React 19) │      │  (10-step, OTP)   │      │  (FastAPI + CV/OCR) │
└─────────────┘      └──────────────────┘      └─────────┬──────────┘
                                                            │
                                                   trust score engine
                                                            │
                                              ┌─────────────▼─────────────┐
                                              │   Mentor Discovery Feed    │
                                              │   + Live Bidding Engine    │
                                              │   (WebSocket, Redis)       │
                                              └────────────────────────────┘
```

Real-time bidding state and abuse-prevention counters live in Redis via `ioredis`; the WebSocket layer (`ws`) pushes live bid updates to connected clients without polling.

## Identity Verification Pipeline

The verification pipeline went through a real iteration cycle rather than being built once and shipped:

1. **First pass — EasyOCR.** Worked on clean, well-lit document scans but accuracy dropped sharply on real, skewed phone photos — which is what most users actually upload.
2. **Second pass — PaddleOCR.** Meaningfully more robust to skew, glare, and low-resolution captures.
3. **Third pass — added a vision-language model (Qwen2.5-VL)** as a second opinion layer, to catch cases OCR alone missed (e.g. partially obscured fields, non-standard document layouts).
4. **Trust score** — combines four independent checks (document authenticity, liveness, face match, data match) into one weighted score, with override rules: a failed liveness check or a duplicate identity match rejects the account outright, regardless of what the composite score says.

A real production bug was found and fixed here: the abuse-prevention system was keying off an ID that silently reset on every page load, meaning rate limiting had never actually been enforced. Fixing it required tracing the session-identity lifecycle end-to-end.

## Mentor Matching — Live Bidding

Rather than a fixed listing price, mentor sessions are matched through a live bidding round with a price slider — mentors set a floor, students bid up in real time, and the session is priced by actual demand rather than a static rate card decided in advance.

## My Role

Founder & Product Lead. I formed and coordinated the build team, made the core product and architecture calls (verification pipeline design, trust-score weighting, the bidding-over-fixed-price decision), and worked hands-on across both the ML service and the signup/consent flow logic. Frontend implementation was accelerated using AI-assisted tooling (Lovable, Claude) for UI velocity, while backend logic, model integration, and system design decisions were mine.

## Challenges & What I Learned

- Real-world documents are messy — the OCR iteration (EasyOCR → PaddleOCR → VLM second opinion) taught me that "works on the demo photo" and "works on a real user's photo" are two different bars.
- Composite trust scores need hard overrides, not just weighted averages — a single failed liveness check should never be "averaged away" by three passing checks.
- Silent bugs are the dangerous ones — the rate-limiting bug shipped invisibly because it failed *open*, not closed.

## Roadmap

- Expand verification to additional document types beyond the current set
- Add mentor rating/review loop post-session
- Move bidding history into a public, searchable ledger for transparency

## Status

Actively developed. This repository is a working build, not a finished/polished open-source release — expect ongoing changes.

---
*Built by Nusrat Ali — [LinkedIn](https://www.linkedin.com/in/nusrat-ali-47073b329)*
