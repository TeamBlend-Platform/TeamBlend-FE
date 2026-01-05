# TeamBlend 폴더 구조

---

## 1. 프로젝트 개요

TeamBlend Frontend는 **기능 기반(Feature-based)** 폴더 구조를 사용합니다.

| 원칙 | 설명 |
| --- | --- |
| **기능 단위 그룹핑** | 관련 컴포넌트, 훅, 스토어를 기능 폴더에 모음 |
| **공통 요소 분리** | 재사용 컴포넌트는 `components/common`에 배치 |
| **단일 책임** | 각 파일은 하나의 역할만 담당 |
| **경로 별칭 사용** | `@/` 접두사로 절대 경로 import |

---

## 2. 전체 구조

```
teamblend/
├── public/                      # 정적 파일
│   ├── favicon.ico
│   ├── logo.svg
│   └── manifest.json
│
├── src/
│   ├── api/                     # API 클라이언트
│   │   ├── index.ts             # API 모듈 re-export
│   │   └── mlService.ts         # ML API 클라이언트
│   │
│   ├── components/              # React 컴포넌트
│   │   ├── common/              # 공통 UI 컴포넌트
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Toast.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Spinner.tsx
│   │   │   └── index.ts         # re-export
│   │   │
│   │   ├── auth/                # 인증 관련
│   │   │   ├── SocialLoginButton.tsx
│   │   │   ├── AuthGuard.tsx
│   │   │   └── index.ts
│   │   │
│   │   ├── meeting/             # 모임 관련
│   │   │   ├── MeetingCard.tsx
│   │   │   ├── MeetingList.tsx
│   │   │   ├── MeetingForm.tsx
│   │   │   ├── ParticipantList.tsx
│   │   │   └── index.ts
│   │   │
│   │   ├── survey/              # 설문 관련
│   │   │   ├── SurveyProgress.tsx
│   │   │   ├── QuestionCard.tsx
│   │   │   ├── OptionButton.tsx
│   │   │   └── index.ts
│   │   │
│   │   ├── visualization/       # 3D 시각화
│   │   │   ├── Scene.tsx
│   │   │   ├── ParticipantSphere.tsx
│   │   │   ├── TeamLegend.tsx
│   │   │   ├── Controls.tsx
│   │   │   └── index.ts
│   │   │
│   │   └── layout/              # 레이아웃
│   │       ├── Header.tsx
│   │       ├── Footer.tsx
│   │       ├── Sidebar.tsx
│   │       └── index.ts
│   │
│   ├── hooks/                   # 커스텀 훅
│   │   ├── useAuth.ts           # 인증 훅
│   │   ├── useMeeting.ts        # 모임 CRUD 훅
│   │   ├── useSurvey.ts         # 설문 상태 훅
│   │   ├── use3DScene.ts        # Three.js 씬 훅
│   │   ├── useMatching.ts       # ML 매칭 훅
│   │   └── index.ts
│   │
│   ├── lib/                     # 외부 라이브러리 설정
│   │   ├── supabase.ts          # Supabase 클라이언트 (싱글톤)
│   │   └── axios.ts             # Axios 인스턴스
│   │
│   ├── pages/                   # 페이지 컴포넌트
│   │   ├── LandingPage.tsx
│   │   ├── LoginPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── CreateMeetingPage.tsx
│   │   ├── JoinSurveyPage.tsx
│   │   ├── SurveyPage.tsx
│   │   ├── ResultsPage.tsx
│   │   └── index.ts
│   │
│   ├── stores/                  # Zustand 스토어
│   │   ├── authStore.ts         # 인증 상태
│   │   ├── meetingStore.ts      # 모임 상태
│   │   ├── surveyStore.ts       # 설문 상태
│   │   ├── visualizationStore.ts # 3D 시각화 상태
│   │   └── index.ts
│   │
│   ├── types/                   # TypeScript 타입
│   │   ├── database.types.ts    # Supabase 생성 타입
│   │   ├── meeting.ts           # 모임 관련 타입
│   │   ├── survey.ts            # 설문 관련 타입
│   │   ├── visualization.ts     # 3D 시각화 타입
│   │   └── index.ts
│   │
│   ├── utils/                   # 유틸리티 함수
│   │   ├── errorHandler.ts      # 에러 처리
│   │   ├── formatters.ts        # 포맷팅 함수
│   │   ├── validators.ts        # 유효성 검증
│   │   ├── codeGenerator.ts     # 코드 생성 (TB-XXXX)
│   │   └── index.ts
│   │
│   ├── constants/               # 상수
│   │   ├── routes.ts            # 라우트 경로
│   │   ├── colors.ts            # 팀 색상 등
│   │   ├── templates.ts         # 설문 템플릿
│   │   └── index.ts
│   │
│   ├── styles/                  # 전역 스타일
│   │   ├── globals.css          # 전역 CSS
│   │   └── tailwind.css         # Tailwind 설정
│   │
│   ├── App.tsx                  # 앱 진입점
│   ├── main.tsx                 # ReactDOM 렌더
│   └── vite-env.d.ts            # Vite 타입 선언
│
├── docs/                        # 문서
│   ├── TEAMBLEND_PRD.md
│   ├── 01_ARCHITECTURE.md
│   ├── 02_FOLDER_STRUCTURE.md
│   ├── 03_CODE_CONVENTIONS.md
│   ├── 04_CODE_GENERATION_GUIDE.md
│   └── 05_WIDGETS_GUIDE.md
│
├── .env.example                 # 환경변수 예시
├── .gitignore
├── eslint.config.js
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
└── README.md
```

---

## 3. 디렉토리별 상세 설명

### 3-1. `src/api/`

ML API 클라이언트 모듈.

```typescript
// mlService.ts
import axios from '@/lib/axios';
import { supabase } from '@/lib/supabase';
import type { MatchingRequest, MatchingResponse } from '@/types/meeting';

const ML_API_URL = import.meta.env.VITE_ML_API_URL;

export async function runMatching(request: MatchingRequest): Promise<MatchingResponse> {
  const { data: { session } } = await supabase.auth.getSession();

  if (!session) {
    throw new Error('인증이 필요합니다');
  }

  const response = await axios.post<MatchingResponse>(
    `${ML_API_URL}/api/matching`,
    request,
    { headers: { Authorization: `Bearer ${session.access_token}` } }
  );

  return response.data;
}
```

### 3-2. `src/components/`

컴포넌트는 **기능별**로 그룹화합니다.

#### 파일 명명 규칙

| 종류 | 규칙 | 예시 |
| --- | --- | --- |
| 컴포넌트 | PascalCase | `Button.tsx`, `MeetingCard.tsx` |
| index 파일 | re-export | `index.ts` |

#### 폴더별 역할

| 폴더 | 역할 |
| --- | --- |
| `common/` | 재사용 가능한 기본 UI 컴포넌트 |
| `auth/` | 로그인, 인증 가드 |
| `meeting/` | 모임 생성, 목록, 상세 |
| `survey/` | 설문 진행, 질문 표시 |
| `visualization/` | Three.js 3D 씬 |
| `layout/` | 헤더, 푸터, 사이드바 |

### 3-3. `src/hooks/`

비즈니스 로직을 캡슐화하는 커스텀 훅.

```typescript
// useMeeting.ts
import { supabase } from '@/lib/supabase';
import { useMeetingStore } from '@/stores/meetingStore';
import type { Meeting, MeetingInsert } from '@/types/database.types';

export function useMeeting() {
  const { meetings, setMeetings, addMeeting } = useMeetingStore();

  async function fetchMeetings() {
    const { data, error } = await supabase
      .from('meetings')
      .select('id, name, code, status, created_at')
      .order('created_at', { ascending: false });

    if (error) throw error;
    setMeetings(data);
    return data;
  }

  async function createMeeting(meeting: MeetingInsert) {
    const { data, error } = await supabase
      .from('meetings')
      .insert(meeting)
      .select()
      .single();

    if (error) throw error;
    addMeeting(data);
    return data;
  }

  return { meetings, fetchMeetings, createMeeting };
}
```

### 3-4. `src/lib/`

외부 라이브러리 클라이언트 설정.

```typescript
// supabase.ts - 싱글톤 패턴 필수
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database.types';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

// 싱글톤 - 앱 전체에서 이 인스턴스만 사용
export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey);
```

### 3-5. `src/pages/`

라우트에 대응하는 페이지 컴포넌트.

| 페이지 | 라우트 | 역할 |
| --- | --- | --- |
| `LandingPage` | `/` | 서비스 소개 |
| `LoginPage` | `/login` | 소셜 로그인 |
| `DashboardPage` | `/dashboard` | 모임 목록 |
| `MeetingDetailPage` | `/meeting/:id` | 모임 상세 |
| `SurveyPage` | `/survey/:code` | 설문 진행 |
| `ResultPage` | `/result/:code` | 결과 확인 |

### 3-6. `src/stores/`

Zustand 상태 관리 스토어.

```typescript
// authStore.ts
import { create } from 'zustand';
import type { User, Session } from '@supabase/supabase-js';

interface AuthState {
  user: User | null;
  session: Session | null;
  isLoading: boolean;
  setUser: (user: User | null) => void;
  setSession: (session: Session | null) => void;
  setIsLoading: (isLoading: boolean) => void;
  reset: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  session: null,
  isLoading: true,
  setUser: (user) => set({ user }),
  setSession: (session) => set({ session }),
  setIsLoading: (isLoading) => set({ isLoading }),
  reset: () => set({ user: null, session: null, isLoading: false }),
}));
```

### 3-7. `src/types/`

TypeScript 타입 정의.

```typescript
// database.types.ts - Supabase CLI로 생성
// npx supabase gen types typescript --project-id xxx > src/types/database.types.ts

export type Database = {
  public: {
    Tables: {
      meetings: {
        Row: {
          id: string;
          owner_id: string;
          name: string;
          code: string;
          status: 'collecting' | 'matching' | 'completed';
          // ...
        };
        Insert: {
          // ...
        };
        Update: {
          // ...
        };
      };
      participants: {
        // ...
      };
    };
  };
};

// 헬퍼 타입
export type Meeting = Database['public']['Tables']['meetings']['Row'];
export type MeetingInsert = Database['public']['Tables']['meetings']['Insert'];
export type Participant = Database['public']['Tables']['participants']['Row'];
```

### 3-8. `src/utils/`

유틸리티 함수 모음.

```typescript
// codeGenerator.ts
export function generateMeetingCode(): string {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  let code = 'TB-';
  for (let i = 0; i < 4; i++) {
    code += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return code;
}

export function generateParticipantCode(): string {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  let code = 'P-';
  for (let i = 0; i < 6; i++) {
    code += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return code;
}
```

### 3-9. `src/constants/`

상수값 정의.

```typescript
// routes.ts
export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  DASHBOARD: '/dashboard',
  MEETING: '/meeting/:id',
  SURVEY: '/survey/:code',
  RESULT: '/result/:code',
} as const;

// colors.ts
export const TEAM_COLORS = [
  '#3B82F6', // blue
  '#10B981', // green
  '#F59E0B', // amber
  '#EF4444', // red
  '#8B5CF6', // purple
  '#EC4899', // pink
  '#06B6D4', // cyan
  '#F97316', // orange
] as const;
```

---

## 4. Import 규칙

### 4-1. 경로 별칭 사용

```typescript
// ✅ Good - 경로 별칭 사용
import { Button } from '@/components/common';
import { useAuth } from '@/hooks/useAuth';
import { supabase } from '@/lib/supabase';
import type { Meeting } from '@/types/database.types';

// ❌ Bad - 상대 경로 사용
import { Button } from '../../../components/common/Button';
import { useAuth } from '../../hooks/useAuth';
```

### 4-2. Import 순서

```typescript
// 1. React 및 외부 라이브러리
import { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. 내부 모듈 (components, hooks, stores)
import { Button, Card } from '@/components/common';
import { useAuth } from '@/hooks/useAuth';
import { useMeetingStore } from '@/stores/meetingStore';

// 3. 유틸리티, 상수
import { formatDate } from '@/utils/formatters';
import { ROUTES } from '@/constants/routes';

// 4. 타입 (type import 사용)
import type { Meeting } from '@/types/database.types';
import type { ComponentProps } from 'react';

// 5. 스타일
import './styles.css';
```

### 4-3. Re-export 패턴

```typescript
// components/common/index.ts
export { default as Button } from './Button';
export { default as Input } from './Input';
export { default as Modal } from './Modal';
export { default as Toast } from './Toast';
export { default as Card } from './Card';
export { default as Spinner } from './Spinner';

// 사용
import { Button, Card, Modal } from '@/components/common';
```

---

## 5. 파일 생성 가이드

### 5-1. 새 컴포넌트 추가

1. 기능에 맞는 폴더 선택 (`common/`, `meeting/`, 등)
2. PascalCase로 파일 생성
3. `index.ts`에 re-export 추가

```bash
# 예: 새 모임 관련 컴포넌트
src/components/meeting/
├── MeetingCard.tsx       # 기존
├── MeetingList.tsx       # 기존
├── MeetingFilter.tsx     # 새로 추가
└── index.ts              # export 추가
```

### 5-2. 새 페이지 추가

1. `src/pages/`에 페이지 컴포넌트 생성
2. `App.tsx`에 라우트 추가
3. `constants/routes.ts`에 경로 상수 추가

### 5-3. 새 훅 추가

1. `src/hooks/`에 `use` 접두사로 파일 생성
2. `index.ts`에 re-export 추가
3. 관련 스토어가 있으면 연동

### 5-4. 새 타입 추가

1. Supabase 스키마 변경 시: `npx supabase gen types` 재실행
2. 커스텀 타입: 해당 도메인 파일에 추가 (`meeting.ts`, `survey.ts` 등)

---

## 6. 금지 사항

### ❌ 하지 말 것

| 금지 | 이유 |
| --- | --- |
| `src/` 루트에 컴포넌트 파일 | 폴더 구조 무시 |
| 상대 경로 import (`../../../`) | 가독성 저하 |
| 컴포넌트 내 Supabase 클라이언트 생성 | 싱글톤 위반 |
| `any` 타입 사용 | 타입 안전성 훼손 |
| 한 파일에 여러 export default | 단일 책임 위반 |

### ✅ 해야 할 것

| 규칙 | 설명 |
| --- | --- |
| `@/` 경로 별칭 사용 | 절대 경로로 import |
| 기능별 폴더 분리 | 관련 코드 응집 |
| `index.ts` re-export | 깔끔한 import 경로 |
| 타입 명시 | 모든 함수/변수에 타입 지정 |
| Supabase 타입 사용 | `database.types.ts` 활용 |

---

## 7. Vite 경로 별칭 설정

### vite.config.ts

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

## 참고 문서

- [01_ARCHITECTURE.md](./01_ARCHITECTURE.md) - 시스템 아키텍처
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 코드 컨벤션
- [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - 코드 생성 가이드
