# TeamBlend AI 코드 생성 가이드

> **최종 수정**: 2026-01-02
> **작성자**: Claude Code
> **대상**: AI 어시스턴트 (Claude, GPT-4, Copilot 등)
> **관련 문서**: [01_ARCHITECTURE.md](./01_ARCHITECTURE.md), [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md)

---

## 목차

1. [개요](#개요)
2. [프로젝트 컨텍스트](#프로젝트-컨텍스트)
3. [Supabase 통합 코드 생성](#supabase-통합-코드-생성) ⭐ NEW
4. [Three.js 컴포넌트 생성](#threejs-컴포넌트-생성)
5. [React 컴포넌트 생성](#react-컴포넌트-생성)
6. [Custom Hook 생성](#custom-hook-생성)
7. [FastAPI ML 서비스 생성](#fastapi-ml-서비스-생성) ⭐ NEW
8. [생성 후 체크리스트](#생성-후-체크리스트)

---

## 개요

이 문서는 AI 어시스턴트가 **Supabase + FastAPI 하이브리드 아키텍처**를 기반으로 TeamBlend 코드를 생성할 때 참조할 **프롬프트 템플릿**을 제공합니다.

### 사용법
1. 원하는 컴포넌트 타입의 템플릿 복사
2. `[placeholder]` 부분을 실제 값으로 교체
3. AI 어시스턴트에게 프롬프트 입력
4. 생성 후 체크리스트로 검증

---

## 프로젝트 컨텍스트

### 하이브리드 아키텍처
```
Frontend (React + Supabase) → Supabase BaaS (Auth/DB/Storage)
                             ↓ (매칭 요청 시)
                          FastAPI ML Service (t-SNE + K-means)
```

### 핵심 기술 스택

**Frontend**:
```
React 18.3.1 + TypeScript 5.6.2 + Vite 6.0.1
Supabase ^2.49.2 (Auth/DB/Storage)
Three.js ^0.160.0 + React Three Fiber ^8.15.0
TailwindCSS 3.4.17 + Zustand ^5.0.2
```

**Backend (ML)**:
```
FastAPI 0.109.0 + scikit-learn 1.4.0
Supabase Python SDK 2.3.0 (JWT 검증)
```

### 프로젝트 구조

**Frontend**: `/Users/luca/workspace/React_Project/teamblend`
```
src/
├── lib/
│   └── supabase.ts              # Supabase 클라이언트
├── api/
│   └── mlService.ts             # FastAPI ML 호출
├── components/
│   ├── visualization/           # Three.js 3D
│   ├── auth/                    # Supabase Auth UI
│   └── common/
├── hooks/
│   ├── useAuth.ts               # Supabase Auth Hook
│   └── useMeeting.ts            # Supabase DB Hook
├── store/
│   ├── authStore.ts
│   └── meetingStore.ts
└── types/
    └── database.types.ts        # Supabase 타입
```

**Backend**: `/Users/luca/workspace/Python_Project/teamblend-ml`
```
app/
├── core/
│   └── supabase_verify.py       # JWT 검증
├── ml/
│   └── clustering.py            # t-SNE + K-means
└── api/v1/
    └── matching.py              # POST /api/matching
```

---

## Supabase 통합 코드 생성

### 1. Supabase Hook 생성

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 [기능명] 기능을 위한 Supabase Hook을 생성해주세요.

**컨텍스트**:
- 프로젝트: React 18 + TypeScript + Supabase
- 위치: src/hooks/use[HookName].ts
- Supabase 클라이언트: '@/lib/supabase'에서 import
- 타입: '@/types/database.types'에서 import
- Store: '@/store/[storeName]'에서 import

**요구사항**:
- Supabase 쿼리 사용 (from, select, insert, update, delete)
- 에러 처리 필수 (error 체크)
- Store 업데이트 포함
- TypeScript strict 모드 준수

**기능**:
[구체적인 기능 설명]

**참고 코드 스타일**:
- Function Declaration 사용
- export function으로 시작
- Supabase 에러는 throw new Error()로 처리
```

**예시**:
```
TeamBlend 프로젝트에서 모임 관리 기능을 위한 Supabase Hook을 생성해주세요.

**컨텍스트**:
- 프로젝트: React 18 + TypeScript + Supabase
- 위치: src/hooks/useMeeting.ts
- Supabase 클라이언트: '@/lib/supabase'에서 import
- 타입: '@/types/database.types'에서 Database import
- Store: '@/store/meetingStore'에서 import

**요구사항**:
- Supabase 쿼리 사용
- 에러 처리 필수
- Store 업데이트 포함
- TypeScript strict 모드 준수

**기능**:
1. createMeeting: 새 모임 생성 (INSERT)
2. fetchMeeting: 모임 조회 (SELECT + single)
3. updateMeeting: 모임 수정 (UPDATE)
4. deleteMeeting: 모임 삭제 (DELETE)
5. fetchParticipants: 참가자 목록 조회 (SELECT + eq)

**참고 코드 스타일**:
- Function Declaration 사용
- Supabase owner_id는 auth.getUser()로 가져오기
- RLS 정책을 신뢰 (수동 권한 체크 불필요)
```

### 2. Supabase Auth Hook 생성

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 Supabase 인증을 위한 useAuth Hook을 생성해주세요.

**컨텍스트**:
- Supabase OAuth 사용 (Google, Kakao)
- Store: useAuthStore (Zustand)
- Session 기반 인증
- useEffect로 세션 초기화 및 변경 감지

**요구사항**:
1. signInWithGoogle: Google OAuth 로그인
2. signInWithKakao: Kakao OAuth 로그인
3. signOut: 로그아웃
4. useEffect:
   - supabase.auth.getSession()으로 초기 세션 로드
   - onAuthStateChange로 세션 변경 감지
   - cleanup: subscription.unsubscribe()

**코드 스타일**:
- import { useEffect } from 'react'
- export function useAuth()
- const { user, setUser } = useAuthStore()
```

### 3. Supabase Storage 업로드 함수 생성

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 QR 코드를 Supabase Storage에 업로드하는 함수를 생성해주세요.

**컨텍스트**:
- 위치: src/utils/qrCodeUpload.ts
- Bucket: 'meeting-assets'
- 경로: qr-codes/{meetingId}.png
- 반환: Public URL (string)

**요구사항**:
1. 매개변수: meetingId (string), qrCodeBlob (Blob)
2. Supabase Storage upload 사용
3. upsert: true 옵션
4. 에러 처리 필수
5. getPublicUrl()로 공개 URL 반환

**코드 스타일**:
- export async function uploadQRCode()
- const { data, error } = await supabase.storage...
- if (error) throw new Error()
```

### 4. ML Service API 클라이언트 생성

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 FastAPI ML 서버를 호출하는 API 클라이언트를 생성해주세요.

**컨텍스트**:
- 위치: src/api/mlService.ts
- Base URL: import.meta.env.VITE_ML_API_URL
- Axios 사용
- Supabase JWT를 Authorization 헤더로 전달

**요구사항**:
1. Axios 인스턴스 생성 (baseURL)
2. Request Interceptor:
   - supabase.auth.getSession()으로 JWT 가져오기
   - Authorization: Bearer {token} 헤더 추가
3. runMatching 함수:
   - POST /api/matching
   - 매개변수: meetingId, participants
   - 반환: { teams, positions_3d }

**코드 스타일**:
- const mlClient = axios.create({ baseURL: ... })
- mlClient.interceptors.request.use(async (config) => { ... })
- export const mlApi = { runMatching: async (...) => { ... } }
```

---

## Three.js 컴포넌트 생성

### 1. Three.js 메인 Scene 컴포넌트

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 3D 클러스터 시각화를 위한 Three.js 컴포넌트를 생성해주세요.

**컨텍스트**:
- 위치: src/components/visualization/Cluster3D.tsx
- React Three Fiber 사용 (Declarative JSX)
- @react-three/drei 헬퍼 사용

**요구사항**:
1. Canvas 래퍼 (카메라: position [0, 0, 10], fov: 75)
2. 조명: ambientLight (intensity 0.5) + pointLight (position [10, 10, 10])
3. OrbitControls (카메라 회전)
4. Stars (배경)
5. Participants 렌더링 (map으로 ParticipantMesh 생성)

**Props**:
- participants: Participant[] (id, position_3d, team_id 포함)

**코드 스타일**:
- import { Canvas } from '@react-three/fiber'
- import { OrbitControls, Stars } from '@react-three/drei'
- export default function Cluster3D({ participants })
- <mesh key={p.id} position={p.position_3d}>
```

### 2. Particle Mesh 컴포넌트

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 참가자를 나타내는 3D Sphere 컴포넌트를 생성해주세요.

**컨텍스트**:
- 위치: src/components/visualization/ParticipantMesh.tsx
- useFrame으로 회전 애니메이션
- useRef로 mesh 참조

**요구사항**:
1. Props: position (Position3D), team (number)
2. useRef<THREE.Mesh>(null)
3. useFrame으로 rotation.y += 0.01
4. sphereGeometry (radius 0.2, segments 32)
5. meshStandardMaterial (color: TEAM_COLORS[team])

**코드 스타일**:
- const meshRef = useRef<THREE.Mesh>(null)
- useFrame(() => { if (meshRef.current) ... })
- <mesh ref={meshRef} position={position}>
```

---

## React 컴포넌트 생성

### 1. 공통 Button 컴포넌트

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 재사용 가능한 Button 컴포넌트를 생성해주세요.

**컨텍스트**:
- 위치: src/components/common/Button.tsx
- TailwindCSS 사용
- variant: 'primary' | 'secondary'

**요구사항**:
1. Props:
   - label: string
   - onClick: () => void
   - variant?: 'primary' | 'secondary' (default: 'primary')
   - disabled?: boolean
2. TailwindCSS 클래스:
   - Primary: bg-blue-500 text-white hover:bg-blue-600
   - Secondary: bg-gray-200 text-gray-800 hover:bg-gray-300
   - Disabled: opacity-50 cursor-not-allowed
3. clsx로 조건부 스타일

**코드 스타일**:
- interface ButtonProps { ... }
- export default function Button({ label, onClick, variant = 'primary', disabled }: ButtonProps)
- <button className={clsx(...)} onClick={onClick} disabled={disabled}>{label}</button>
```

### 2. Supabase Auth UI 컴포넌트

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 소셜 로그인 버튼 컴포넌트를 생성해주세요.

**컨텍스트**:
- 위치: src/components/auth/SocialLoginButton.tsx
- Supabase OAuth 사용
- useAuth Hook 사용

**요구사항**:
1. Props: provider ('google' | 'kakao')
2. useAuth()로 signInWithGoogle, signInWithKakao 가져오기
3. 클릭 시 해당 provider 로그인 함수 호출
4. TailwindCSS로 스타일링
5. 아이콘 포함 (선택적)

**코드 스타일**:
- import { useAuth } from '@/hooks/useAuth'
- interface SocialLoginButtonProps { provider: 'google' | 'kakao' }
- const { signInWithGoogle, signInWithKakao } = useAuth()
- onClick={() => provider === 'google' ? signInWithGoogle() : signInWithKakao()}
```

---

## Custom Hook 생성

### 1. Supabase Realtime Hook

**프롬프트 템플릿**:
```
TeamBlend 프로젝트에서 참가자 실시간 업데이트를 위한 Supabase Realtime Hook을 생성해주세요.

**컨텍스트**:
- 위치: src/hooks/useMeetingRealtime.ts
- Supabase Realtime 구독 사용
- INSERT 이벤트 감지

**요구사항**:
1. 매개변수: meetingId (string)
2. useEffect:
   - supabase.channel() 생성
   - postgres_changes 구독 (event: INSERT, table: participants)
   - filter: meeting_id=eq.{meetingId}
   - payload 수신 시 Store 업데이트
3. Cleanup: supabase.removeChannel(channel)

**코드 스타일**:
- export function useMeetingRealtime(meetingId: string)
- const channel = supabase.channel(`meeting:${meetingId}`)
- .on('postgres_changes', { ... }, (payload) => { ... })
- return () => { supabase.removeChannel(channel) }
```

---

## FastAPI ML 서비스 생성

### 1. JWT 검증 의존성

**프롬프트 템플릿**:
```
TeamBlend ML 서비스에서 Supabase JWT를 검증하는 FastAPI 의존성을 생성해주세요.

**컨텍스트**:
- 위치: app/core/supabase_verify.py
- Supabase Python SDK 사용
- HTTPBearer 인증

**요구사항**:
1. Supabase Client 생성 (SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY)
2. HTTPBearer security
3. verify_jwt 함수:
   - HTTPAuthorizationCredentials 받기
   - supabase.auth.get_user(credentials.credentials) 호출
   - 성공 시 user 반환
   - 실패 시 HTTPException(401)

**코드 스타일**:
- from fastapi import HTTPException, Security
- from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
- supabase: Client = create_client(os.getenv("SUPABASE_URL"), ...)
- async def verify_jwt(credentials: ... = Security(security)):
```

### 2. Matching API 엔드포인트

**프롬프트 템플릿**:
```
TeamBlend ML 서비스에서 팀 매칭 API 엔드포인트를 생성해주세요.

**컨텍스트**:
- 위치: app/api/v1/matching.py
- FastAPI Router 사용
- Supabase JWT 검증 필수

**요구사항**:
1. POST /api/matching
2. Pydantic 스키마:
   - MatchingRequest: meeting_id, survey_responses, num_teams
   - MatchingResponse: teams, positions_3d
3. Depends(verify_jwt)로 JWT 검증
4. perform_matching(responses, num_teams) 호출
5. 에러 처리 (ValueError, Exception)

**코드 스타일**:
- router = APIRouter()
- @router.post("/api/matching", response_model=MatchingResponse)
- async def match_participants(request: MatchingRequest, user = Depends(verify_jwt)):
- try-except ValueError / Exception
```

### 3. ML 클러스터링 로직

**프롬프트 템플릿**:
```
TeamBlend ML 서비스에서 t-SNE + K-means 클러스터링 로직을 생성해주세요.

**컨텍스트**:
- 위치: app/ml/clustering.py
- scikit-learn 사용
- Feature Extraction → t-SNE 3D → K-means

**요구사항**:
1. perform_matching(responses: list[dict], num_teams: int) -> dict
2. 단계:
   a. extract_features(responses) → numpy array
   b. StandardScaler로 정규화
   c. TSNE(n_components=3, random_state=42)
   d. KMeans(n_clusters=num_teams, random_state=42)
3. 반환: { "teams": [...], "positions_3d": [[x,y,z], ...] }

**Feature Extraction**:
- MBTI 원핫 인코딩 (16차원)
- 슬라이더 질문 (interest, energy, planning 등)

**코드 스타일**:
- from sklearn.manifold import TSNE
- from sklearn.cluster import KMeans
- def perform_matching(responses: list[dict], num_teams: int) -> dict:
- Docstring 포함
```

---

## 생성 후 체크리스트

### Frontend (React + Supabase)

#### ✅ Supabase 통합 검증
- [ ] Supabase 클라이언트 import (`from '@/lib/supabase'`)
- [ ] Database 타입 import (`from '@/types/database.types'`)
- [ ] 에러 처리 (`if (error) throw new Error(...)`)
- [ ] Store 업데이트 포함

#### ✅ TypeScript 검증
- [ ] strict 모드 준수 (any 사용 금지)
- [ ] Props 타입 명시 (interface 사용)
- [ ] import type 사용 (타입 전용 import)
- [ ] useEffect 의존성 배열 올바른지

#### ✅ 코드 스타일
- [ ] Function Declaration 사용 (`export default function`)
- [ ] camelCase (파일명, 변수명)
- [ ] PascalCase (컴포넌트명)
- [ ] Import 순서 (React → 타입 → 내부 모듈)

#### ✅ Three.js 검증
- [ ] React Three Fiber JSX 사용 (Imperative 금지)
- [ ] useFrame 훅 사용 (requestAnimationFrame 금지)
- [ ] useRef로 mesh 참조
- [ ] drei 헬퍼 활용 (OrbitControls, Stars)

#### ✅ TailwindCSS 검증
- [ ] Utility-first 클래스 사용
- [ ] clsx로 조건부 스타일
- [ ] Inline style 금지

### Backend (FastAPI ML)

#### ✅ Supabase JWT 검증
- [ ] verify_jwt 의존성 사용
- [ ] Depends(verify_jwt) 적용
- [ ] HTTPException(401) 처리

#### ✅ Pydantic 스키마
- [ ] BaseModel 상속
- [ ] Field 검증 (ge, le 등)
- [ ] 타입 힌트 명시

#### ✅ 코드 스타일
- [ ] snake_case (파일명, 함수명)
- [ ] Docstring 작성
- [ ] Import 순서 (표준 라이브러리 → 외부 → 내부)

#### ✅ ML 로직
- [ ] scikit-learn 버전 호환성
- [ ] random_state 고정 (재현성)
- [ ] 에러 처리 (ValueError, Exception)

### 공통

#### ✅ 에러 처리
- [ ] try-catch 블록
- [ ] 명확한 에러 메시지
- [ ] HTTPException / Error 타입

#### ✅ 환경 변수
- [ ] .env 파일 사용
- [ ] 하드코딩 금지
- [ ] 환경 변수 검증

#### ✅ 주석 및 문서화
- [ ] JSDoc / Docstring 작성
- [ ] 복잡한 로직 설명
- [ ] TODO 주석 금지 (완전한 구현만)

---

## 마무리

이 가이드를 활용하여:
- ✅ Supabase 하이브리드 아키텍처 준수
- ✅ 일관된 코드 스타일 유지
- ✅ 타입 안전성 보장
- ✅ 빠른 프로토타이핑

**추가 참고 문서**:
- [01_ARCHITECTURE.md](./01_ARCHITECTURE.md) - 상세 아키텍처
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 코드 규칙
