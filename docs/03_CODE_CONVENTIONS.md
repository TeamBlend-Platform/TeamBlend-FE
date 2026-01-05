# TeamBlend 코드 컨벤션

---

## 1. 핵심 원칙

| 원칙 | 설명 |
| --- | --- |
| **타입 안전성** | `strict: true`, `any` 사용 금지 |
| **Supabase First** | 인증/DB는 Supabase, 직접 구현 X |
| **선언적 패턴** | 명령형 코드보다 선언적 패턴 선호 |
| **단일 책임** | 한 파일/함수는 하나의 역할만 |

---

## 2. TypeScript 규칙

### 2-1. 타입 시스템

#### Supabase 타입 사용

```typescript
// ✅ Good - Supabase 생성 타입 사용
import type { Database } from '@/types/database.types';

type Meeting = Database['public']['Tables']['meetings']['Row'];
type MeetingInsert = Database['public']['Tables']['meetings']['Insert'];
type MeetingUpdate = Database['public']['Tables']['meetings']['Update'];

// ❌ Bad - 타입 직접 정의
interface Meeting {
  id: string;
  name: string;
  // ...수동으로 모든 필드 정의
}
```

#### any 사용 금지

```typescript
// ✅ Good
function handleResponse(data: MatchingResponse): void {
  // 타입이 명확함
}

// ❌ Bad
function handleResponse(data: any): void {
  // 타입 정보 손실
}
```

#### unknown 사용

```typescript
// ✅ Good - unknown 후 타입 가드
function handleError(error: unknown): string {
  if (error instanceof Error) {
    return error.message;
  }
  return String(error);
}

// ❌ Bad - any 사용
function handleError(error: any): string {
  return error.message; // 런타임 에러 가능
}
```

### 2-2. 타입 vs 인터페이스

```typescript
// interface - 컴포넌트 Props, 확장 가능한 객체
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  onClick?: () => void;
  children: React.ReactNode;
}

// type - Union, Tuple, 유틸리티 타입
type Status = 'collecting' | 'matching' | 'completed';
type Position3D = [number, number, number];
type MeetingWithParticipants = Meeting & { participants: Participant[] };
```

### 2-3. 타입 Import

```typescript
// ✅ Good - type import 사용
import type { Meeting, Participant } from '@/types/database.types';
import type { ComponentProps, ReactNode } from 'react';

// ❌ Bad - 일반 import로 타입 가져오기
import { Meeting } from '@/types/database.types';
```

### 2-4. 제네릭 활용

```typescript
// ✅ Good - 제네릭으로 재사용 (실제 사용 시 명시적 컬럼 지정 필요)
async function fetchMeetingsByStatus(
  status: string
): Promise<Meeting[]> {
  const { data, error } = await supabase
    .from('meetings')
    .select('id, name, code, status, created_at')
    .eq('status', status);

  if (error) throw error;
  return data;
}

// 사용
const meetings = await fetchMeetingsByStatus('collecting');
```

---

## 3. React 컴포넌트 규칙

### 3-1. 함수 선언 스타일

```typescript
// ✅ Good - function 선언문 + default export
export default function MeetingCard({ meeting, onSelect }: MeetingCardProps) {
  return (
    <div className="rounded-lg border p-4">
      <h3>{meeting.name}</h3>
      <Button onClick={() => onSelect(meeting.id)}>선택</Button>
    </div>
  );
}

// ❌ Bad - 화살표 함수 + 별도 export
const MeetingCard = ({ meeting, onSelect }: MeetingCardProps) => {
  return <div>...</div>;
};

export default MeetingCard;
```

### 3-2. Props 인터페이스

```typescript
// ✅ Good - 명확한 Props 정의
interface QuestionCardProps {
  question: Question;
  selectedOption: string | null;
  onSelect: (optionId: string) => void;
  disabled?: boolean;
}

export default function QuestionCard({
  question,
  selectedOption,
  onSelect,
  disabled = false,
}: QuestionCardProps) {
  // ...
}

// ❌ Bad - 인라인 타입, 구조분해 없음
export default function QuestionCard(props: {
  question: any;
  onSelect: Function;
}) {
  const q = props.question; // 구조분해 미사용
  // ...
}
```

### 3-3. 컴포넌트 구조

```typescript
// 1. Imports
import { useState, useCallback } from 'react';
import { Button, Card } from '@/components/common';
import { useMeeting } from '@/hooks/useMeeting';
import type { Meeting } from '@/types/database.types';

// 2. Props Interface
interface MeetingListProps {
  status?: Meeting['status'];
  onSelect: (id: string) => void;
}

// 3. Component
export default function MeetingList({ status, onSelect }: MeetingListProps) {
  // 3a. Hooks (순서: state → custom hooks → callbacks → effects)
  const [isLoading, setIsLoading] = useState(false);
  const { meetings, fetchMeetings } = useMeeting();

  const handleSelect = useCallback((id: string) => {
    onSelect(id);
  }, [onSelect]);

  // 3b. Early returns
  if (isLoading) {
    return <Spinner />;
  }

  if (meetings.length === 0) {
    return <EmptyState message="모임이 없습니다" />;
  }

  // 3c. Main render
  return (
    <div className="grid gap-4">
      {meetings.map((meeting) => (
        <MeetingCard
          key={meeting.id}
          meeting={meeting}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
}
```

### 3-4. 조건부 렌더링

```typescript
// ✅ Good - 논리 연산자, 삼항 연산자
function StatusBadge({ status }: { status: Status }) {
  return (
    <span className={status === 'completed' ? 'text-green-500' : 'text-gray-500'}>
      {status === 'collecting' && '응답 수집 중'}
      {status === 'matching' && '매칭 진행 중'}
      {status === 'completed' && '완료'}
    </span>
  );
}

// ❌ Bad - if/else 체인
function StatusBadge({ status }: { status: Status }) {
  let text = '';
  let color = '';

  if (status === 'collecting') {
    text = '응답 수집 중';
    color = 'text-gray-500';
  } else if (status === 'matching') {
    text = '매칭 진행 중';
    color = 'text-gray-500';
  } else {
    text = '완료';
    color = 'text-green-500';
  }

  return <span className={color}>{text}</span>;
}
```

### 3-5. 이벤트 핸들러 네이밍

```typescript
// ✅ Good - handle 접두사
function SurveyForm() {
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // ...
  };

  const handleOptionClick = (optionId: string) => {
    // ...
  };

  return (
    <form onSubmit={handleSubmit}>
      {options.map((opt) => (
        <button key={opt.id} onClick={() => handleOptionClick(opt.id)}>
          {opt.label}
        </button>
      ))}
    </form>
  );
}

// ❌ Bad - 불분명한 네이밍
function SurveyForm() {
  const submit = () => {}; // 동사만
  const click = () => {}; // 너무 일반적
  const doStuff = () => {}; // 의미 없음
}
```

---

## 4. Hooks 규칙

### 4-1. 커스텀 훅 패턴

```typescript
// ✅ Good - 명확한 반환 타입
interface UseMeetingReturn {
  meetings: Meeting[];
  isLoading: boolean;
  error: Error | null;
  fetchMeetings: () => Promise<void>;
  createMeeting: (data: MeetingInsert) => Promise<Meeting>;
}

export function useMeeting(): UseMeetingReturn {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const { meetings, setMeetings, addMeeting } = useMeetingStore();

  const fetchMeetings = useCallback(async () => {
    setIsLoading(true);
    setError(null);

    try {
      const { data, error } = await supabase
        .from('meetings')
        .select('id, name, code, status, created_at')
        .order('created_at', { ascending: false });

      if (error) throw error;
      setMeetings(data);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setIsLoading(false);
    }
  }, [setMeetings]);

  const createMeeting = useCallback(async (data: MeetingInsert) => {
    const { data: meeting, error } = await supabase
      .from('meetings')
      .insert(data)
      .select()
      .single();

    if (error) throw error;
    addMeeting(meeting);
    return meeting;
  }, [addMeeting]);

  return { meetings, isLoading, error, fetchMeetings, createMeeting };
}
```

### 4-2. useEffect 규칙

```typescript
// ✅ Good - 의존성 명시, 클린업 함수
useEffect(() => {
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    (event, session) => {
      if (event === 'SIGNED_IN') {
        setUser(session?.user ?? null);
      } else if (event === 'SIGNED_OUT') {
        setUser(null);
      }
    }
  );

  // 클린업
  return () => {
    subscription.unsubscribe();
  };
}, []); // 빈 배열 = 마운트/언마운트 시에만

// ❌ Bad - 의존성 누락, 클린업 없음
useEffect(() => {
  supabase.auth.onAuthStateChange((event, session) => {
    setUser(session?.user);
  });
  // 클린업 없음 → 메모리 누수
}); // 의존성 배열 없음 → 매 렌더마다 실행
```

### 4-3. useCallback, useMemo

```typescript
// ✅ Good - 콜백이 props로 전달될 때
const handleSelect = useCallback((id: string) => {
  onSelect(id);
  trackEvent('meeting_selected', { id });
}, [onSelect]);

// ✅ Good - 비싼 계산
const sortedParticipants = useMemo(() => {
  return [...participants].sort((a, b) => a.team_id - b.team_id);
}, [participants]);

// ❌ Bad - 불필요한 메모이제이션
const title = useMemo(() => `${name}의 모임`, [name]); // 문자열 연결은 비용 낮음
const handleClick = useCallback(() => {
  console.log('clicked');
}, []); // 내부에서만 사용되면 불필요
```

---

## 5. Zustand 스토어 규칙

### 5-1. 스토어 분리

```typescript
// ✅ Good - 도메인별 분리
// authStore.ts
export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  session: null,
  setUser: (user) => set({ user }),
  setSession: (session) => set({ session }),
}));

// meetingStore.ts
export const useMeetingStore = create<MeetingState>((set) => ({
  meetings: [],
  selectedMeeting: null,
  setMeetings: (meetings) => set({ meetings }),
  addMeeting: (meeting) => set((state) => ({
    meetings: [...state.meetings, meeting],
  })),
}));

// ❌ Bad - 하나의 거대한 스토어
export const useStore = create((set) => ({
  user: null,
  session: null,
  meetings: [],
  participants: [],
  surveyResponses: [],
  // ... 모든 상태가 한 곳에
}));
```

### 5-2. 선택자 사용

```typescript
// ✅ Good - 선택자로 구독 최소화
function Header() {
  // user만 구독
  const user = useAuthStore((state) => state.user);
  return <span>{user?.email}</span>;
}

// ❌ Bad - 전체 스토어 구독
function Header() {
  // 스토어의 어떤 값이 바뀌어도 리렌더링
  const { user } = useAuthStore();
  return <span>{user?.email}</span>;
}
```

### 5-3. 불변성 유지

```typescript
// ✅ Good - 새 객체/배열 생성
updateParticipant: (id, updates) =>
  set((state) => ({
    participants: state.participants.map((p) =>
      p.id === id ? { ...p, ...updates } : p
    ),
  })),

removeParticipant: (id) =>
  set((state) => ({
    participants: state.participants.filter((p) => p.id !== id),
  })),

// ❌ Bad - 직접 수정
updateParticipant: (id, updates) =>
  set((state) => {
    const participant = state.participants.find((p) => p.id === id);
    participant.team_id = updates.team_id; // 직접 수정
    return { participants: state.participants };
  }),
```

---

## 6. Supabase 규칙

### 6-1. 클라이언트 사용

```typescript
// ✅ Good - 싱글톤 import
import { supabase } from '@/lib/supabase';

// ❌ Bad - 컴포넌트 내 클라이언트 생성
import { createClient } from '@supabase/supabase-js';

function MyComponent() {
  const supabase = createClient(url, key); // 매 렌더마다 새 인스턴스
}
```

### 6-2. 쿼리 패턴

```typescript
// ✅ Good - 명시적 컬럼 선택
const { data, error } = await supabase
  .from('meetings')
  .select('id, name, code, status, created_at')
  .eq('status', 'collecting')
  .order('created_at', { ascending: false });

// ❌ Bad - select('*')
const { data, error } = await supabase
  .from('meetings')
  .select('*'); // 불필요한 데이터까지 전송
```

### 6-3. 에러 처리

```typescript
// ✅ Good - 에러 확인 후 throw
async function fetchMeeting(id: string): Promise<Meeting> {
  const { data, error } = await supabase
    .from('meetings')
    .select('id, name, code, status')
    .eq('id', id)
    .single();

  if (error) {
    throw new Error(`모임 조회 실패: ${error.message}`);
  }

  return data; // TypeScript가 data는 null이 아님을 알음
}

// ❌ Bad - 에러 무시
async function fetchMeeting(id: string) {
  const { data } = await supabase
    .from('meetings')
    .select('*')
    .eq('id', id)
    .single();

  return data; // data가 null일 수 있음
}
```

### 6-4. RLS 신뢰

```typescript
// ✅ Good - RLS가 접근 제어를 담당
async function fetchMyMeetings(): Promise<Meeting[]> {
  const { data, error } = await supabase
    .from('meetings')
    .select('id, name, code, status');

  if (error) throw error;
  return data; // RLS가 owner_id = auth.uid() 필터링
}

// ❌ Bad - Frontend에서 수동 필터
async function fetchMyMeetings(userId: string): Promise<Meeting[]> {
  const { data, error } = await supabase
    .from('meetings')
    .select('*')
    .eq('owner_id', userId); // RLS가 이미 처리함

  if (error) throw error;
  return data;
}
```

---

## 7. Three.js / React Three Fiber 규칙

### 7-1. 선언적 패턴

```typescript
// ✅ Good - JSX 기반 선언적
function ParticipantSphere({ position, color, name }: ParticipantSphereProps) {
  const meshRef = useRef<THREE.Mesh>(null);

  return (
    <mesh ref={meshRef} position={position}>
      <sphereGeometry args={[0.2, 32, 32]} />
      <meshStandardMaterial color={color} />
    </mesh>
  );
}

// ❌ Bad - 명령형 Three.js
function ParticipantSphere({ position, color }: Props) {
  useEffect(() => {
    const geometry = new THREE.SphereGeometry(0.2, 32, 32);
    const material = new THREE.MeshStandardMaterial({ color });
    const mesh = new THREE.Mesh(geometry, material);
    mesh.position.set(...position);
    scene.add(mesh);

    return () => {
      scene.remove(mesh);
      geometry.dispose();
      material.dispose();
    };
  }, [position, color]);

  return null;
}
```

### 7-2. 애니메이션

```typescript
// ✅ Good - useFrame 사용
import { useFrame } from '@react-three/fiber';

function RotatingMesh() {
  const meshRef = useRef<THREE.Mesh>(null);

  useFrame((state, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta * 0.5;
    }
  });

  return (
    <mesh ref={meshRef}>
      <boxGeometry />
      <meshStandardMaterial />
    </mesh>
  );
}

// ❌ Bad - requestAnimationFrame 사용
function RotatingMesh() {
  const meshRef = useRef<THREE.Mesh>(null);

  useEffect(() => {
    let animationId: number;

    const animate = () => {
      if (meshRef.current) {
        meshRef.current.rotation.y += 0.01;
      }
      animationId = requestAnimationFrame(animate);
    };

    animate();
    return () => cancelAnimationFrame(animationId);
  }, []);

  return <mesh ref={meshRef}>...</mesh>;
}
```

### 7-3. 리소스 관리

```typescript
// ✅ Good - useLoader 사용
import { useLoader } from '@react-three/fiber';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

function Model({ url }: { url: string }) {
  const gltf = useLoader(GLTFLoader, url);
  return <primitive object={gltf.scene} />;
}

// ❌ Bad - 수동 로딩
function Model({ url }: { url: string }) {
  const [model, setModel] = useState<THREE.Group | null>(null);

  useEffect(() => {
    const loader = new GLTFLoader();
    loader.load(url, (gltf) => {
      setModel(gltf.scene);
    });
  }, [url]);

  if (!model) return null;
  return <primitive object={model} />;
}
```

---

## 8. 스타일링 규칙 (TailwindCSS)

### 8-1. 유틸리티 클래스 사용

```tsx
// ✅ Good - Tailwind 유틸리티
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div className="rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
      {children}
    </div>
  );
}

// ❌ Bad - 인라인 스타일
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div
      style={{
        borderRadius: '8px',
        border: '1px solid #e5e7eb',
        backgroundColor: 'white',
        padding: '16px',
        boxShadow: '0 1px 2px rgba(0,0,0,0.05)',
      }}
    >
      {children}
    </div>
  );
}
```

### 8-2. 조건부 클래스

```tsx
// ✅ Good - 템플릿 리터럴
function Button({ variant, disabled }: ButtonProps) {
  return (
    <button
      className={`
        rounded-lg px-4 py-2 font-medium transition-colors
        ${variant === 'primary' ? 'bg-blue-500 text-white' : 'bg-gray-100 text-gray-700'}
        ${disabled ? 'cursor-not-allowed opacity-50' : 'hover:opacity-90'}
      `}
      disabled={disabled}
    >
      클릭
    </button>
  );
}

// 또는 clsx/classnames 사용
import clsx from 'clsx';

function Button({ variant, disabled }: ButtonProps) {
  return (
    <button
      className={clsx(
        'rounded-lg px-4 py-2 font-medium transition-colors',
        {
          'bg-blue-500 text-white': variant === 'primary',
          'bg-gray-100 text-gray-700': variant === 'secondary',
          'cursor-not-allowed opacity-50': disabled,
          'hover:opacity-90': !disabled,
        }
      )}
    >
      클릭
    </button>
  );
}
```

### 8-3. 반응형 디자인

```tsx
// ✅ Good - 모바일 퍼스트
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  {meetings.map((m) => (
    <MeetingCard key={m.id} meeting={m} />
  ))}
</div>

// 텍스트 크기
<h1 className="text-xl font-bold md:text-2xl lg:text-3xl">
  팀 편성 결과
</h1>
```

---

## 9. 마이크로카피 규칙

토스 라이팅 원칙을 따릅니다.

### 9-1. 예측 가능한 힌트

```typescript
// ✅ Good - 다음 단계 안내
const BUTTON_TEXT = {
  survey_start: '설문 시작하기',      // 동작 설명
  survey_next: '다음 질문',           // 진행 방향
  survey_submit: '제출하고 결과 보기', // 결과 예고
  matching_start: '팀 편성 시작',
};

// ❌ Bad - 불분명한 텍스트
const BUTTON_TEXT = {
  survey_start: '확인',
  survey_next: '다음',
  survey_submit: '완료',
};
```

### 9-2. 잡초 제거 (간결함)

```typescript
// ✅ Good
const MESSAGES = {
  loading: '불러오는 중...',
  empty_meetings: '아직 모임이 없어요',
  success_create: '모임을 만들었어요',
};

// ❌ Bad
const MESSAGES = {
  loading: '데이터를 불러오는 중입니다. 잠시만 기다려 주세요.',
  empty_meetings: '현재 생성된 모임이 없습니다. 새 모임을 만들어 보세요.',
  success_create: '새로운 모임이 성공적으로 생성되었습니다.',
};
```

### 9-3. 보편적 단어 사용

```typescript
// ✅ Good
const LABELS = {
  code: '참가 코드',
  team: '팀',
  member: '멤버',
  result: '결과',
};

// ❌ Bad
const LABELS = {
  code: '인비테이션 토큰',
  team: '클러스터 그룹',
  member: '파티시펀트',
  result: '아웃풋 데이터',
};
```

### 9-4. 에러 메시지

```typescript
// ✅ Good - 원인과 해결책
const ERROR_MESSAGES = {
  network: '인터넷 연결을 확인해 주세요',
  not_found: '모임을 찾을 수 없어요. 코드를 다시 확인해 주세요',
  unauthorized: '로그인이 필요해요',
  server: '잠시 후 다시 시도해 주세요',
};

// ❌ Bad - 기술적 메시지
const ERROR_MESSAGES = {
  network: 'Network Error',
  not_found: '404 Not Found',
  unauthorized: 'Unauthorized',
  server: 'Internal Server Error',
};
```

---

## 10. 네이밍 규칙

### 10-1. 파일명

| 종류 | 규칙 | 예시 |
| --- | --- | --- |
| 컴포넌트 | PascalCase | `MeetingCard.tsx` |
| 훅 | camelCase + use 접두사 | `useMeeting.ts` |
| 스토어 | camelCase + Store 접미사 | `authStore.ts` |
| 유틸리티 | camelCase | `formatters.ts` |
| 타입 | camelCase 또는 PascalCase | `meeting.ts`, `database.types.ts` |
| 상수 | camelCase | `routes.ts` |

### 10-2. 변수/함수명

```typescript
// 변수: camelCase
const meetingCode = 'TB-ABCD';
const isLoading = true;
const participantCount = 25;

// 함수: camelCase, 동사로 시작
function fetchMeetings() {}
function createMeeting() {}
function handleSubmit() {}

// 상수: SCREAMING_SNAKE_CASE
const MAX_PARTICIPANTS = 100;
const API_TIMEOUT = 30000;
const TEAM_COLORS = ['#3B82F6', '#10B981'];

// 타입: PascalCase
type MeetingStatus = 'collecting' | 'matching' | 'completed';
interface ParticipantSphereProps {}
```

### 10-3. Boolean 네이밍

```typescript
// ✅ Good - is/has/can/should 접두사
const isLoading = true;
const hasError = false;
const canSubmit = true;
const shouldShowModal = false;

// ❌ Bad - 불분명한 네이밍
const loading = true;
const error = false;
const submit = true;
const modal = false;
```

---

## 11. 금지 사항 요약

| 분류 | ❌ 하지 말 것 | ✅ 대신 할 것 |
| --- | --- | --- |
| 타입 | `any` 사용 | 구체적 타입 정의 |
| Import | 상대 경로 (`../../../`) | 경로 별칭 (`@/`) |
| Supabase | 컴포넌트 내 클라이언트 생성 | 싱글톤 import |
| Supabase | `select('*')` | 명시적 컬럼 지정 |
| 컴포넌트 | 화살표 함수 + 별도 export | function 선언 + default export |
| Three.js | `requestAnimationFrame` | `useFrame` |
| 스타일 | 인라인 스타일 | TailwindCSS 클래스 |
| 상태 | 거대한 단일 스토어 | 도메인별 분리 |
| 에러 | 에러 무시 | throw 또는 에러 상태 관리 |

---

## 참고 문서

- [01_ARCHITECTURE.md](./01_ARCHITECTURE.md) - 시스템 아키텍처
- [02_FOLDER_STRUCTURE.md](./02_FOLDER_STRUCTURE.md) - 폴더 구조
- [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - AI 코드 생성 가이드
- [TEAMBLEND_PRD.md](./TEAMBLEND_PRD.md) - 마이크로카피 가이드
