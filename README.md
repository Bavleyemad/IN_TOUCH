## What is IN_TOUCH?

A real-time sign language recognition and translation mobile app. Point your camera at someone signing in ASL, and we give you text. Want to go the other way? Type something and watch an avatar sign it back.

We also built a community platform for the deaf and hard-of-hearing community, because the tech is only half the story.

This isn't a demo. It's a full-stack production system — mobile client, backend API, and a custom-trained deep learning model running live inference.



## Features

 🤟 Sign → Text
Your camera feeds into a **MediaPipe** skeleton extraction pipeline that tracks **57 joints** across both hands, pose, and face. That skeleton goes into our **ST-GCN** (Spatial-Temporal Graph Convolutional Network) model — trained on WLASL, MS-ASL, and Sem-Lex — and comes out the other side as text.

We support **continuous sentence inference** too, not just isolated signs. Motion-energy segmentation handles the word boundaries automatically so you can sign naturally.

 ✍️ Text → Sign
Type in text, get back an animated avatar performing the signs. No more static image dictionaries.

Video  →  sign
Drop in a local video file or paste a YouTube URL and the pipeline handles the rest

### 🌐 Community Platform
Forums, shared resources, and a space that actually belongs to the community. Built with real-time features via **SignalR**.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Mobile App (Flutter)                  │
│         Camera  →  MediaPipe  →  Skeleton Frames        │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTP / SignalR
          ┌───────────┴────────────┐
          │                        │
┌─────────▼──────────┐   ┌────────▼────────────┐
│  ASP.NET Core API  │   │   FastAPI AI Server │
│  (Clean Arch)      │   │   (HuggingFace 🤗)  │
│  SQL Server        │   │   ST-GCN v27        │
│  Firebase          │   │   ~750K params      │
└────────────────────┘   └─────────────────────┘
```

---

## The Model — What's Under the Hood

We spent an embarrassing number of nights on this. Here's what v27 ended up being:

| Property | Value |
|---|---|
| Architecture | ST-GCN |
| Parameters | ~750K |
| Input Channels | 6 (position, velocity, acceleration, bone vectors, etc.) |
| Frames | 64 |
| Joint Schema | 57 joints (21 dominant + 21 passive hand + 9 pose + 6 face) |
| Signs Supported | 60 ASL signs |
| Loss Function | PolyLoss (ε = 2.0) |
| Optimizer | AdamW + SGDR with snapshot ensembling |
| Test-Time Augmentation | TTA × 5 |
| Validation Signal | MS-ASL (signer-independent) |

We went from ~1.3M parameters down to ~750K without sacrificing accuracy — just cleaner architecture and smarter training. The model's deployed on HuggingFace Spaces under [`a7medwa2l/INTOUCHFeature2`](https://huggingface.co/spaces/a7medwa2l/INTOUCHFeature2).

---

## Tech Stack

**Mobile**
- Flutter — cross-platform, one codebase

**Backend**
- ASP.NET Core — Clean Architecture (no spaghetti, we promise)
- SQL Server
- SignalR — real-time community features
- Firebase — notifications & auth

**AI Serving**
- FastAPI — lightweight, async, exactly what we needed
- MediaPipe — skeleton extraction, 57-joint schema
- ST-GCN — custom-trained, not a pretrained checkpoint we just fine-tuned

**Data Sources**
- WLASL (raw videos + frame folders + `.npy` cache)
- MS-ASL
- Sem-Lex

---

## Project Structure

```
IN_TOUCH/
├── mobile/              # Flutter app
├── backend/             # ASP.NET Core (Clean Architecture)
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── API/
├── ai-server/           # FastAPI inference server
│   ├── model/           # ST-GCN weights
│   ├── pipeline/        # MediaPipe + preprocessing
│   └── endpoints/       # Sentence & isolated inference
└── notebooks/           # Training + evaluation notebooks
```

---

## Getting Started

### Prerequisites

- Flutter SDK 3.x
- .NET 8 SDK
- Python 3.10+
- SQL Server
- Firebase project

### AI Server (FastAPI)

```bash
cd ai-server
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

The model checkpoint gets loaded on startup. First inference takes a second — after that it's fast.

### Backend (ASP.NET Core)

```bash
cd backend
dotnet restore
dotnet ef database update
dotnet run --project API
```

Update `appsettings.json` with your SQL Server connection string and Firebase config.

### Mobile App (Flutter)

```bash
cd mobile
flutter pub get
flutter run
```

Point the base URL in your env config to wherever the backend is running.

---

## Inference Modes

We built two approaches for sentence-level inference:

**Approach A — Multiple Videos**
Feed in pre-segmented video clips, one per sign. Good for controlled environments.

**Approach B — Continuous Video with Auto-Segmentation**
One continuous video stream. Motion-energy segmentation detects where signs start and end automatically. This is what powers the live camera feature.

**Approach C — Video File / YouTube Import No camera needed. Drop in a local video file or paste a YouTube URL —
the pipeline downloads it, runs the full skeleton extraction + segmentation + inference stack on it, and hands
you back the translation. Good for processing existing content, recorded lectures, or anything you can't (or don't want to) do in real time.
---


## Acknowledgements

WLASL, MS-ASL, and Sem-Lex dataset creators — without open data, none of this exists.

The deaf and hard-of-hearing community who gave us a reason to actually care about getting this right.

---
*If this helped you in any way, leave a ⭐ — it costs nothing and means a lot.*

</div>
