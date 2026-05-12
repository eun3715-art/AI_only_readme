# AI_only_readme
# 공장마켓 — 산업 전문 M&A 플랫폼

공장·산업시설의 매매·임대·M&A를 중계하는 플랫폼
산업기계기술사 수준의 자동 기술 진단 점수를 제공한다.

---

## 기술 스택

| 영역 | 기술 |
|---|---|
| 프론트엔드 | React 18 + TypeScript + Vite |
| 상태/데이터 | TanStack Query v5 |
| 라우팅 | React Router v6 |
| 지도 | Leaflet + OpenStreetMap (react-leaflet) |
| 백엔드 | Node.js + Express + TypeScript |
| 데이터베이스 | Supabase (PostgreSQL + Auth + Storage) |

---

## 주요 기능

- **매물 목록** — 업종별 프리셋 필터(금형/자동차부품/식품 등 8종) + 상세 필터(전력·층고·바닥하중·호이스트 등)
- **기술사 자동 진단** — 층고·바닥하중·계약전력·폐수처리 등 18개 규칙으로 0~100점 산출
- **매물 상세** — 이미지 갤러리, 스펙 그리드, 기술 진단 보고서, OpenStreetMap 위치 지도
- **M&A 가치평가 계산기** — 정률법 감가상각 기반 인수가 산출 (백엔드 API)
- **매물 등록** — 사진 최대 8장 업로드 + 전체 스펙 입력 폼
- **전문가 상담 신청** — 모달 폼으로 문의 접수
- **Supabase 인증** — 이메일 로그인/회원가입, 미로그인 시 등록 페이지 접근 제한
- **Mock 모드** — Supabase 미연결 시 내장 샘플 데이터 6건으로 전체 기능 동작

---

## 빠른 시작

### 1. 저장소 클론 & 의존성 설치

```bash
git clone <repo-url>

# 프론트엔드
cd frontend
npm install

# 백엔드 (M&A 가치평가 API)
cd ../backend
npm install
```

### 2. 환경 변수 설정 (선택 — 없으면 Mock 모드로 동작)

```bash
cd frontend
cp .env.example .env
# .env 파일을 열어 Supabase URL과 Anon Key를 입력
```

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_API_URL=http://localhost:3001
```

### 3. 실행

```bash
# 프론트엔드 (터미널 1)
cd frontend
npm run dev
# → http://localhost:5173

# 백엔드 — M&A 가치평가 계산기 사용 시 필요 (터미널 2)
cd backend
npm run dev
# → http://localhost:3001
```

> **Supabase 없이도 동작합니다.**  
> `.env`가 없거나 기본값이면 자동으로 Mock 모드로 전환되어 샘플 매물 6건으로 전체 UI를 확인할 수 있습니다.

---

## 프로젝트 구조

```
공장중계플랫폼/
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── FactoryCard/      # 매물 목록 카드
│       │   ├── FactoryFilter/    # 필터 사이드바 + 업종 프리셋
│       │   ├── FactoryMap/       # Leaflet 지도
│       │   ├── InquiryModal/     # 상담 신청 모달
│       │   ├── AuthModal/        # 로그인/회원가입 모달
│       │   ├── ReportCard/       # 기술사 진단 보고서
│       │   └── ValuationCalc/    # M&A 가치평가 계산기
│       ├── pages/
│       │   ├── ListingsPage.tsx  # 매물 목록 페이지
│       │   ├── DetailPage.tsx    # 매물 상세 페이지
│       │   └── RegisterPage.tsx  # 매물 등록 페이지
│       ├── hooks/
│       │   ├── useFactoryListings.ts
│       │   └── useValuation.ts
│       ├── lib/
│       │   ├── reportEngine.ts   # 기술사 진단 점수 계산 엔진
│       │   └── supabaseClient.ts
│       ├── data/
│       │   ├── mockListings.ts   # 샘플 매물 6건 (Mock 모드용)
│       │   └── industryPresets.ts
│       └── types/
│           └── factory.ts
├── backend/
│   └── src/
│       ├── routes/valuation.ts
│       └── services/valuationService.ts
└── supabase/
    ├── migrations/001_initial_schema.sql
    └── seed.sql
```

---

## Supabase 연동 (선택)

### DB 마이그레이션

[Supabase 대시보드](https://app.supabase.com) → SQL Editor에서 아래 파일을 순서대로 실행:

```
supabase/migrations/001_initial_schema.sql
supabase/seed.sql
```

### Storage 버킷 생성 (이미지 업로드용)

Supabase 대시보드 → Storage → `factory-images` 버킷 생성 (Public)

### 문의 테이블 추가 (상담 신청 저장용)

```sql
CREATE TABLE factory_inquiries (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  factory_id UUID REFERENCES factory_listings(id) ON DELETE CASCADE,
  name       VARCHAR(50) NOT NULL,
  phone      VARCHAR(20) NOT NULL,
  email      VARCHAR(100),
  message    TEXT
);
```

---

## 기술사 진단 점수 기준

`frontend/src/lib/reportEngine.ts`에 18개 규칙이 정의되어 있습니다.

| 카테고리 | 항목 | 최대 점수 |
|---|---|---|
| 구조 | 층고(처마높이) 4단계, 바닥하중 3단계 | 15~27점 |
| 전기/설비 | 계약전력 3단계, 호이스트 2단계, 콤프레서 | 5~12점 |
| 물류 | 대형차량 진입, 진입로 폭 | 5~8점 |
| 환경 | 폐수처리, 소음등급 | 6~8점 |

점수 구간: **80점↑ 우수 / 60-79 양호 / 40-59 보통 / 40점↓ 주의**
