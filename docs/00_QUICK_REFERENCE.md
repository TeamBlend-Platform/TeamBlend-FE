# TeamBlend 빠른 참조 가이드

> **최종 수정**: 2026-01-02
> **작성자**: Claude Code
> **목적**: 프로젝트 전체 문서의 요약 및 빠른 참조

---

## 📋 프로젝트 개요

**TeamBlend**는 AI 기반 팀 매칭 플랫폼으로, **Three.js 3D 시각화**를 핵심 기능으로 합니다.

### 핵심 3가지 기능
1. **3D 시각화**: t-SNE 알고리즘으로 생성된 3D 공간에 참가자를 배치하고 팀별 클러스터 시각화
2. **AI 팀 매칭**: K-means 클러스터링 기반 설문 결과 분석 및 최적 팀 편성
3. **실시간 참여**: QR 코드 기반 간편 입장 및 Google/Kakao OAuth 소셜 로그인

### 아키텍처 전략
> "**Supabase (BaaS) + FastAPI (ML)** 하이브리드 구조: 인프라는 Supabase, ML 알고리즘은 FastAPI"

---

## 🚀 빠른 시작

### 프로젝트 구조
TeamBlend는 **두 개의 독립 프로젝트**로 구성됩니다:

1. **Frontend (React + Supabase)** - `/Users/luca/workspace/React_Project/teamblend`
2. **ML Service (FastAPI)** - `/Users/luca/workspace/Python_Project/teamblend-ml`

### Frontend 설치 및 실행

```bash
# 1. 클론 및 설치
cd /Users/luca/workspace/React_Project/teamblend
npm install

# 2. 환경 변수 설정 (.env)
VITE_SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
VITE_ML_API_URL=http://localhost:8000

# 3. 개발 서버 실행
npm run dev  # http://localhost:5173

# 4. 빌드
npm run build
npm run preview
```

### ML Service 설치 및 실행

```bash
# 1. 가상환경 생성
cd /Users/luca/workspace/Python_Project/teamblend-ml
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. 의존성 설치
pip install -r requirements.txt

# 3. 환경 변수 설정 (.env)
SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
CORS_ORIGINS=["http://localhost:5173"]

# 4. 서버 실행
uvicorn app.main:app --reload  # http://localhost:8000
```

### Supabase 스키마 설정

```sql
-- Supabase SQL Editor에서 실행

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

### TypeScript 타입 생성 (Supabase)

```bash
# Frontend 프로젝트에서 실행
npx supabase gen types typescript --project-id twbakqeemdcaljkymywk > src/types/database.types.ts
```

---

## 📁 프로젝트 구조

### 워크스페이스 전체

```
workspace/
├── React_Project/teamblend/          # Frontend (React + Supabase)
└── Python_Project/teamblend-ml/      # ML Service (FastAPI)
```

### Frontend 핵심 구조

```
teamblend/
├── src/
│   ├── lib/
│   │   └── supabase.ts            # ⭐ Supabase 클라이언트 초기화
│   ├── api/
│   │   └── mlService.ts           # FastAPI ML 서버 호출
│   ├── components/
│   │   ├── visualization/         # ⭐ Three.js 3D (핵심)
│   │   │   ├── Cluster3D.tsx
│   │   │   ├── ParticipantMesh.tsx
│   │   │   └── TeamLabel.tsx
│   │   ├── auth/                  # Supabase Auth UI
│   │   ├── meeting/               # 모임 관리
│   │   └── survey/                # 설문 시스템
│   ├── hooks/
│   │   ├── useAuth.ts             # ⭐ Supabase Auth Hook
│   │   ├── useMeeting.ts          # ⭐ Supabase DB CRUD Hook
│   │   └── use3DScene.ts          # Three.js Scene Hook
│   ├── store/                     # Zustand
│   │   ├── authStore.ts
│   │   └── meetingStore.ts
│   ├── types/
│   │   ├── database.types.ts      # ⭐ Supabase 자동 생성 타입
│   │   └── matching.ts            # ML API 타입
│   └── pages/
├── .env                           # ⭐ Supabase 환경변수
└── docs/                          # 📚 프로젝트 문서
```

### ML Service 핵심 구조

```
teamblend-ml/
├── app/
│   ├── main.py                    # FastAPI 진입점
│   ├── core/
│   │   └── supabase_verify.py     # ⭐ Supabase JWT 검증
│   ├── ml/
│   │   ├── clustering.py          # K-means + t-SNE
│   │   └── feature_extraction.py  # 설문 → 벡터 변환
│   └── api/v1/
│       └── matching.py            # POST /api/matching
├── .env                           # ⭐ Supabase JWT 검증용
└── requirements.txt
```

---

## 💻 기술 스택

### Frontend
| 카테고리 | 기술 | 버전 | 역할 |
|---------|------|------|------|
| **UI 프레임워크** | React | 18.3.1 | 컴포넌트 기반 UI |
| **언어** | TypeScript | 5.6.2 | 타입 안정성 |
| **빌드 도구** | Vite | 6.0.1 | 개발 서버 + 빌드 |
| **3D 렌더링** | Three.js | ^0.160.0 | WebGL 3D 엔진 ⭐ |
| **React 3D** | React Three Fiber | ^8.15.0 | Three.js React 통합 ⭐ |
| **3D 헬퍼** | @react-three/drei | ^9.92.0 | 카메라, 조명, 로더 ⭐ |
| **스타일링** | TailwindCSS | 3.4.17 | Utility-first CSS |
| **상태 관리** | Zustand | ^5.0.2 | 경량 전역 상태 |
| **BaaS** | Supabase | ^2.49.2 | 인증/DB/스토리지 ⭐ |
| **HTTP 클라이언트** | Axios | ^1.7.9 | ML 서버 통신 |

### Backend (ML Service)
| 카테고리 | 기술 | 버전 | 역할 |
|---------|------|------|------|
| **Web Framework** | FastAPI | 0.109.0 | REST API |
| **ML 라이브러리** | scikit-learn | 1.4.0 | K-means, t-SNE ⭐ |
| **인증** | Supabase Python SDK | 2.3.0 | JWT 검증 ⭐ |
| **데이터 처리** | pandas, numpy | 2.2.0, 1.26.0 | 데이터 변환 |

---

## 📐 아키텍처 핵심

### 하이브리드 BaaS + ML 구조

```
[사용자 브라우저]
     ↓ HTTPS
[React 18 Frontend]
     ↓ Supabase SDK
[Supabase BaaS] ← Auth/DB/Storage
     ↓ Axios (매칭 요청 시에만)
[FastAPI ML Service] ← JWT 검증 + ML 처리
     ↓ scikit-learn (t-SNE + K-means)
[3D 좌표 + 팀 ID]
     ↓ Zustand Store
[Three.js 3D Scene]
     ↓ WebGL
[Canvas Rendering]
```

### 데이터 흐름 (인증)

```
User → React (소셜 로그인 클릭)
→ Supabase Auth (OAuth)
→ Google/Kakao (인증)
→ Supabase (JWT 발급)
→ React (Session 저장)
→ Zustand (User State)
```

### 데이터 흐름 (매칭)

```
Host → React ("매칭 시작" 클릭)
→ Supabase (SELECT participants)
→ React → FastAPI ML Service (POST /api/matching + JWT)
→ FastAPI (JWT 검증)
→ ML Engine (t-SNE + K-means)
→ React (3D 좌표 + 팀 ID)
→ Supabase (UPDATE participants)
→ Zustand Store
→ Three.js (3D 렌더링)
```

---

## 🎨 핵심 코드 패턴

### Supabase 클라이언트 초기화

```typescript
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database.types';

export const supabase = createClient<Database>(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);
```

### Supabase Auth Hook

```typescript
// src/hooks/useAuth.ts
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/store/authStore';

export function useAuth() {
  const { user, setUser } = useAuthStore();

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_, session) => {
      setUser(session?.user ?? null);
    });

    return () => subscription.unsubscribe();
  }, []);

  return {
    user,
    signInWithGoogle: () => supabase.auth.signInWithOAuth({ provider: 'google' }),
    signOut: () => supabase.auth.signOut(),
  };
}
```

### Supabase DB CRUD

```typescript
// src/hooks/useMeeting.ts
import { supabase } from '@/lib/supabase';

export function useMeeting() {
  const createMeeting = async (data: CreateMeetingDto) => {
    const { data: meeting, error } = await supabase
      .from('meetings')
      .insert({
        title: data.title,
        survey_template: data.survey_template,
        owner_id: (await supabase.auth.getUser()).data.user?.id,
      })
      .select()
      .single();

    if (error) throw error;
    return meeting;
  };

  return { createMeeting };
}
```

### FastAPI ML 호출

```typescript
// src/api/mlService.ts
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

### Three.js 3D 시각화

```tsx
// src/components/visualization/Cluster3D.tsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Stars } from '@react-three/drei';

export default function Cluster3D({ participants }) {
  return (
    <Canvas camera={{ position: [0, 0, 10], fov: 75 }}>
      <ambientLight intensity={0.5} />
      <pointLight position={[10, 10, 10]} />
      <Stars />
      <OrbitControls />

      {participants.map((p) => (
        <mesh key={p.id} position={p.position_3d}>
          <sphereGeometry args={[0.2, 32, 32]} />
          <meshStandardMaterial color={TEAM_COLORS[p.team_id]} />
        </mesh>
      ))}
    </Canvas>
  );
}
```

---

## ✍️ 코드 작성 규칙 TOP 5

### 1. Supabase SDK 우선 사용
```typescript
// ✅ Good: Supabase SDK 사용
const { data, error } = await supabase.from('meetings').select('*');

// ❌ Bad: 직접 API 호출
const response = await axios.get('/api/meetings');
```

### 2. TypeScript Strict 모드 필수
```typescript
// ✅ Good: 명시적 타입
import type { Database } from '@/types/database.types';
type Meeting = Database['public']['Tables']['meetings']['Row'];

// ❌ Bad: any 사용
const meeting: any = { ... };
```

### 3. React Three Fiber 선언적 사용
```tsx
// ✅ Good: Declarative JSX
<mesh position={[0, 0, 0]}>
  <sphereGeometry args={[1, 32, 32]} />
  <meshStandardMaterial color="blue" />
</mesh>

// ❌ Bad: Imperative Three.js
const geometry = new THREE.SphereGeometry(1, 32, 32);
scene.add(new THREE.Mesh(geometry, material));
```

### 4. Zustand 상태 관리
```typescript
// ✅ Good: Domain별 Store 분리
export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));

export const useMeetingStore = create<MeetingState>((set) => ({
  meeting: null,
  setMeeting: (meeting) => set({ meeting }),
}));
```

### 5. Path Alias (@/) 사용
```typescript
// ✅ Good
import { supabase } from '@/lib/supabase';
import Button from '@/components/common/Button';

// ❌ Bad
import { supabase } from '../../../lib/supabase';
```

---

## 🔐 보안 체크리스트

- [ ] `.env` 파일을 `.gitignore`에 추가 ✅
- [ ] Supabase RLS (Row Level Security) 정책 활성화 ✅
- [ ] FastAPI에서 Supabase JWT 검증 구현 ✅
- [ ] CORS 설정 (프로덕션 도메인만 허용) ✅
- [ ] 환경 변수는 플랫폼 시크릿 사용 (Vercel, Render) ✅

---

## 📚 문서 가이드

### 문서 읽기 순서
1. **00_QUICK_REFERENCE.md** (이 문서) - 전체 개요 파악
2. **TeamBlend_PRD.md** - 제품 요구사항 및 기능 명세
3. **01_ARCHITECTURE.md** - 아키텍처 상세 (Supabase + FastAPI)
4. **02_FOLDER_STRUCTURE.md** - 폴더 구조 및 코드 예시
5. **03_CODE_CONVENTIONS.md** - 코딩 규칙
6. **04_CODE_GENERATION_GUIDE.md** - AI 코드 생성 프롬프트

### 문서 업데이트 규칙
- 모든 문서는 Supabase + FastAPI 하이브리드 아키텍처 기준
- 코드 예시는 실제 구현 가능한 수준으로 작성
- 환경 변수는 실제 Supabase URL 사용 (키는 예시)

---

## 🚀 배포 가이드

### Frontend (Vercel)

1. **Vercel 프로젝트 생성**
   - GitHub 연동
   - Root Directory: `teamblend`
   - Build Command: `npm run build`
   - Output Directory: `dist`

2. **환경 변수 설정**
   ```
   VITE_SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
   VITE_SUPABASE_ANON_KEY=eyJhbGci...
   VITE_ML_API_URL=https://teamblend-ml.onrender.com
   ```

### ML Service (Render)

1. **Render Web Service 생성**
   - Docker 배포 선택
   - Dockerfile 경로 지정

2. **환경 변수 설정**
   ```
   SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=eyJhbGci...
   CORS_ORIGINS=["https://teamblend.vercel.app"]
   ```

---

## 🎯 다음 단계

1. **Supabase 스키마 생성** (위의 SQL 실행)
2. **Frontend 개발 서버 실행** (`npm run dev`)
3. **ML Service 개발 서버 실행** (`uvicorn app.main:app --reload`)
4. **Supabase Auth 테스트** (Google/Kakao 로그인)
5. **ML 매칭 테스트** (설문 → 매칭 → 3D 시각화)

**Happy Coding!** 🚀
