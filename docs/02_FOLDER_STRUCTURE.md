# TeamBlend 폴더 구조

> **참조**: [TeamBlend PRD](./TeamBlend_PRD.md) | [아키텍처](./01_ARCHITECTURE.md) | [코드 컨벤션](./03_CODE_CONVENTIONS.md)

---

## 프로젝트 개요

TeamBlend는 **두 개의 독립적인 프로젝트**로 구성된 하이브리드 아키텍처입니다:

1. **Frontend (React + Supabase)** - `/Users/luca/workspace/React_Project/teamblend`
2. **ML Service (FastAPI)** - `/Users/luca/workspace/Python_Project/teamblend-ml`

---

## 워크스페이스 전체 구조

```
workspace/
├── React_Project/
│   └── teamblend/                 # Frontend + Supabase 통합
│       ├── src/
│       ├── docs/                  # 프로젝트 문서 (이 파일 포함)
│       ├── .env                   # Supabase 환경변수
│       └── package.json
│
└── Python_Project/
    └── teamblend-ml/              # ML 마이크로서비스 (FastAPI)
        ├── app/
        ├── requirements.txt
        ├── Dockerfile
        └── .env                   # Supabase JWT 검증용
```

---

## 1. Frontend 프로젝트 (teamblend/)

### 전체 구조 (Tree)

```
teamblend/
├── public/
│   └── index.html
│
├── src/
│   ├── lib/                       # 외부 서비스 초기화
│   │   └── supabase.ts            # Supabase 클라이언트
│   │
│   ├── api/                       # API 통신 레이어
│   │   └── mlService.ts           # FastAPI ML 서버 호출 (Axios)
│   │
│   ├── components/                # UI 컴포넌트
│   │   ├── common/                # 공통 컴포넌트
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── ErrorMessage.tsx
│   │   │
│   │   ├── auth/                  # 인증 (Supabase Auth)
│   │   │   ├── SocialLoginButton.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   │
│   │   ├── survey/                # 설문 시스템
│   │   │   ├── SurveyForm.tsx
│   │   │   ├── SliderQuestion.tsx
│   │   │   ├── ChoiceQuestion.tsx
│   │   │   └── MBTISelector.tsx
│   │   │
│   │   ├── visualization/         # 🎯 3D 시각화 (Three.js)
│   │   │   ├── Cluster3D.tsx      # Three.js 메인 컴포넌트
│   │   │   ├── ParticipantMesh.tsx
│   │   │   ├── TeamLabel.tsx
│   │   │   └── CameraControls.tsx
│   │   │
│   │   └── meeting/               # 모임 관리 (Supabase CRUD)
│   │       ├── TemplateCard.tsx
│   │       ├── CodeDisplay.tsx
│   │       └── QRCode.tsx
│   │
│   ├── pages/                     # 라우트 페이지
│   │   ├── Landing.tsx            # 랜딩 페이지
│   │   ├── Login.tsx              # 운영자 로그인 (Supabase OAuth)
│   │   ├── Dashboard.tsx          # 운영자 대시보드
│   │   ├── CreateMeeting.tsx      # 모임 생성 (Supabase INSERT)
│   │   ├── ManageMeeting.tsx      # 모임 관리 (Supabase UPDATE)
│   │   ├── JoinSurvey.tsx         # 참가자 입장
│   │   ├── Survey.tsx             # 설문 진행 (Supabase INSERT)
│   │   └── Results.tsx            # 결과 화면 (3D 시각화)
│   │
│   ├── hooks/                     # Custom Hooks
│   │   ├── useAuth.ts             # Supabase Auth Hook
│   │   ├── useMeeting.ts          # Supabase DB CRUD Hook
│   │   ├── useSupabaseQuery.ts    # Supabase 쿼리 래퍼
│   │   └── use3DScene.ts          # Three.js Scene 관리 Hook
│   │
│   ├── store/                     # Zustand Store
│   │   ├── authStore.ts           # Supabase Auth 상태
│   │   ├── meetingStore.ts        # 모임 상태 (Supabase 동기화)
│   │   └── visualizationStore.ts  # 3D 시각화 상태
│   │
│   ├── types/                     # TypeScript 타입 정의
│   │   ├── database.types.ts      # Supabase 자동 생성 타입
│   │   ├── auth.ts                # 인증 관련 타입
│   │   ├── meeting.ts             # 모임 관련 타입
│   │   ├── survey.ts              # 설문 관련 타입
│   │   ├── matching.ts            # ML 매칭 요청/응답 타입
│   │   ├── error.ts               # 에러 코드 타입
│   │   └── visualization.ts       # 3D 좌표, 팀 데이터 타입
│   │
│   ├── utils/                     # 유틸리티 함수
│   │   ├── errorHandler.ts        # 에러 처리
│   │   ├── qrCodeUpload.ts        # Supabase Storage 업로드
│   │   └── three-helpers.ts       # Three.js 헬퍼 함수
│   │
│   ├── constants/                 # 상수 정의
│   │   ├── errorCodes.ts          # 에러 코드
│   │   ├── routes.ts              # 라우트 경로
│   │   └── colors.ts              # 팀 색상, 테마 색상
│   │
│   ├── App.tsx                    # 앱 루트 컴포넌트
│   ├── main.tsx                   # React 진입점
│   └── index.css                  # 글로벌 스타일
│
├── docs/                          # 프로젝트 문서
│   ├── TeamBlend_PRD.md           # 프로덕트 요구사항 문서
│   ├── 00_QUICK_REFERENCE.md      # 빠른 참조 가이드
│   ├── 01_ARCHITECTURE.md         # 아키텍처 문서
│   ├── 02_FOLDER_STRUCTURE.md     # 이 파일
│   ├── 03_CODE_CONVENTIONS.md     # 코딩 컨벤션
│   └── 04_CODE_GENERATION_GUIDE.md  # AI 코드 생성 가이드
│
├── .env                           # 환경변수 (git ignore)
├── .env.example                   # 환경변수 예시
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── eslint.config.js
```

### 디렉토리별 상세 설명

#### 📁 src/lib/ - 외부 서비스 초기화

**supabase.ts**:
```typescript
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database.types';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey);
```

#### 📁 src/api/ - API 통신 레이어

**mlService.ts** (FastAPI ML 서버 호출):
```typescript
import axios from 'axios';
import { supabase } from '@/lib/supabase';

const mlClient = axios.create({
  baseURL: import.meta.env.VITE_ML_API_URL,
});

// Supabase JWT를 FastAPI로 전달
mlClient.interceptors.request.use(async (config) => {
  const { data: { session } } = await supabase.auth.getSession();
  if (session?.access_token) {
    config.headers.Authorization = `Bearer ${session.access_token}`;
  }
  return config;
});

export const mlApi = {
  runMatching: async (meetingId: string, participants: Participant[]) => {
    const { data } = await mlClient.post('/api/matching', {
      meeting_id: meetingId,
      survey_responses: participants.map(p => p.survey_response),
      num_teams: 5,
    });
    return data;
  },
};
```

#### 📁 src/hooks/ - Custom Hooks

**useAuth.ts** (Supabase Auth Hook):
```typescript
import { useEffect } from 'react';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/store/authStore';

export function useAuth() {
  const { user, setUser, clearUser } = useAuthStore();

  useEffect(() => {
    // 세션 초기화
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
    });

    // 세션 변경 감지
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setUser(session?.user ?? null);
    });

    return () => subscription.unsubscribe();
  }, []);

  return {
    user,
    signInWithGoogle: () => supabase.auth.signInWithOAuth({ provider: 'google' }),
    signInWithKakao: () => supabase.auth.signInWithOAuth({ provider: 'kakao' }),
    signOut: () => supabase.auth.signOut().then(clearUser),
  };
}
```

**useMeeting.ts** (Supabase DB CRUD Hook):
```typescript
import { supabase } from '@/lib/supabase';
import { useMeetingStore } from '@/store/meetingStore';

export function useMeeting() {
  const { setMeeting, setParticipants } = useMeetingStore();

  const createMeeting = async (data: CreateMeetingDto) => {
    const { data: meeting, error } = await supabase
      .from('meetings')
      .insert({
        title: data.title,
        description: data.description,
        survey_template: data.survey_template,
        owner_id: (await supabase.auth.getUser()).data.user?.id,
      })
      .select()
      .single();

    if (error) throw error;
    setMeeting(meeting);
    return meeting;
  };

  const fetchParticipants = async (meetingId: string) => {
    const { data, error } = await supabase
      .from('participants')
      .select('*')
      .eq('meeting_id', meetingId);

    if (error) throw error;
    setParticipants(data);
  };

  return { createMeeting, fetchParticipants };
}
```

#### 📁 src/types/ - TypeScript 타입

**database.types.ts** (Supabase 자동 생성):
```typescript
// supabase gen types typescript --local > src/types/database.types.ts

export type Json = string | number | boolean | null | { [key: string]: Json | undefined } | Json[];

export interface Database {
  public: {
    Tables: {
      meetings: {
        Row: {
          id: string;
          created_at: string;
          owner_id: string;
          title: string;
          description: string | null;
          survey_template: Json;
          status: string;
          qr_code_url: string | null;
          max_participants: number;
        };
        Insert: Omit<Row, 'id' | 'created_at'>;
        Update: Partial<Insert>;
      };
      participants: {
        Row: {
          id: string;
          created_at: string;
          meeting_id: string;
          name: string;
          survey_response: Json;
          team_id: number | null;
          position_3d: Json | null;
        };
        Insert: Omit<Row, 'id' | 'created_at'>;
        Update: Partial<Insert>;
      };
    };
  };
}
```

**matching.ts** (ML API 타입):
```typescript
export interface MatchingRequest {
  meeting_id: string;
  survey_responses: SurveyResponse[];
  num_teams: number;
}

export interface MatchingResult {
  teams: number[];           // [0, 1, 2, 0, 1, ...] 팀 ID 배열
  positions_3d: number[][];  // [[x, y, z], [x, y, z], ...] 3D 좌표
}
```

---

## 2. ML Service 프로젝트 (teamblend-ml/)

### 전체 구조 (Tree)

```
teamblend-ml/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI 앱 진입점
│   ├── config.py                  # 환경 설정
│   │
│   ├── api/                       # API 라우터
│   │   ├── __init__.py
│   │   ├── deps.py                # Supabase JWT 검증 의존성
│   │   └── v1/
│   │       ├── __init__.py
│   │       └── matching.py        # POST /api/matching
│   │
│   ├── core/                      # 핵심 로직
│   │   ├── __init__.py
│   │   ├── supabase_verify.py     # Supabase JWT 검증
│   │   └── errors.py              # 에러 클래스
│   │
│   ├── ml/                        # ML 알고리즘
│   │   ├── __init__.py
│   │   ├── clustering.py          # K-means 클러스터링
│   │   ├── dimensionality.py      # t-SNE 3D 차원 축소
│   │   ├── feature_extraction.py  # 설문 응답 → 벡터 변환
│   │   └── team_builder.py        # 팀 배분 최적화
│   │
│   └── schemas/                   # Pydantic 스키마
│       ├── __init__.py
│       ├── matching.py            # 매칭 요청/응답 스키마
│       └── error.py               # 에러 응답 스키마
│
├── tests/                         # 테스트
│   ├── test_matching.py
│   └── test_ml.py
│
├── .env                           # 환경변수 (git ignore)
├── .env.example                   # 환경변수 예시
├── requirements.txt               # Python 패키지
├── Dockerfile                     # 배포용 Docker 이미지
└── README.md                      # ML 서비스 문서
```

### 디렉토리별 상세 설명

#### 📁 app/core/ - 핵심 로직

**supabase_verify.py** (Supabase JWT 검증):
```python
from fastapi import HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from supabase import create_client, Client
import os

supabase: Client = create_client(
    os.getenv("SUPABASE_URL"),
    os.getenv("SUPABASE_SERVICE_ROLE_KEY")
)

security = HTTPBearer()

async def verify_jwt(credentials: HTTPAuthorizationCredentials = Security(security)):
    """Supabase JWT 토큰 검증"""
    try:
        user = supabase.auth.get_user(credentials.credentials)
        return user
    except Exception as e:
        raise HTTPException(status_code=401, detail="Invalid JWT token")
```

**errors.py** (에러 클래스):
```python
from fastapi import HTTPException

class MLProcessingError(HTTPException):
    def __init__(self, detail: str):
        super().__init__(status_code=500, detail=detail)

class InvalidJWTError(HTTPException):
    def __init__(self):
        super().__init__(status_code=401, detail="Invalid or expired JWT token")
```

#### 📁 app/api/v1/ - API 라우터

**matching.py** (매칭 엔드포인트):
```python
from fastapi import APIRouter, Depends
from app.core.supabase_verify import verify_jwt
from app.ml.clustering import perform_matching
from app.schemas.matching import MatchingRequest, MatchingResponse

router = APIRouter()

@router.post("/api/matching", response_model=MatchingResponse)
async def match_participants(
    request: MatchingRequest,
    user = Depends(verify_jwt)
):
    """
    ML 기반 팀 매칭 수행

    - Supabase JWT 인증 필요
    - t-SNE 3D 차원 축소 + K-means 클러스터링
    - 3D 좌표와 팀 ID 반환
    """
    result = perform_matching(
        responses=request.survey_responses,
        num_teams=request.num_teams
    )

    return MatchingResponse(
        teams=result["teams"],
        positions_3d=result["positions_3d"]
    )
```

#### 📁 app/ml/ - ML 알고리즘

**clustering.py** (K-means + t-SNE):
```python
from sklearn.manifold import TSNE
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import numpy as np

def perform_matching(responses: list[dict], num_teams: int) -> dict:
    """
    설문 응답 → ML 매칭 → 3D 좌표 + 팀 ID

    Args:
        responses: 설문 응답 JSON 리스트
        num_teams: 팀 개수

    Returns:
        {
          "teams": [0, 1, 2, 0, 1, ...],
          "positions_3d": [[x, y, z], [x, y, z], ...]
        }
    """
    # 1. Feature Extraction
    features = extract_features(responses)

    # 2. Feature Scaling
    scaler = StandardScaler()
    features_scaled = scaler.fit_transform(features)

    # 3. t-SNE 3D 차원 축소
    tsne = TSNE(n_components=3, random_state=42, perplexity=min(30, len(features) - 1))
    positions_3d = tsne.fit_transform(features_scaled)

    # 4. K-means 클러스터링
    kmeans = KMeans(n_clusters=num_teams, random_state=42)
    team_ids = kmeans.fit_predict(features_scaled)

    return {
        "teams": team_ids.tolist(),
        "positions_3d": positions_3d.tolist()
    }

def extract_features(responses: list[dict]) -> np.ndarray:
    """설문 응답 → 숫자 벡터 변환"""
    features = []
    for response in responses:
        feature_vector = []

        # MBTI 원핫 인코딩 (16차원)
        mbti_map = {type_: i for i, type_ in enumerate(['INTJ', 'INTP', ...])}
        mbti_vector = [0] * 16
        mbti_vector[mbti_map[response['mbti']]] = 1
        feature_vector.extend(mbti_vector)

        # 슬라이더 질문 (5~10개)
        for key in ['interest', 'energy', 'planning', ...]:
            feature_vector.append(response.get(key, 0))

        features.append(feature_vector)

    return np.array(features)
```

#### 📁 app/schemas/ - Pydantic 스키마

**matching.py**:
```python
from pydantic import BaseModel, Field

class SurveyResponse(BaseModel):
    mbti: str
    interest: int = Field(..., ge=0, le=10)
    energy: int = Field(..., ge=0, le=10)
    planning: int = Field(..., ge=0, le=10)
    # ... 추가 필드

class MatchingRequest(BaseModel):
    meeting_id: str
    survey_responses: list[SurveyResponse]
    num_teams: int = Field(..., ge=2, le=20)

class MatchingResponse(BaseModel):
    teams: list[int]
    positions_3d: list[list[float]]
```

#### 📁 루트 파일

**main.py** (FastAPI 앱):
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1 import matching
from app.config import settings

app = FastAPI(
    title="TeamBlend ML Service",
    description="ML 기반 팀 매칭 서비스 (t-SNE + K-means)",
    version="1.0.0"
)

# CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 라우터 등록
app.include_router(matching.router, tags=["matching"])

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

**requirements.txt**:
```txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
supabase==2.3.0
scikit-learn==1.4.0
pandas==2.2.0
numpy==1.26.0
pydantic==2.5.0
python-dotenv==1.0.0
```

**Dockerfile**:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 시스템 패키지
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Python 패키지
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 소스 코드
COPY ./app ./app

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 3. 환경 설정 파일

### Frontend (.env)

```bash
# Supabase 설정
VITE_SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# FastAPI ML 서버 주소
VITE_ML_API_URL=http://localhost:8000
# Production: https://teamblend-ml.onrender.com
```

### ML Backend (.env)

```bash
# 서버 설정
HOST=0.0.0.0
PORT=8000
ENVIRONMENT=development

# Supabase 설정 (JWT 검증용)
SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# CORS 설정
CORS_ORIGINS=["http://localhost:5173", "https://teamblend.vercel.app"]
```

---

## 4. 개발 워크플로우

### 로컬 개발 시작

**Frontend**:
```bash
cd /Users/luca/workspace/React_Project/teamblend
npm install
npm run dev  # http://localhost:5173
```

**ML Service**:
```bash
cd /Users/luca/workspace/Python_Project/teamblend-ml
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload  # http://localhost:8000
```

### 타입 생성 (Supabase)

```bash
# Frontend에서 실행
npx supabase gen types typescript --local > src/types/database.types.ts
```

---

## 마무리

이 폴더 구조는 Supabase + FastAPI 하이브리드 아키텍처를 반영한 실제 구현 가이드입니다.

**다음 문서**:
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - Supabase/FastAPI 코드 규칙
- [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - 프롬프트 템플릿
