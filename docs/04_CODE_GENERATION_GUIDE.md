# TeamBlend AI 코드 생성 가이드

---

## 1. 개요

이 문서는 AI(Claude, ChatGPT 등)를 활용해 TeamBlend 코드를 생성할 때 사용하는 프롬프트 템플릿과 가이드라인입니다.

### 핵심 원칙

| 원칙 | 설명 |
| --- | --- |
| **컨텍스트 제공** | PRD, 아키텍처 문서 참조 필수 |
| **타입 안전성** | Supabase 생성 타입 사용 명시 |
| **패턴 준수** | 기존 코드 스타일 유지 |
| **증분 생성** | 한 번에 하나의 파일/기능 |

---

## 2. 기본 시스템 프롬프트

모든 코드 생성 요청 전에 다음 시스템 프롬프트를 사용합니다.

```markdown
당신은 TeamBlend 프로젝트의 시니어 프론트엔드 개발자입니다.

## 프로젝트 아키텍처
- Frontend: React 18 + TypeScript + Vite + TailwindCSS
- Backend: Supabase (Auth, PostgreSQL, Storage) + FastAPI (ML only)
- 3D: Three.js + React Three Fiber
- 상태관리: Zustand

## 핵심 규칙
1. TypeScript strict 모드, any 사용 금지
2. Supabase 타입은 database.types.ts에서 import
3. 컴포넌트: function 선언문 + default export
4. Import: @/ 경로 별칭 사용
5. 스타일: TailwindCSS 유틸리티 클래스
6. Supabase 클라이언트: @/lib/supabase에서 싱글톤 import

## 마이크로카피 규칙
- 간결하고 명확하게 (토스 라이팅 원칙)
- 기술 용어 대신 일상 단어 사용
- 에러 메시지에는 해결 방법 포함
```

---

## 3. 컴포넌트 생성 프롬프트

### 3-1. 기본 UI 컴포넌트

```markdown
## 요청
[컴포넌트명] 컴포넌트를 생성해 주세요.

## 스펙
- 위치: src/components/[폴더]/[ComponentName].tsx
- Props: [props 목록]
- 스타일: TailwindCSS

## 예시 코드 (참고용)
```tsx
// 기존 Button.tsx 참고
export default function Button({ variant, children, onClick }: ButtonProps) {
  return (
    <button
      className={`rounded-lg px-4 py-2 ${variant === 'primary' ? 'bg-blue-500' : 'bg-gray-100'}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

## 요구사항
- function 선언문 사용
- Props interface 정의
- TailwindCSS 클래스 사용
- 접근성 고려 (aria 속성)
```

#### 예시: Button 컴포넌트

```markdown
## 요청
Button 컴포넌트를 생성해 주세요.

## 스펙
- 위치: src/components/common/Button.tsx
- Props:
  - variant: 'primary' | 'secondary' | 'ghost'
  - size: 'sm' | 'md' | 'lg'
  - disabled?: boolean
  - loading?: boolean
  - onClick?: () => void
  - children: ReactNode

## 요구사항
- 로딩 시 Spinner 표시
- disabled와 loading 시 클릭 불가
- 호버, 포커스 상태 스타일
```

### 3-2. 설문 관련 컴포넌트

```markdown
## 요청
QuestionCard 컴포넌트를 생성해 주세요.

## 스펙
- 위치: src/components/survey/QuestionCard.tsx
- 역할: 설문 질문과 선택지를 표시

## Props
```typescript
interface QuestionCardProps {
  question: {
    id: string;
    text: string;
    type: 'single' | 'multiple' | 'scale';
    options: Array<{ id: string; label: string }>;
  };
  selectedOption: string | null;
  onSelect: (optionId: string) => void;
}
```

## 마이크로카피
- 선택지 클릭 시 부드러운 전환
- 선택된 항목 시각적 강조

## 접근성
- 키보드 탐색 가능
- role="radiogroup" 사용
```

### 3-3. 3D 시각화 컴포넌트

```markdown
## 요청
ParticipantSphere 컴포넌트를 생성해 주세요.

## 스펙
- 위치: src/components/visualization/ParticipantSphere.tsx
- 역할: 참가자를 3D 구체로 표현

## Props
```typescript
interface ParticipantSphereProps {
  position: [number, number, number];
  color: string;
  name: string;
  isMe: boolean;
  isHighlighted: boolean;
  onClick: () => void;
}
```

## 기술 요구사항
- React Three Fiber 사용
- useRef<THREE.Mesh> 사용
- 호버 시 크기 변화 (1.0 → 1.2)
- isMe가 true면 글로우 효과
- 명령형 Three.js 코드 금지

## 참고 패턴
```tsx
import { useRef, useState } from 'react';
import { useFrame } from '@react-three/fiber';
import type { Mesh } from 'three';

export default function ParticipantSphere({ position, color }: Props) {
  const meshRef = useRef<Mesh>(null);
  const [hovered, setHovered] = useState(false);

  useFrame(() => {
    if (meshRef.current) {
      meshRef.current.scale.lerp(
        { x: hovered ? 1.2 : 1, y: hovered ? 1.2 : 1, z: hovered ? 1.2 : 1 },
        0.1
      );
    }
  });

  return (
    <mesh
      ref={meshRef}
      position={position}
      onPointerOver={() => setHovered(true)}
      onPointerOut={() => setHovered(false)}
    >
      <sphereGeometry args={[0.2, 32, 32]} />
      <meshStandardMaterial color={color} />
    </mesh>
  );
}
```
```

---

## 4. 커스텀 훅 생성 프롬프트

### 4-1. Supabase 데이터 훅

```markdown
## 요청
useMeeting 훅을 생성해 주세요.

## 스펙
- 위치: src/hooks/useMeeting.ts
- 역할: 모임 CRUD 작업 캡슐화

## 반환 타입
```typescript
interface UseMeetingReturn {
  meetings: Meeting[];
  isLoading: boolean;
  error: Error | null;
  fetchMeetings: () => Promise<void>;
  fetchMeeting: (id: string) => Promise<Meeting>;
  createMeeting: (data: MeetingInsert) => Promise<Meeting>;
  updateMeeting: (id: string, data: MeetingUpdate) => Promise<void>;
  deleteMeeting: (id: string) => Promise<void>;
}
```

## 요구사항
1. Supabase 클라이언트: import { supabase } from '@/lib/supabase'
2. 타입: import type { Meeting } from '@/types/database.types'
3. 스토어 연동: useMeetingStore 사용
4. 에러 처리: Supabase error 체크 후 throw
5. 명시적 컬럼 선택 (select('*') 금지)
6. useCallback으로 함수 메모이제이션
```

### 4-2. 인증 훅

```markdown
## 요청
useAuth 훅을 생성해 주세요.

## 스펙
- 위치: src/hooks/useAuth.ts
- 역할: Supabase Auth 상태 관리

## 기능
1. onAuthStateChange 리스너 설정
2. 소셜 로그인 (Google, Kakao)
3. 로그아웃
4. 세션 상태 동기화

## 반환 타입
```typescript
interface UseAuthReturn {
  user: User | null;
  session: Session | null;
  isLoading: boolean;
  signInWithGoogle: () => Promise<void>;
  signInWithKakao: () => Promise<void>;
  signOut: () => Promise<void>;
}
```

## 요구사항
- useEffect에서 onAuthStateChange 구독
- 클린업 함수에서 unsubscribe
- authStore와 동기화
```

### 4-3. ML 매칭 훅

```markdown
## 요청
useMatching 훅을 생성해 주세요.

## 스펙
- 위치: src/hooks/useMatching.ts
- 역할: ML API 호출 및 결과 처리

## 기능
1. ML API 호출 (JWT 포함)
2. 결과를 Supabase에 저장
3. 로딩/에러 상태 관리

## 반환 타입
```typescript
interface UseMatchingReturn {
  isMatching: boolean;
  matchingError: Error | null;
  runMatching: (meetingId: string, numTeams: number) => Promise<MatchingResult>;
}
```

## 데이터 흐름
1. Supabase에서 participants 조회
2. JWT 추출 (supabase.auth.getSession)
3. ML API POST /api/matching
4. 결과로 participants 업데이트 (team_id, position_3d)
```

---

## 5. 페이지 생성 프롬프트

### 5-1. 대시보드 페이지

```markdown
## 요청
DashboardPage를 생성해 주세요.

## 스펙
- 위치: src/pages/DashboardPage.tsx
- 라우트: /dashboard
- 역할: 사용자의 모임 목록 표시

## 구조
```
DashboardPage
├── Header (로고, 사용자 정보, 로그아웃)
├── CreateMeetingButton
├── MeetingList
│   ├── MeetingCard (반복)
│   └── EmptyState (모임 없을 때)
└── Footer
```

## 상태 관리
- 모임 목록: useMeeting 훅
- 사용자 정보: useAuthStore

## 마이크로카피
- 빈 상태: "아직 모임이 없어요. 새 모임을 만들어 보세요!"
- 로딩: "모임을 불러오는 중..."
- 에러: "모임을 불러올 수 없어요. 다시 시도해 주세요."

## 요구사항
- AuthGuard로 인증 필수
- 반응형 그리드 레이아웃
- 모임 카드 클릭 시 상세 페이지 이동
```

### 5-2. 설문 페이지

```markdown
## 요청
SurveyPage를 생성해 주세요.

## 스펙
- 위치: src/pages/SurveyPage.tsx
- 라우트: /survey/:code
- 역할: 참가자 설문 진행

## 구조
```
SurveyPage
├── SurveyProgress (진행률 표시)
├── QuestionCard (현재 질문)
├── NavigationButtons
│   ├── PrevButton
│   └── NextButton / SubmitButton
└── ExitConfirmModal
```

## 상태 관리
- 설문 상태: useSurvey 훅 또는 surveyStore
- 현재 질문 인덱스
- 각 질문별 응답

## 마이크로카피
- 진행률: "3 / 8"
- 다음: "다음 질문"
- 제출: "제출하고 결과 보기"
- 이탈 확인: "지금 나가면 응답이 저장되지 않아요"

## 요구사항
- URL 파라미터(:code)로 모임 조회
- 질문 전환 시 애니메이션
- 제출 전 유효성 검증
- 제출 완료 후 ResultPage로 이동
```

---

## 6. 스토어 생성 프롬프트

```markdown
## 요청
meetingStore를 생성해 주세요.

## 스펙
- 위치: src/stores/meetingStore.ts
- 역할: 모임 상태 관리

## 상태 타입
```typescript
interface MeetingState {
  meetings: Meeting[];
  selectedMeeting: Meeting | null;
  participants: Participant[];

  // Actions
  setMeetings: (meetings: Meeting[]) => void;
  addMeeting: (meeting: Meeting) => void;
  updateMeeting: (id: string, updates: Partial<Meeting>) => void;
  removeMeeting: (id: string) => void;
  setSelectedMeeting: (meeting: Meeting | null) => void;
  setParticipants: (participants: Participant[]) => void;
  updateParticipant: (id: string, updates: Partial<Participant>) => void;
  reset: () => void;
}
```

## 요구사항
1. Zustand create 사용
2. 불변성 유지 (새 배열/객체 생성)
3. 타입: import type from '@/types/database.types'
4. reset 액션으로 초기 상태 복원
```

---

## 7. 유틸리티 함수 프롬프트

```markdown
## 요청
에러 처리 유틸리티를 생성해 주세요.

## 스펙
- 위치: src/utils/errorHandler.ts

## 함수 목록
```typescript
// Supabase 에러를 사용자 메시지로 변환
function getErrorMessage(error: unknown): string;

// 에러 종류 판별
function isNetworkError(error: unknown): boolean;
function isAuthError(error: unknown): boolean;
function isNotFoundError(error: unknown): boolean;

// 에러 로깅
function logError(error: unknown, context?: string): void;
```

## 에러 메시지 매핑
| 에러 코드 | 사용자 메시지 |
| --- | --- |
| PGRST116 | 찾을 수 없어요 |
| 23505 | 이미 존재해요 |
| 401 | 로그인이 필요해요 |
| network | 인터넷 연결을 확인해 주세요 |
| default | 잠시 후 다시 시도해 주세요 |
```

---

## 8. 통합 프롬프트 예시

### 8-1. 전체 기능 구현 요청

```markdown
## 요청
모임 생성 기능 전체를 구현해 주세요.

## 필요 파일
1. src/components/meeting/MeetingForm.tsx
2. src/hooks/useMeeting.ts (createMeeting 추가)
3. src/pages/CreateMeetingPage.tsx

## 데이터 흐름
1. 사용자가 폼 작성 (이름, 템플릿 선택, 팀 수)
2. 제출 시 useMeeting.createMeeting 호출
3. Supabase에 저장
4. 성공 시 모임 상세 페이지로 이동

## 폼 필드
- name: 모임 이름 (필수, 2-50자)
- template_id: 설문 템플릿 (select)
- team_count: 팀 수 (2-20)
- matching_strategy: 매칭 전략 (select)

## 유효성 검증
- 클라이언트 검증 (zod 또는 커스텀)
- 에러 시 해당 필드 하이라이트

## 마이크로카피
- 제목: "새 모임 만들기"
- 이름 힌트: "예: 2025 신입사원 OT"
- 팀 수 힌트: "참가자 수에 맞게 설정해 주세요"
- 제출 버튼: "모임 만들기"
- 성공: "모임을 만들었어요!"
```

### 8-2. 버그 수정 요청

```markdown
## 문제
useAuth 훅에서 로그아웃 후에도 user 상태가 남아있음

## 현재 코드
```typescript
const signOut = useCallback(async () => {
  await supabase.auth.signOut();
}, []);
```

## 예상 원인
signOut 후 authStore.reset()이 호출되지 않음

## 수정 요청
1. signOut 함수에서 authStore.reset() 호출
2. onAuthStateChange에서 SIGNED_OUT 이벤트 처리 확인
3. localStorage에 남은 세션 정리 확인

## 확인 사항
- 로그아웃 후 /dashboard 접근 시 /login으로 리다이렉트
- 네트워크 탭에서 새 토큰 요청 없음
```

---

## 9. 체크리스트

### 생성된 코드 검토 항목

```markdown
## TypeScript
- [ ] any 타입 사용 없음
- [ ] Supabase 타입 사용 (database.types.ts)
- [ ] Props interface 정의됨
- [ ] 함수 반환 타입 명시

## React
- [ ] function 선언문 + default export
- [ ] 적절한 훅 사용 (useCallback, useMemo)
- [ ] useEffect 의존성 배열 정확함
- [ ] 클린업 함수 있음 (필요시)

## Supabase
- [ ] 싱글톤 클라이언트 사용
- [ ] 명시적 컬럼 선택
- [ ] 에러 체크 후 throw
- [ ] RLS 신뢰 (프론트엔드 접근 제어 없음)

## 스타일
- [ ] TailwindCSS 클래스 사용
- [ ] 인라인 스타일 없음
- [ ] 반응형 고려

## 마이크로카피
- [ ] 간결함
- [ ] 기술 용어 없음
- [ ] 에러에 해결책 포함

## 접근성
- [ ] 버튼에 type 속성
- [ ] 이미지에 alt
- [ ] 폼에 label
- [ ] 키보드 탐색 가능
```

---

## 10. 자주 사용하는 코드 스니펫

### 10-1. Supabase 쿼리

```typescript
// SELECT
const { data, error } = await supabase
  .from('meetings')
  .select('id, name, code, status')
  .eq('status', 'collecting')
  .order('created_at', { ascending: false });

// INSERT
const { data, error } = await supabase
  .from('meetings')
  .insert({ name, code: generateCode(), template_id })
  .select()
  .single();

// UPDATE
const { error } = await supabase
  .from('participants')
  .update({ team_id, position_3d })
  .eq('id', participantId);

// DELETE
const { error } = await supabase
  .from('meetings')
  .delete()
  .eq('id', meetingId);
```

### 10-2. 에러 처리 패턴

```typescript
try {
  const { data, error } = await supabase.from('meetings').select('*');
  if (error) throw error;
  return data;
} catch (err) {
  const message = err instanceof Error ? err.message : '알 수 없는 오류';
  toast.error(message);
  throw err;
}
```

### 10-3. 로딩 상태

```typescript
const [isLoading, setIsLoading] = useState(false);

const fetchData = async () => {
  setIsLoading(true);
  try {
    await doSomething();
  } finally {
    setIsLoading(false);
  }
};

if (isLoading) {
  return <Spinner />;
}
```

---

## 참고 문서

- [01_ARCHITECTURE.md](./01_ARCHITECTURE.md) - 시스템 아키텍처
- [02_FOLDER_STRUCTURE.md](./02_FOLDER_STRUCTURE.md) - 폴더 구조
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 코드 컨벤션
- [05_WIDGETS_GUIDE.md](./05_WIDGETS_GUIDE.md) - UI 위젯 가이드
- [TEAMBLEND_PRD.md](./TEAMBLEND_PRD.md) - 제품 요구사항
