# TeamBlend 🎯

> AI 기반 팀 매칭 플랫폼 with Three.js 3D 시각화

TeamBlend는 설문 응답을 기반으로 AI가 최적의 팀을 구성하고, **Three.js 3D 공간**에 참가자를 시각화하여 팀 클러스터를 직관적으로 보여주는 혁신적인 팀 빌딩 플랫폼입니다.

---

## ✨ 핵심 기능

### 1️⃣ 3D 시각화 (Three.js + React Three Fiber)
- **t-SNE 알고리즘**으로 생성된 3D 공간에 참가자 배치
- **팀별 컬러 클러스터링**으로 매칭 결과를 한눈에 확인
- **인터랙티브 3D 조작** (회전, 확대/축소, 시점 변경)

### 2️⃣ AI 팀 매칭 (ML 기반)
- **K-means 클러스터링**으로 성향이 유사한 참가자 그룹화
- 설문 응답 데이터를 **고차원 벡터**로 변환하여 분석
- FastAPI + scikit-learn 기반 **Python ML 엔진**

### 3️⃣ 실시간 참여 (QR Code + OAuth)
- **QR 코드 스캔**으로 간편한 모임 입장
- **Google/Kakao OAuth** 소셜 로그인 (호스트용)
- **익명 참가자** 설문 응답 (로그인 불필요)

---

## 🏗️ 아키텍처

TeamBlend는 **Supabase (BaaS) + FastAPI (ML)** 하이브리드 구조로 설계되었습니다:

```
┌─────────────────────────────────────────────────────────┐
│                   Frontend (React 18)                   │
│  • Three.js 3D Visualization                            │
│  • Supabase Client SDK (Auth/DB/Storage)                │
│  • Zustand State Management                             │
└─────────────────┬───────────────────────────────────────┘
                  │
      ┌───────────┴──────────────┐
      │                          │
      ▼                          ▼
┌─────────────────┐    ┌──────────────────────┐
│  Supabase BaaS  │    │  FastAPI ML Service  │
│                 │    │                      │
│  • Auth (OAuth) │    │  • t-SNE 3D          │
│  • PostgreSQL   │    │  • K-means           │
│  • Storage      │    │  • scikit-learn      │
│  • RLS Policies │    │  • JWT Verification  │
└─────────────────┘    └──────────────────────┘
```

**프로젝트 물리적 분리**:
- **Frontend**: `/Users/luca/workspace/React_Project/teamblend` (이 레포지토리)
- **ML Service**: `/Users/luca/workspace/Python_Project/teamblend-ml` (별도 레포지토리)

**핵심 설계 원칙**: 인프라는 Supabase, ML 알고리즘은 FastAPI

---

## 🚀 빠른 시작

### 필수 요구사항

- **Node.js** 18 이상
- **npm** 또는 **yarn**
- **Supabase 계정** (무료 플랜 가능)

### 1. 설치

```bash
# 레포지토리 클론
git clone https://github.com/TeamBlend-Platform/TeamBlend-FE.git
cd teamblend

# 의존성 설치
npm install
```

### 2. 환경 변수 설정

`.env` 파일을 프로젝트 루트에 생성하세요:

```bash
# Supabase 설정
VITE_SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key_here

# FastAPI ML 서버 주소
VITE_ML_API_URL=http://localhost:8000
```

**환경 변수 획득 방법**:
1. [Supabase Dashboard](https://supabase.com/dashboard) → 프로젝트 설정
2. `Settings` → `API` → `Project URL` 및 `anon public` 키 복사

### 3. Supabase 데이터베이스 스키마 생성

Supabase SQL Editor에서 다음 스키마를 실행하세요:

```sql
-- meetings 테이블
CREATE TABLE meetings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at TIMESTAMPTZ DEFAULT now(),
  owner_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  survey_template JSONB NOT NULL,
  status TEXT DEFAULT 'active',
  qr_code_url TEXT,
  max_participants INT DEFAULT 100
);

-- participants 테이블
CREATE TABLE participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at TIMESTAMPTZ DEFAULT now(),
  meeting_id UUID REFERENCES meetings(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  survey_response JSONB NOT NULL,
  team_id INT,
  position_3d JSONB
);

-- RLS 정책 활성화
ALTER TABLE meetings ENABLE ROW LEVEL SECURITY;
ALTER TABLE participants ENABLE ROW LEVEL SECURITY;

-- meetings 정책
CREATE POLICY "Users can create meetings" ON meetings FOR INSERT WITH CHECK (auth.uid() = owner_id);
CREATE POLICY "Users can update own meetings" ON meetings FOR UPDATE USING (auth.uid() = owner_id);
CREATE POLICY "Anyone can view meetings" ON meetings FOR SELECT USING (true);

-- participants 정책
CREATE POLICY "Anyone can join" ON participants FOR INSERT WITH CHECK (true);
CREATE POLICY "Anyone can view participants" ON participants FOR SELECT USING (true);
```

### 4. TypeScript 타입 생성 (Supabase)

```bash
npx supabase gen types typescript --project-id twbakqeemdcaljkymywk > src/types/database.types.ts
```

### 5. 개발 서버 실행

```bash
npm run dev
```

브라우저에서 [http://localhost:5173](http://localhost:5173) 접속

---

## 📚 기술 스택

### Frontend

| 카테고리 | 기술 | 버전 | 역할 |
|---------|------|------|------|
| **UI 프레임워크** | React | 19.2.0 | 컴포넌트 기반 UI |
| **언어** | TypeScript | ~5.9.3 | 타입 안정성 |
| **빌드 도구** | Vite | ^7.2.4 | 개발 서버 + 빌드 |
| **3D 렌더링** | Three.js | ^0.160.0 | WebGL 3D 엔진 ⭐ |
| **React 3D** | React Three Fiber | ^8.15.0 | Three.js React 통합 ⭐ |
| **3D 헬퍼** | @react-three/drei | ^9.92.0 | 카메라, 조명, 로더 ⭐ |
| **스타일링** | TailwindCSS | 3.4.17 | Utility-first CSS |
| **상태 관리** | Zustand | ^5.0.2 | 경량 전역 상태 |
| **BaaS** | Supabase | ^2.49.2 | 인증/DB/스토리지 ⭐ |
| **HTTP 클라이언트** | Axios | ^1.7.9 | ML 서버 통신 |

### Backend (ML Service - 별도 레포지토리)

| 카테고리 | 기술 | 버전 | 역할 |
|---------|------|------|------|
| **Web Framework** | FastAPI | 0.109.0 | REST API |
| **ML 라이브러리** | scikit-learn | 1.4.0 | K-means, t-SNE ⭐ |
| **인증** | Supabase Python SDK | 2.3.0 | JWT 검증 ⭐ |
| **데이터 처리** | pandas, numpy | 2.2.0, 1.26.0 | 데이터 변환 |

---

## 📁 프로젝트 구조

```
teamblend/
├── src/
│   ├── lib/
│   │   └── supabase.ts              # ⭐ Supabase 클라이언트 초기화
│   ├── api/
│   │   └── mlService.ts             # FastAPI ML 서버 호출
│   ├── components/
│   │   ├── visualization/           # ⭐ Three.js 3D 시각화 (핵심)
│   │   │   ├── Cluster3D.tsx
│   │   │   ├── ParticipantMesh.tsx
│   │   │   └── TeamLabel.tsx
│   │   ├── auth/                    # Supabase Auth UI
│   │   ├── meeting/                 # 모임 관리
│   │   └── survey/                  # 설문 시스템
│   ├── hooks/
│   │   ├── useAuth.ts               # ⭐ Supabase Auth Hook
│   │   ├── useMeeting.ts            # ⭐ Supabase DB CRUD Hook
│   │   └── use3DScene.ts            # Three.js Scene Hook
│   ├── store/                       # Zustand
│   │   ├── authStore.ts
│   │   └── meetingStore.ts
│   ├── types/
│   │   ├── database.types.ts        # ⭐ Supabase 자동 생성 타입
│   │   └── matching.ts              # ML API 타입
│   ├── pages/
│   │   ├── Landing.tsx
│   │   ├── Login.tsx
│   │   ├── Dashboard.tsx
│   │   ├── CreateMeeting.tsx
│   │   ├── Survey.tsx
│   │   └── Results.tsx              # 3D 시각화 결과
│   └── utils/
├── docs/                            # 📚 프로젝트 문서
│   ├── TeamBlend_PRD.md
│   ├── 00_QUICK_REFERENCE.md
│   ├── 01_ARCHITECTURE.md
│   ├── 02_FOLDER_STRUCTURE.md
│   ├── 03_CODE_CONVENTIONS.md
│   └── 04_CODE_GENERATION_GUIDE.md
├── .env                             # 환경 변수 (git ignore)
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.js
```

---

## ⚙️ 개발 가이드

### 개발 명령어

```bash
# 개발 서버 실행 (http://localhost:5173)
npm run dev

# 프로덕션 빌드
npm run build

# 빌드 결과 미리보기
npm run preview

# ESLint 실행
npm run lint
```

### ML Service 연동 (선택적)

ML 매칭 기능을 사용하려면 별도 FastAPI 서버가 필요합니다:

```bash
# 별도 터미널에서 실행
cd /Users/luca/workspace/Python_Project/teamblend-ml
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload  # http://localhost:8000
```

**개발 초기 단계**: ML 서비스 없이도 3D 시각화와 UI 개발 가능

### Supabase 타입 업데이트

데이터베이스 스키마 변경 시 타입을 재생성하세요:

```bash
npx supabase gen types typescript --project-id twbakqeemdcaljkymywk > src/types/database.types.ts
```

---

## 📖 문서

프로젝트 상세 문서는 `docs/` 디렉토리에서 확인하세요.

**문서 읽기 순서** (권장):
1. **[빠른 참조](docs/00_QUICK_REFERENCE.md)** - 전체 개요 파악
2. **[아키텍처](docs/01_ARCHITECTURE.md)** - Supabase + FastAPI 상세 설계
3. **[폴더 구조](docs/02_FOLDER_STRUCTURE.md)** - 코드 조직 및 예시
4. **[코드 규칙](docs/03_CODE_CONVENTIONS.md)** - Supabase/FastAPI 코딩 표준
5. **[코드 생성 가이드](docs/04_CODE_GENERATION_GUIDE.md)** - AI 프롬프트 템플릿

---

## 🚢 배포

### Frontend (Vercel)

```bash
# Vercel CLI 설치
npm install -g vercel

# 배포
vercel
```

**환경 변수 설정** (Vercel Dashboard):
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`
- `VITE_ML_API_URL` (프로덕션 ML 서버 주소)

### ML Service (Render - Docker)

별도 레포지토리의 `Dockerfile`을 Render에 배포

### Database (Supabase Cloud)

Supabase는 클라우드 호스팅 (무료 플랜 가능)

---

## 🤝 기여

이 프로젝트는 TeamBlend Platform의 오픈소스 프로젝트입니다.

**브랜치 전략**:
- `main` - 프로덕션 배포 브랜치
- `develop` - 개발 통합 브랜치
- `feature/*` - 기능 개발 브랜치

**커밋 컨벤션**:
```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포매팅
refactor: 코드 리팩토링
test: 테스트 코드
chore: 빌드 설정
```

---

## 📄 라이선스

MIT License

---

<!-- AUTO-VERSION-SECTION: DO NOT EDIT MANUALLY -->
<!-- 이 섹션은 .github/workflows/PROJECT-README-VERSION-UPDATE.yaml에 의해 자동으로 업데이트됩니다 -->
<!-- 수정하지마세요 자동으로 동기화 됩니다 -->
## 최신 버전 : v0.0.1 (2026-01-02)

[전체 버전 기록 보기](CHANGELOG.md)
<!-- END-AUTO-VERSION-SECTION -->
