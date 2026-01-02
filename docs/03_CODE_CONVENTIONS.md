# TeamBlend 코드 작성 규칙

> **최종 수정**: 2026-01-02
> **작성자**: Claude Code
> **관련 문서**: [01_ARCHITECTURE.md](./01_ARCHITECTURE.md), [02_FOLDER_STRUCTURE.md](./02_FOLDER_STRUCTURE.md)

---

## 목차

1. [Supabase 규칙](#supabase-규칙) ⭐ NEW
2. [TypeScript 규칙](#typescript-규칙)
3. [React 컴포넌트 규칙](#react-컴포넌트-규칙)
4. [Three.js 규칙](#threejs-규칙)
5. [스타일링 규칙 (TailwindCSS)](#스타일링-규칙-tailwindcss)
6. [상태 관리 규칙 (Zustand)](#상태-관리-규칙-zustand)
7. [FastAPI ML 서비스 규칙](#fastapi-ml-서비스-규칙) ⭐ NEW
8. [파일 및 폴더 명명 규칙](#파일-및-폴더-명명-규칙)
9. [Import 순서 및 구성](#import-순서-및-구성)
10. [에러 처리 규칙](#에러-처리-규칙)
11. [테스트 규칙](#테스트-규칙)

---

## Supabase 규칙

### 1. 클라이언트 초기화

#### ✅ 단일 인스턴스 사용 (싱글톤 패턴)

```typescript
// ✅ Good: src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database.types';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey);

// ❌ Bad: 컴포넌트마다 새로운 클라이언트 생성
const supabase = createClient(url, key); // 매번 새로 생성 금지
```

### 2. 타입 안전성

#### ✅ Database 타입 사용

```typescript
// ✅ Good: Supabase 자동 생성 타입 사용
import type { Database } from '@/types/database.types';

type Meeting = Database['public']['Tables']['meetings']['Row'];
type MeetingInsert = Database['public']['Tables']['meetings']['Insert'];
type MeetingUpdate = Database['public']['Tables']['meetings']['Update'];

// ❌ Bad: any 또는 수동 타입 정의
type Meeting = any; // 타입 안전성 상실
```

**타입 생성 명령어**:
```bash
npx supabase gen types typescript --project-id twbakqeemdcaljkymywk > src/types/database.types.ts
```

### 3. 쿼리 패턴

#### ✅ 명시적 Select

```typescript
// ✅ Good: 필요한 필드만 선택
const { data, error } = await supabase
  .from('meetings')
  .select('id, title, created_at, owner_id')
  .eq('status', 'active');

// ❌ Bad: 모든 필드 가져오기 (성능 저하)
const { data, error } = await supabase.from('meetings').select('*');
```

#### ✅ 에러 처리 필수

```typescript
// ✅ Good: 에러 검사 후 처리
const { data, error } = await supabase
  .from('meetings')
  .select('*')
  .single();

if (error) {
  throw new Error(`Failed to fetch meeting: ${error.message}`);
}

return data; // 타입: Meeting (not Meeting | null)

// ❌ Bad: 에러 무시
const { data } = await supabase.from('meetings').select('*').single();
return data; // data가 null일 수 있음
```

#### ✅ 조인 관계 처리

```typescript
// ✅ Good: Foreign Key 관계 조인
const { data, error } = await supabase
  .from('participants')
  .select(`
    id,
    name,
    survey_response,
    meeting:meetings (
      id,
      title
    )
  `)
  .eq('meeting_id', meetingId);

// ❌ Bad: 별도 쿼리로 N+1 문제 발생
const participants = await supabase.from('participants').select('*');
for (const p of participants) {
  const meeting = await supabase.from('meetings').select('*').eq('id', p.meeting_id);
}
```

### 4. 인증 패턴

#### ✅ Session 기반 인증

```typescript
// ✅ Good: Session 사용
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/store/authStore';

export function useAuth() {
  const { user, setUser } = useAuthStore();

  useEffect(() => {
    // 초기 세션 로드
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
    });

    // 세션 변경 감지
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => {
        setUser(session?.user ?? null);
      }
    );

    return () => subscription.unsubscribe();
  }, []);

  return { user };
}

// ❌ Bad: localStorage에 직접 JWT 저장
localStorage.setItem('token', jwt); // Supabase가 자동 관리함
```

#### ✅ OAuth 플로우

```typescript
// ✅ Good: signInWithOAuth 사용
const signInWithGoogle = async () => {
  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${window.location.origin}/auth/callback`,
    },
  });

  if (error) throw error;
};

// ❌ Bad: 직접 OAuth 구현
window.location.href = 'https://accounts.google.com/...'; // 직접 구현 금지
```

### 5. Row Level Security (RLS) 정책

#### ✅ 정책 기반 접근 제어

```sql
-- ✅ Good: RLS 정책으로 데이터 접근 제어
CREATE POLICY "Users can create meetings"
ON meetings FOR INSERT
WITH CHECK (auth.uid() = owner_id);

CREATE POLICY "Users can update own meetings"
ON meetings FOR UPDATE
USING (auth.uid() = owner_id);

-- ❌ Bad: 애플리케이션 레벨에서만 체크
-- (DB 직접 접근 시 보안 우회 가능)
```

#### ✅ 프론트엔드에서 RLS 신뢰

```typescript
// ✅ Good: RLS가 보호하므로 간단한 쿼리
const { data, error } = await supabase
  .from('meetings')
  .update({ title: newTitle })
  .eq('id', meetingId); // RLS가 owner_id 자동 검증

// ❌ Bad: 수동으로 owner_id 체크 (불필요)
const user = await supabase.auth.getUser();
if (meeting.owner_id !== user.id) throw new Error('Unauthorized');
```

### 6. Storage 사용

#### ✅ QR 코드 업로드

```typescript
// ✅ Good: Supabase Storage 사용
import { supabase } from '@/lib/supabase';

export async function uploadQRCode(meetingId: string, qrCodeBlob: Blob): Promise<string> {
  const fileName = `qr-codes/${meetingId}.png`;

  const { data, error } = await supabase.storage
    .from('meeting-assets')
    .upload(fileName, qrCodeBlob, {
      cacheControl: '3600',
      upsert: true,
    });

  if (error) throw new Error(`Upload failed: ${error.message}`);

  const { data: { publicUrl } } = supabase.storage
    .from('meeting-assets')
    .getPublicUrl(fileName);

  return publicUrl;
}

// ❌ Bad: Base64 인코딩 후 DB 저장
const base64 = await convertToBase64(qrCodeBlob); // DB 용량 낭비
await supabase.from('meetings').update({ qr_code: base64 });
```

### 7. Realtime 구독 (선택적)

#### ✅ 참가자 실시간 업데이트

```typescript
// ✅ Good: Realtime 구독
import { useEffect } from 'react';
import { supabase } from '@/lib/supabase';

export function useMeetingRealtime(meetingId: string) {
  useEffect(() => {
    const channel = supabase
      .channel(`meeting:${meetingId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'participants',
          filter: `meeting_id=eq.${meetingId}`,
        },
        (payload) => {
          console.log('New participant:', payload.new);
          // Zustand store 업데이트
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [meetingId]);
}

// ❌ Bad: 폴링으로 계속 쿼리
setInterval(async () => {
  const { data } = await supabase.from('participants').select('*');
}, 1000); // 비효율적
```

---

## TypeScript 규칙

### 1. Strict 모드 필수

**tsconfig.json**:
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### 2. Interface vs Type

#### ✅ Interface 사용 (권장)
- React Props (확장 가능성)
- 객체 구조 (클래스/모델)

```tsx
// ✅ Good
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

interface User {
  id: string;
  name: string;
  email: string;
}
```

#### ✅ Type 사용 (특수 케이스)
- Union Types
- Tuple Types
- Utility Types

```tsx
// ✅ Good
type Position3D = [number, number, number];
type AuthProvider = 'google' | 'kakao';
type UserBasic = Pick<User, 'id' | 'name'>;
```

### 3. 타입 임포트

```typescript
// ✅ Good: type 키워드로 타입만 임포트
import type { Database } from '@/types/database.types';
import type { FC } from 'react';

// ✅ Good: 런타임 값과 타입 분리
import { supabase } from '@/lib/supabase';
import type { User } from '@supabase/supabase-js';

// ❌ Bad: 타입과 값 혼재
import { Database, supabase } from '@/lib/supabase';
```

---

## React 컴포넌트 규칙

### 1. 함수 컴포넌트 스타일

#### ✅ Function Declaration (권장)

```tsx
// ✅ Good: 명시적 export default
export default function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}

// ❌ Bad: Arrow function with export default
const Button = ({ label, onClick }: ButtonProps) => {
  return <button onClick={onClick}>{label}</button>;
};
export default Button;
```

**이유**: 함수 선언문이 호이스팅되어 디버깅 시 스택 트레이스에 명확히 표시됨

### 2. Props 구조분해

```tsx
// ✅ Good: 매개변수에서 직접 구조분해
export default function UserCard({ name, email, avatar }: UserCardProps) {
  return (
    <div>
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

// ❌ Bad: 함수 내부에서 구조분해
export default function UserCard(props: UserCardProps) {
  const { name, email, avatar } = props; // 불필요한 단계
  return <div>...</div>;
}
```

### 3. 조건부 렌더링

```tsx
// ✅ Good: Early return으로 간결하게
export default function UserProfile({ user }: { user: User | null }) {
  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// ❌ Bad: 중첩된 삼항 연산자
export default function UserProfile({ user }: { user: User | null }) {
  return user ? (
    <div>
      <h1>{user.name}</h1>
      {user.email ? <p>{user.email}</p> : null}
    </div>
  ) : (
    <div>Loading...</div>
  );
}
```

### 4. Hooks 사용

#### ✅ 커스텀 훅 네이밍

```typescript
// ✅ Good: use- prefix
export function useAuth() {
  const { user, setUser } = useAuthStore();
  // ...
  return { user, signIn, signOut };
}

export function useMeeting(meetingId: string) {
  // ...
}

// ❌ Bad: use- 없음
export function getAuth() { ... } // 일반 함수와 구분 안 됨
```

#### ✅ useEffect 의존성 배열

```typescript
// ✅ Good: 모든 의존성 명시
useEffect(() => {
  fetchData(userId, meetingId);
}, [userId, meetingId]);

// ❌ Bad: 의존성 누락
useEffect(() => {
  fetchData(userId, meetingId);
}, []); // ESLint 경고 발생
```

---

## Three.js 규칙

### 1. React Three Fiber 선언적 사용

#### ✅ JSX로 Scene 구성

```tsx
// ✅ Good: Declarative JSX
import { Canvas } from '@react-three/fiber';
import { OrbitControls } from '@react-three/drei';

export default function Cluster3D({ participants }: { participants: Participant[] }) {
  return (
    <Canvas camera={{ position: [0, 0, 10], fov: 75 }}>
      <ambientLight intensity={0.5} />
      <pointLight position={[10, 10, 10]} />
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

// ❌ Bad: Imperative Three.js
const scene = new THREE.Scene();
const geometry = new THREE.SphereGeometry(0.2, 32, 32);
scene.add(new THREE.Mesh(geometry, material)); // React 방식 아님
```

### 2. useFrame 훅 사용

```tsx
// ✅ Good: useFrame으로 애니메이션
import { useFrame } from '@react-three/fiber';
import { useRef } from 'react';

function ParticipantMesh({ position, team }: Props) {
  const meshRef = useRef<THREE.Mesh>(null);

  useFrame(() => {
    if (meshRef.current) {
      meshRef.current.rotation.y += 0.01;
    }
  });

  return (
    <mesh ref={meshRef} position={position}>
      <sphereGeometry args={[0.2, 32, 32]} />
      <meshStandardMaterial color={TEAM_COLORS[team]} />
    </mesh>
  );
}

// ❌ Bad: requestAnimationFrame 직접 사용
requestAnimationFrame(() => {
  mesh.rotation.y += 0.01;
}); // React Three Fiber 방식 아님
```

---

## 스타일링 규칙 (TailwindCSS)

### 1. Utility-First 원칙

```tsx
// ✅ Good: Tailwind 유틸리티 클래스
<button className="rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600">
  Submit
</button>

// ❌ Bad: Inline style
<button style={{ backgroundColor: 'blue', padding: '8px 16px', borderRadius: '4px' }}>
  Submit
</button>
```

### 2. 조건부 스타일

```tsx
// ✅ Good: clsx 사용
import clsx from 'clsx';

<button
  className={clsx(
    'rounded px-4 py-2',
    variant === 'primary' && 'bg-blue-500 text-white',
    variant === 'secondary' && 'bg-gray-200 text-gray-800',
    disabled && 'opacity-50 cursor-not-allowed'
  )}
>
  {label}
</button>

// ❌ Bad: 문자열 템플릿
<button className={`rounded px-4 py-2 ${variant === 'primary' ? 'bg-blue-500' : 'bg-gray-200'}`}>
```

---

## 상태 관리 규칙 (Zustand)

### 1. Store 분리

```typescript
// ✅ Good: Domain별 Store 분리
// src/store/authStore.ts
import { create } from 'zustand';
import type { User } from '@supabase/supabase-js';

interface AuthState {
  user: User | null;
  setUser: (user: User | null) => void;
  clearUser: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
}));

// src/store/meetingStore.ts
interface MeetingState {
  meeting: Meeting | null;
  participants: Participant[];
  setMeeting: (meeting: Meeting) => void;
  setParticipants: (participants: Participant[]) => void;
}

export const useMeetingStore = create<MeetingState>((set) => ({
  meeting: null,
  participants: [],
  setMeeting: (meeting) => set({ meeting }),
  setParticipants: (participants) => set({ participants }),
}));

// ❌ Bad: 모든 상태를 하나의 Store에
const useStore = create((set) => ({
  user: null,
  meeting: null,
  participants: [],
  // 너무 많은 상태가 한 곳에
}));
```

### 2. Selector 사용

```typescript
// ✅ Good: Selector로 필요한 상태만 구독
const user = useAuthStore((state) => state.user);
const setUser = useAuthStore((state) => state.setUser);

// ❌ Bad: 전체 store 구독 (불필요한 리렌더링)
const { user, setUser, clearUser } = useAuthStore(); // clearUser 안 쓰는데 구독
```

---

## FastAPI ML 서비스 규칙

### 1. Supabase JWT 검증

```python
# ✅ Good: Supabase JWT 검증 의존성
from fastapi import Depends, HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from supabase import create_client, Client
import os

supabase: Client = create_client(
    os.getenv("SUPABASE_URL"),
    os.getenv("SUPABASE_SERVICE_ROLE_KEY")
)

security = HTTPBearer()

async def verify_jwt(credentials: HTTPAuthorizationCredentials = Security(security)):
    try:
        user = supabase.auth.get_user(credentials.credentials)
        return user
    except Exception as e:
        raise HTTPException(status_code=401, detail="Invalid JWT token")

# 엔드포인트에서 사용
@router.post("/api/matching")
async def match_participants(
    request: MatchingRequest,
    user = Depends(verify_jwt)  # JWT 검증 의존성
):
    # user는 인증된 사용자 정보
    result = perform_matching(request.survey_responses, request.num_teams)
    return result

# ❌ Bad: JWT 검증 생략
@router.post("/api/matching")
async def match_participants(request: MatchingRequest):
    # 인증 없이 처리 (보안 취약)
    return perform_matching(...)
```

### 2. Pydantic 스키마

```python
# ✅ Good: Pydantic으로 타입 검증
from pydantic import BaseModel, Field

class SurveyResponse(BaseModel):
    mbti: str
    interest: int = Field(..., ge=0, le=10)
    energy: int = Field(..., ge=0, le=10)
    planning: int = Field(..., ge=0, le=10)

class MatchingRequest(BaseModel):
    meeting_id: str
    survey_responses: list[SurveyResponse]
    num_teams: int = Field(..., ge=2, le=20)

class MatchingResponse(BaseModel):
    teams: list[int]
    positions_3d: list[list[float]]

# ❌ Bad: dict 사용 (타입 안전성 없음)
def match_participants(request: dict):
    responses = request["survey_responses"]  # 타입 불명확
```

### 3. 에러 처리

```python
# ✅ Good: 명확한 에러 클래스
from fastapi import HTTPException

class MLProcessingError(HTTPException):
    def __init__(self, detail: str):
        super().__init__(status_code=500, detail=detail)

class InvalidJWTError(HTTPException):
    def __init__(self):
        super().__init__(status_code=401, detail="Invalid or expired JWT token")

# 사용
if len(responses) < num_teams:
    raise MLProcessingError("Not enough participants for requested team count")

# ❌ Bad: 일반 Exception
raise Exception("Error")  # HTTP 상태 코드 불명확
```

---

## 파일 및 폴더 명명 규칙

### Frontend (React)

```
✅ Good:
src/
├── lib/
│   └── supabase.ts              # camelCase
├── components/
│   ├── auth/
│   │   └── SocialLoginButton.tsx  # PascalCase (컴포넌트)
│   └── common/
│       └── Button.tsx
├── hooks/
│   └── useAuth.ts               # camelCase
├── store/
│   └── authStore.ts             # camelCase
└── types/
    └── database.types.ts        # camelCase

❌ Bad:
src/components/socialLoginButton.tsx  # 컴포넌트는 PascalCase
src/hooks/UseAuth.ts                  # 훅은 camelCase
```

### Backend (FastAPI)

```
✅ Good:
app/
├── api/
│   └── v1/
│       └── matching.py          # snake_case
├── core/
│   └── supabase_verify.py       # snake_case
├── ml/
│   └── clustering.py
└── schemas/
    └── matching.py

❌ Bad:
app/api/matchingAPI.py           # Python은 snake_case
app/ml/Clustering.py             # 클래스 파일도 snake_case
```

---

## Import 순서 및 구성

### Frontend

```typescript
// ✅ Good: 그룹화 및 순서
// 1. React 및 외부 라이브러리
import { useEffect, useState } from 'react';
import { Canvas } from '@react-three/fiber';
import axios from 'axios';

// 2. 타입 임포트
import type { Database } from '@/types/database.types';
import type { User } from '@supabase/supabase-js';

// 3. 내부 모듈 (절대 경로)
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/store/authStore';
import Button from '@/components/common/Button';

// 4. 상대 경로 (같은 디렉토리)
import { ParticipantMesh } from './ParticipantMesh';
import styles from './Cluster3D.module.css';

// ❌ Bad: 순서 없음
import Button from '@/components/common/Button';
import { useEffect } from 'react';
import type { User } from '@supabase/supabase-js';
import { supabase } from '@/lib/supabase';
```

### Backend

```python
# ✅ Good: 그룹화 및 순서
# 1. Python 표준 라이브러리
import os
from typing import List, Dict

# 2. 외부 라이브러리
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel
from sklearn.cluster import KMeans
from supabase import create_client

# 3. 내부 모듈
from app.core.supabase_verify import verify_jwt
from app.ml.clustering import perform_matching
from app.schemas.matching import MatchingRequest

# ❌ Bad: 순서 없음
from app.core.supabase_verify import verify_jwt
import os
from fastapi import APIRouter
```

---

## 에러 처리 규칙

### Frontend

```typescript
// ✅ Good: try-catch with 구체적 처리
import { handleSupabaseError } from '@/utils/errorHandler';

async function createMeeting(data: CreateMeetingDto) {
  try {
    const { data: meeting, error } = await supabase
      .from('meetings')
      .insert(data)
      .select()
      .single();

    if (error) {
      throw new Error(handleSupabaseError(error));
    }

    return meeting;
  } catch (error) {
    if (error instanceof Error) {
      toast.error(error.message);
    }
    throw error;
  }
}

// ❌ Bad: 에러 무시
async function createMeeting(data: CreateMeetingDto) {
  const { data } = await supabase.from('meetings').insert(data);
  return data; // error 체크 안 함
}
```

### Backend

```python
# ✅ Good: 명확한 에러 분류
from fastapi import HTTPException

@router.post("/api/matching")
async def match_participants(request: MatchingRequest, user = Depends(verify_jwt)):
    try:
        if len(request.survey_responses) < request.num_teams:
            raise HTTPException(
                status_code=400,
                detail="Not enough participants for team count"
            )

        result = perform_matching(
            responses=request.survey_responses,
            num_teams=request.num_teams
        )

        return result

    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail="Internal server error")

# ❌ Bad: 일반 Exception
except Exception:
    pass  # 에러 무시
```

---

## 테스트 규칙

### Frontend (Vitest)

```typescript
// ✅ Good: AAA 패턴 (Arrange, Act, Assert)
import { describe, it, expect, beforeEach } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useAuth } from '@/hooks/useAuth';

describe('useAuth', () => {
  beforeEach(() => {
    // 테스트 전 초기화
  });

  it('should return null user initially', () => {
    // Arrange
    const { result } = renderHook(() => useAuth());

    // Assert
    expect(result.current.user).toBeNull();
  });

  it('should sign in with Google', async () => {
    // Arrange
    const { result } = renderHook(() => useAuth());

    // Act
    await act(async () => {
      await result.current.signInWithGoogle();
    });

    // Assert
    expect(result.current.user).not.toBeNull();
  });
});

// ❌ Bad: 명확하지 않은 테스트
it('test', () => {
  const result = useAuth();
  expect(result).toBeTruthy(); // 무엇을 테스트하는지 불명확
});
```

### Backend (pytest)

```python
# ✅ Good: 명확한 테스트 케이스
import pytest
from app.ml.clustering import perform_matching

def test_perform_matching_success():
    # Arrange
    responses = [
        {"mbti": "INTJ", "interest": 5, "energy": 7},
        {"mbti": "ENFP", "interest": 8, "energy": 3},
    ]
    num_teams = 2

    # Act
    result = perform_matching(responses, num_teams)

    # Assert
    assert "teams" in result
    assert "positions_3d" in result
    assert len(result["teams"]) == 2
    assert len(result["positions_3d"]) == 2

def test_perform_matching_insufficient_participants():
    # Arrange
    responses = [{"mbti": "INTJ", "interest": 5, "energy": 7}]
    num_teams = 5  # 참가자보다 많은 팀

    # Act & Assert
    with pytest.raises(ValueError):
        perform_matching(responses, num_teams)

# ❌ Bad: 검증 없는 테스트
def test_matching():
    result = perform_matching([], 2)
    # Assert 없음
```

---

## 주석 및 문서화

### 1. JSDoc/Docstring 사용

```typescript
// ✅ Good: JSDoc으로 함수 설명
/**
 * Supabase에서 모임 정보를 가져옵니다.
 *
 * @param meetingId - 모임 고유 ID
 * @returns Meeting 객체
 * @throws {Error} 모임을 찾을 수 없을 때
 */
export async function fetchMeeting(meetingId: string): Promise<Meeting> {
  const { data, error } = await supabase
    .from('meetings')
    .select('*')
    .eq('id', meetingId)
    .single();

  if (error) throw new Error(`Meeting not found: ${error.message}`);
  return data;
}

// ❌ Bad: 주석 없음
export async function fetchMeeting(meetingId: string) {
  const { data, error } = await supabase.from('meetings').select('*').eq('id', meetingId).single();
  if (error) throw new Error(error.message);
  return data;
}
```

```python
# ✅ Good: Docstring으로 함수 설명
def perform_matching(responses: list[dict], num_teams: int) -> dict:
    """
    설문 응답을 기반으로 ML 팀 매칭을 수행합니다.

    Args:
        responses: 설문 응답 JSON 리스트
        num_teams: 생성할 팀 개수

    Returns:
        {
          "teams": [0, 1, 2, ...],
          "positions_3d": [[x, y, z], ...]
        }

    Raises:
        ValueError: 참가자 수가 팀 개수보다 적을 때
    """
    if len(responses) < num_teams:
        raise ValueError("Insufficient participants")

    # ML 처리
    return {"teams": ..., "positions_3d": ...}

# ❌ Bad: Docstring 없음
def perform_matching(responses, num_teams):
    return {"teams": [], "positions_3d": []}
```

### 2. 복잡한 로직에만 주석

```typescript
// ✅ Good: 복잡한 비즈니스 로직 설명
function calculateMatchScore(response1: Survey, response2: Survey): number {
  // MBTI 유사도: 같은 문자가 많을수록 높은 점수
  const mbtiScore = response1.mbti
    .split('')
    .filter((char, idx) => char === response2.mbti[idx])
    .length * 25;

  // 관심사 거리: 유클리드 거리의 역수
  const interestDistance = Math.sqrt(
    Math.pow(response1.interest - response2.interest, 2) +
    Math.pow(response1.energy - response2.energy, 2)
  );
  const interestScore = 100 / (1 + interestDistance);

  return (mbtiScore + interestScore) / 2;
}

// ❌ Bad: 자명한 코드에 불필요한 주석
// 사용자 이름을 가져옴
const userName = user.name; // 의미 없는 주석
```

---

## 환경 변수 관리

### Frontend

```bash
# ✅ Good: .env 파일
VITE_SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGci...
VITE_ML_API_URL=http://localhost:8000

# .env.example (git에 커밋)
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_anon_key
VITE_ML_API_URL=http://localhost:8000
```

```typescript
// ✅ Good: 환경 변수 검증
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

// ❌ Bad: 하드코딩
const supabaseUrl = 'https://twbakqeemdcaljkymywk.supabase.co'; // git에 노출
```

### Backend

```bash
# ✅ Good: .env 파일
SUPABASE_URL=https://twbakqeemdcaljkymywk.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGci...
CORS_ORIGINS=["http://localhost:5173"]
```

```python
# ✅ Good: python-dotenv로 로드
from dotenv import load_dotenv
import os

load_dotenv()

SUPABASE_URL = os.getenv("SUPABASE_URL")
if not SUPABASE_URL:
    raise ValueError("SUPABASE_URL environment variable is not set")

# ❌ Bad: 하드코딩
SUPABASE_URL = "https://..."  # 보안 취약
```

---

## 마무리

이 규칙을 준수하면:
- ✅ Supabase와 안전하게 통합
- ✅ 타입 안전성 보장
- ✅ 일관된 코드 스타일 유지
- ✅ 유지보수 용이한 코드베이스 구축

**다음 문서**: [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - AI 프롬프트 템플릿
