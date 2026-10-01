# 🎬 ReelMind — Instagram-Grade Reel Recommendation Engine

A production-grade, research-focused recommendation engine for short-form video content, implementing the same two-stage retrieval + ranking pipeline used by Instagram, TikTok, and YouTube Shorts.

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     ReelMind System                         │
├──────────────┬──────────────────────┬───────────────────────┤
│  Data Layer  │   ML Pipeline        │   Serving Layer       │
│              │                      │                       │
│  MongoDB     │  1. Data Collector   │  Node.js + Express    │
│  Redis       │  2. Feature Eng.     │  REST API             │
│  KuaiRec     │  3. Two-Tower Model  │  WebSocket Feed       │
│  MicroLens   │  4. Ranker (LightGBM)│  Redis Cache          │
│  Pinterest   │  5. Re-ranker        │  A/B Testing          │
│  Trends API  │  6. Online Learning  │  Metrics Dashboard    │
└──────────────┴──────────────────────┴───────────────────────┘
```

## 📦 Stack

| Layer | Technology |
|-------|-----------|
| Backend API | Node.js + Express |
| Database | MongoDB (content + users) |
| Cache | Redis (feeds + embeddings) |
| ML Engine | Python + PyTorch |
| Vector Store | FAISS (local) / Pinecone (cloud) |
| Feature Store | Redis + MongoDB |
| Data Pipeline | Python (pandas, asyncio) |
| Message Queue | BullMQ (Redis-backed) |
| Monitoring | Custom metrics dashboard |

## 🧠 ML Pipeline

### Stage 1: Candidate Retrieval (Two-Tower Model)
- User Tower: embeds user history, demographics, time-of-day
- Item Tower: embeds video metadata, audio, tags, engagement stats
- ANN search via FAISS returns top-1000 candidates

### Stage 2: Ranking (LightGBM + Neural Ranker)
- Cross-features between user + item
- Engagement velocity signals
- Diversity injection (MMR)
- Business rules (content policies, freshness boost)

### Stage 3: Re-ranking
- Diversity (same creator / topic deduplication)
- Freshness decay
- Exploration vs exploitation (ε-greedy / UCB)
- A/B test bucket assignment

## 📊 Data Sources

| Source | Type | Use |
|--------|------|-----|
| KuaiRec dataset | Public benchmark | Offline training |
| MicroLens dataset | Public benchmark | Offline training |
| First-party logs | User interactions | Online training |
| Pinterest Trends API | Trending topics | Content signals |
| Synthetic generator | Simulated users | Cold start testing |

## 🗂️ Project Structure

```
reelmind/
├── data/                   # Datasets and data processing
│   ├── collectors/         # API data collectors
│   ├── processors/         # Feature engineering
│   └── datasets/           # Raw datasets storage
├── ml/                     # Machine learning pipeline
│   ├── models/             # PyTorch model definitions
│   │   ├── two_tower.py    # Two-Tower retrieval model
│   │   ├── ranker.py       # LightGBM + Neural ranker
│   │   └── embeddings.py   # Embedding utilities
│   ├── training/           # Training scripts
│   ├── evaluation/         # Offline evaluation (NDCG, HR@K)
│   └── serving/            # Model export + ONNX
├── backend/                # Node.js API server
│   ├── src/
│   │   ├── routes/         # API endpoints
│   │   ├── services/       # Business logic
│   │   ├── models/         # MongoDB schemas
│   │   ├── workers/        # BullMQ background jobs
│   │   └── middleware/     # Auth, rate limiting, logging
│   └── package.json
├── feature_store/          # Feature engineering service (Python)
├── scripts/                # Setup, seed, and utility scripts
├── docker-compose.yml      # Full stack orchestration
└── config/                 # Environment configs
```

## 🚀 Quick Start

```bash
# 1. Install all dependencies
./scripts/setup.sh

# 2. Download and prepare datasets
python data/collectors/download_datasets.py

# 3. Start infrastructure
docker-compose up -d

# 4. Train models
python ml/training/train_two_tower.py
python ml/training/train_ranker.py

# 5. Start API server
cd backend && npm run dev

# 6. Test recommendations
curl http://localhost:3000/api/v1/recommendations/user123
```

## 📈 Evaluation Metrics

- **Retrieval**: Recall@100, Recall@1000
- **Ranking**: NDCG@10, HR@10, MRR
- **Online**: CTR, Watch-time, Session length, Return rate
- **Diversity**: ILD (Intra-List Diversity), Coverage
# inst-rec
