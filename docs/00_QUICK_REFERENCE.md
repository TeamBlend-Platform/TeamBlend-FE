# TeamBlend 빠른 참조 가이드

> 📌 TeamBlend 문서의 진입점입니다. 필요한 정보를 빠르게 찾아보세요.

---

## 1. 문서 목차

| # | 문서 | 목적 | 바로가기 |
|---|------|------|----------|
| 📋 | **TEAMBLEND_PRD** | 제품 요구사항, 마이크로카피, 와이어프레임 | [열기](./TEAMBLEND_PRD.md) |
| 🏗️ | **01_ARCHITECTURE** | 시스템 아키텍처, 데이터 흐름, DB 스키마 | [열기](./01_ARCHITECTURE.md) |
| 📁 | **02_FOLDER_STRUCTURE** | 폴더 구조, 파일 위치, import 규칙 | [열기](./02_FOLDER_STRUCTURE.md) |
| 📝 | **03_CODE_CONVENTIONS** | 코딩 규칙, TypeScript/React 패턴 | [열기](./03_CODE_CONVENTIONS.md) |
| 🤖 | **04_CODE_GENERATION_GUIDE** | AI 코드 생성 프롬프트 템플릿 | [열기](./04_CODE_GENERATION_GUIDE.md) |
| 🎨 | **05_WIDGETS_GUIDE** | UI 컴포넌트, 토스 UX 원칙 | [열기](./05_WIDGETS_GUIDE.md) |

---

## 2. 상황별 가이드

### 🆕 새 기능 개발 시

| 단계 | 참조 문서 | 확인 내용 |
|------|----------|----------|
| 1. 요구사항 확인 | [PRD](./TEAMBLEND_PRD.md) | 기능 정의, 사용자 플로우 |
| 2. 파일 위치 결정 | [02_FOLDER_STRUCTURE](./02_FOLDER_STRUCTURE.md) | 어디에 파일 생성할지 |
| 3. 코드 작성 | [03_CODE_CONVENTIONS](./03_CODE_CONVENTIONS.md) | 코딩 규칙 준수 |
| 4. UI 개발 | [05_WIDGETS_GUIDE](./05_WIDGETS_GUIDE.md) | 컴포넌트 패턴, UX 원칙 |

### 🔌 Supabase 연동 시

| 작업 | 참조 문서 | 섹션 |
|------|----------|------|
| 인증 구현 | [01_ARCHITECTURE](./01_ARCHITECTURE.md) | 4. 인증 아키텍처 |
| DB 쿼리 | [03_CODE_CONVENTIONS](./03_CODE_CONVENTIONS.md) | 6. Supabase 규칙 |
| RLS 정책 | [01_ARCHITECTURE](./01_ARCHITECTURE.md) | 5. 데이터베이스 설계 |

### 🎨 UI 개발 시

| 작업 | 참조 문서 | 섹션 |
|------|----------|------|
| 버튼, 입력 필드 | [05_WIDGETS_GUIDE](./05_WIDGETS_GUIDE.md) | 2~4장 |
| 로딩 상태 | [05_WIDGETS_GUIDE](./05_WIDGETS_GUIDE.md) | 8. 로딩 상태 |
| 마이크로카피 | [PRD](./TEAMBLEND_PRD.md) | 4. 마이크로카피 가이드 |
| 3D 시각화 | [05_WIDGETS_GUIDE](./05_WIDGETS_GUIDE.md) | 10. 3D 시각화 |

### 🤖 AI로 코드 생성 시

| 생성 대상 | 참조 문서 | 프롬프트 위치 |
|----------|----------|--------------|
| 컴포넌트 | [04_CODE_GENERATION_GUIDE](./04_CODE_GENERATION_GUIDE.md) | 3. 컴포넌트 생성 |
| 페이지 | [04_CODE_GENERATION_GUIDE](./04_CODE_GENERATION_GUIDE.md) | 4. 페이지 생성 |
| 훅/스토어 | [04_CODE_GENERATION_GUIDE](./04_CODE_GENERATION_GUIDE.md) | 5. 훅/스토어 생성 |

---

## 3. 핵심 원칙 요약

### 🏛️ 아키텍처 원칙

| 원칙 | 설명 |
|------|------|
| **Supabase First** | 인증/DB/Storage는 Supabase, FastAPI는 ML만 |
| **RLS Trust** | Frontend에서 접근제어 코드 X, RLS가 담당 |
| **Stateless ML** | FastAPI는 상태 저장 X, 입출력만 처리 |

### 📝 코딩 원칙

| 원칙 | 설명 |
|------|------|
| **타입 안전성** | `strict: true`, `any` 금지, Supabase 타입 사용 |
| **선언적 패턴** | React Three Fiber JSX, 명령형 Three.js 금지 |
| **단일 책임** | 한 파일/함수는 하나의 역할만 |

### ✍️ 라이팅 원칙 (토스 UX)

| 원칙 | 예시 |
|------|------|
| **Clear** | ❌ "처리됨" → ✅ "팀 편성이 완료됐어요" |
| **Concise** | ❌ "현재 진행 중인 설문입니다" → ✅ "설문 진행 중" |
| **Casual** | ❌ "입력하십시오" → ✅ "입력해 주세요" |

---

## 4. 자주 쓰는 코드 패턴

### Supabase 클라이언트 Import

```typescript
// ✅ 항상 싱글톤 import
import { supabase } from '@/lib/supabase';
```

### 타입 Import

```typescript
// ✅ type import 사용
import type { Database } from '@/types/database.types';

type Meeting = Database['public']['Tables']['meetings']['Row'];
type MeetingInsert = Database['public']['Tables']['meetings']['Insert'];
```

### 컴포넌트 선언

```typescript
// ✅ function 선언 + default export
interface ButtonProps {
  variant: 'primary' | 'secondary';
  children: React.ReactNode;
}

export default function Button({ variant, children }: ButtonProps) {
  return <button className={...}>{children}</button>;
}
```

### Supabase 쿼리

```typescript
// ✅ 명시적 컬럼 + 에러 처리
const { data, error } = await supabase
  .from('meetings')
  .select('id, name, code, status')
  .eq('status', 'collecting');

if (error) throw new Error(`조회 실패: ${error.message}`);
```

### Zustand 스토어 사용

```typescript
// ✅ 선택자로 구독 최소화
const user = useAuthStore((state) => state.user);

// ❌ 전체 스토어 구독 피하기
const { user } = useAuthStore();
```

### Three.js 애니메이션

```typescript
// ✅ useFrame 사용
import { useFrame } from '@react-three/fiber';

useFrame((state, delta) => {
  meshRef.current.rotation.y += delta * 0.5;
});

// ❌ requestAnimationFrame 사용 금지
```

---

## 5. 폴더 구조 빠른 참조

```
src/
├── api/              # API 클라이언트 (mlService.ts)
├── components/       # UI 컴포넌트
│   ├── common/       # 공통 (Button, Input, Modal)
│   ├── auth/         # 인증 관련
│   ├── survey/       # 설문 관련
│   ├── visualization/# 3D 시각화
│   └── meeting/      # 모임 관리
├── hooks/            # 커스텀 훅 (useAuth, useMeeting)
├── lib/              # 외부 라이브러리 설정 (supabase.ts)
├── pages/            # 페이지 컴포넌트 (~Page.tsx)
├── stores/           # Zustand 스토어 (~Store.ts)
├── types/            # TypeScript 타입
└── utils/            # 유틸리티 함수
```

---

## 6. 네이밍 규칙 빠른 참조

| 종류 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | PascalCase | `MeetingCard.tsx` |
| 페이지 | PascalCase + Page | `DashboardPage.tsx` |
| 훅 | camelCase + use | `useMeeting.ts` |
| 스토어 | camelCase + Store | `authStore.ts` |
| 유틸리티 | camelCase | `formatters.ts` |
| 상수 | SCREAMING_SNAKE | `MAX_PARTICIPANTS` |
| Boolean | is/has/can 접두사 | `isLoading`, `hasError` |

---

## 7. 금지 사항 체크리스트

### ❌ 하지 말 것

- [ ] `any` 타입 사용
- [ ] 상대 경로 import (`../../../`)
- [ ] 컴포넌트 내 Supabase 클라이언트 생성
- [ ] `select('*')` 사용
- [ ] 화살표 함수 컴포넌트 + 별도 export
- [ ] `requestAnimationFrame` 사용
- [ ] 인라인 스타일
- [ ] 거대한 단일 스토어
- [ ] 에러 무시

### ✅ 대신 할 것

- [x] 구체적 타입 정의
- [x] 경로 별칭 (`@/`)
- [x] 싱글톤 import
- [x] 명시적 컬럼 지정
- [x] function 선언 + default export
- [x] `useFrame` 사용
- [x] TailwindCSS 클래스
- [x] 도메인별 스토어 분리
- [x] throw 또는 에러 상태 관리

---

## 8. 개발 체크리스트

### 📋 개발 시작 전

- [ ] PRD에서 기능 요구사항 확인
- [ ] 02_FOLDER_STRUCTURE에서 파일 위치 결정
- [ ] 기존 유사 코드 참고

### 🔨 개발 중

- [ ] 03_CODE_CONVENTIONS 코딩 규칙 준수
- [ ] 05_WIDGETS_GUIDE UI 패턴 적용
- [ ] 타입 안전성 유지 (`any` 금지)
- [ ] 마이크로카피 PRD 참고

### ✅ 개발 완료 후

- [ ] `npm run build` 타입 에러 없음
- [ ] `npm run lint` 경고 없음
- [ ] 변경사항 문서 반영 필요 여부 확인

---

## 9. 기술 스택 요약

| 계층 | 기술 |
|------|------|
| **Frontend** | React 18 + TypeScript + Vite |
| **스타일** | TailwindCSS |
| **상태관리** | Zustand |
| **3D** | Three.js + React Three Fiber |
| **인증/DB** | Supabase (Auth + PostgreSQL + Storage) |
| **ML Backend** | FastAPI + scikit-learn |
| **배포** | Vercel (FE) + Render (ML) |

---

## 10. 유용한 명령어

```bash
# 개발 서버
npm run dev

# 타입 체크 + 빌드
npm run build

# 린트
npm run lint

# Supabase 타입 생성
npx supabase gen types typescript --project-id twbakqeemdcaljkymywk > src/types/database.types.ts
```

---

## 참고 문서

- [TEAMBLEND_PRD.md](./TEAMBLEND_PRD.md) - 제품 요구사항
- [01_ARCHITECTURE.md](./01_ARCHITECTURE.md) - 시스템 아키텍처
- [02_FOLDER_STRUCTURE.md](./02_FOLDER_STRUCTURE.md) - 폴더 구조
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 코드 컨벤션
- [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - AI 코드 생성
- [05_WIDGETS_GUIDE.md](./05_WIDGETS_GUIDE.md) - UI 위젯 가이드
