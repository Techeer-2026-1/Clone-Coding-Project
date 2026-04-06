# LocalBiz Intelligence

서울의 파편화된 로컬 생활 정보를 통합하여 자연어 채팅으로 맞춤 코스와 일정을 제안하는 AI 어시스턴트.

## 레포지토리 구조

| 레포 | 설명 | 링크 |
|------|------|------|
| **localbiz-backend** | FastAPI + LangGraph 백엔드, 비정형 데이터 ETL | [바로가기](https://github.com/Techeer-2026-1/localbiz-backend) |
| **localbiz-frontend** | Next.js 14 채팅 UI | [바로가기](https://github.com/Techeer-2026-1/localbiz-frontend) |
| **localbiz-etl** | 정형 데이터 ETL (공공 CSV → PostgreSQL) | [바로가기](https://github.com/Techeer-2026-1/localbiz-etl) |

## 기술 스택

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14, Tailwind CSS, Leaflet.js, Recharts |
| Backend | FastAPI, LangGraph, WebSocket |
| LLM | Gemini 2.5 Flash (런타임), Claude Haiku (배치 캡셔닝) |
| 임베딩 | Gemini gemini-embedding-001 (768d, 무료) |
| Vector DB | OpenSearch 2.17 (k-NN + nori 한국어 분석기) |
| Relational DB | PostgreSQL 16 + PostGIS 3.4 |
| Cache | Redis 7.4 |
| Monitoring | Grafana + Prometheus + Loki + Jaeger |

## 아키텍처

```
Client (Next.js / Vercel)
    ↓ WebSocket
FastAPI + LangGraph (Google Compute Engine)
    ├── Intent Router (12+1 intent)
    ├── 전용 노드 5개 (place_search, place_recommend, event_search, course_plan, image_search)
    ├── ReAct 에이전트 2개 (search_agent, action_agent)
    └── Response Composer → typed 블록 스트리밍
    ↓
Cloud SQL(PostgreSQL+PostGIS) ←→ OpenSearch(GCE) ←→ Redis (캐시)
    ↓                                                    ↓
External APIs: Gemini, Claude, Google Places,    Monitoring(GCE):
  Naver Blog/News, Google Calendar(MCP),         Grafana+Prometheus+Loki+Jaeger → Slack
  서울 열린데이터
```

## 빠른 시작

### 백엔드

```bash
git clone https://github.com/Techeer-2026-1/localbiz-backend.git
cd localbiz-backend
cp .env.example .env   # API 키만 채우기 (DB/OpenSearch 접속 정보는 이미 있음)
python3.11 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python -m uvicorn src.entry:app --host 0.0.0.0 --port 8000 --reload
```

> Docker 불필요 — Cloud SQL(34.47.71.231) + OpenSearch(34.50.28.16) 공동 서버 직접 연결

### 프론트엔드

```bash
git clone https://github.com/Techeer-2026-1/localbiz-frontend.git
cd localbiz-frontend
npm install
npm run dev   # http://localhost:3000
```

### 데이터 적재 (ETL)

```bash
# 정형 데이터 (공공 CSV → PostgreSQL)
git clone https://github.com/Techeer-2026-1/localbiz-etl.git
cd localbiz-etl && python etl.py

# 비정형 데이터 (리뷰/가격/임베딩 → OpenSearch)
cd localbiz-backend
python scripts/load_places_vector.py          # 장소 임베딩
python scripts/batch_review_analysis.py --batch  # 리뷰 분석
python scripts/load_place_reviews.py          # 리뷰 임베딩
python scripts/collect_price_data.py          # 가격 수집
python scripts/load_events_vector.py          # 행사 임베딩
```

## 팀원

| 이름 | 역할 |
|------|------|
| 이정 | BE/PM — LangGraph 에이전트, 비정형 데이터 ETL |
| 정조셉 | BE — 정형 데이터 ETL, DB 설계 |
| 한정수 | BE |
| 강민서 | BE |
| 이정원 | FE — Next.js 채팅 UI |

## 기획 문서

기획서 및 상세 기술 문서는 각 레포의 `기획/` 디렉토리 또는 프로젝트 Wiki를 참고하세요.
