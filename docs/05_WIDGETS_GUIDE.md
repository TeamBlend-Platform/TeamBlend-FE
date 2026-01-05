# TeamBlend 위젯 가이드

토스 UX 법칙을 기반으로 한 UI 컴포넌트 설계 가이드입니다.

---

## 1. UX 원칙 요약

### 1-1. 10가지 UX 심리학 법칙

| 법칙 | 핵심 | TeamBlend 적용 |
| --- | --- | --- |
| **제이콥의 법칙** | 익숙한 패턴 사용 | 일반적인 앱 UI 패턴 따르기 |
| **피츠의 법칙** | 터치 영역 최적화 | 버튼 최소 44x44px, 간격 8px 이상 |
| **힉의 법칙** | 선택지 줄이기 | 한 화면에 한 가지 목적 |
| **밀러의 법칙** | 정보 덩어리화 | 최대 7±2개 항목 그룹핑 |
| **포스텔의 법칙** | 입력 최소화 | 1 Thing / 1 Page |
| **피크엔드의 법칙** | 마지막 경험 중시 | 성공/실패 화면에 감동 요소 |
| **테슬러의 법칙** | 복잡성 시스템이 흡수 | 자동 완성, 추천 기능 |
| **심미적 사용성** | 아름다운 디자인 | 둥글고 부드러운 UI |
| **폰 레스토프 효과** | 차별화된 강조 | 중요 버튼에 컬러 강조 |
| **도허티 임계** | 0.4초 내 피드백 | 스켈레톤 UI, 로딩 애니메이션 |

### 1-2. 8가지 라이팅 원칙

| 원칙 | 핵심 | 예시 |
| --- | --- | --- |
| **예측 가능한 힌트** | 다음 단계 안내 | "다음 질문" → "제출하고 결과 보기" |
| **잡초 제거** | 불필요한 단어 삭제 | "혹시", "앞으로" 제거 |
| **빈 문장 제거** | 의미 없는 문장 삭제 | 중복 설명 제거 |
| **핵심 메시지 집중** | 꼭 필요한 정보만 | 이미 아는 정보 생략 |
| **말하기 쉽게** | 전문 용어 피하기 | "클러스터링" → "팀 나누기" |
| **강요 대신 권유** | 공포감 없이 안내 | "놓치면 손해" X |
| **보편적 표현** | 모두 이해 가능 | 유행어, 밈 지양 |
| **숨겨진 감정** | 감정에 공감 | 완료 시 축하 메시지 |

### 1-3. 5가지 코어밸류

| 가치 | 의미 | 적용 |
| --- | --- | --- |
| **Clear** | 명확한 | 한 번에 이해되는 문장 |
| **Concise** | 간결한 | 빠르게 스캔 가능 |
| **Casual** | 친근한 | 딱딱하지 않은 톤 |
| **Respect** | 존중하는 | 과장/숨김 없이 진실되게 |
| **Emotional** | 공감하는 | 사용자 감정에 연결 |

---

## 2. 버튼 (Button)

### 2-1. 디자인 원칙

```
피츠의 법칙: 충분한 터치 영역 (최소 44px 높이)
폰 레스토프 효과: Primary 버튼은 강조색으로 차별화
힉의 법칙: 한 화면에 주요 액션 버튼은 1-2개
```

### 2-2. 버튼 종류

| 종류 | 용도 | 스타일 |
| --- | --- | --- |
| **Primary** | 주요 액션 | bg-blue-500, 크게 |
| **Secondary** | 보조 액션 | bg-gray-100, 동일 크기 |
| **Ghost** | 취소, 부가 액션 | 투명 배경, 텍스트만 |
| **Destructive** | 삭제, 위험 액션 | bg-red-500 |

### 2-3. 구현 예시

```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost' | 'destructive';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export default function Button({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  children,
  onClick,
}: ButtonProps) {
  const baseStyles = 'rounded-xl font-medium transition-all duration-200';

  const sizeStyles = {
    sm: 'h-10 px-4 text-sm',    // 40px 높이
    md: 'h-12 px-6 text-base',  // 48px 높이 (기본)
    lg: 'h-14 px-8 text-lg',    // 56px 높이
  };

  const variantStyles = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600 active:scale-[0.98]',
    secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200',
    ghost: 'bg-transparent text-gray-600 hover:bg-gray-50',
    destructive: 'bg-red-500 text-white hover:bg-red-600',
  };

  const disabledStyles = 'opacity-50 cursor-not-allowed';

  return (
    <button
      type="button"
      className={`
        ${baseStyles}
        ${sizeStyles[size]}
        ${variantStyles[variant]}
        ${(disabled || loading) ? disabledStyles : ''}
      `}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading ? (
        <span className="flex items-center gap-2">
          <Spinner size="sm" />
          <span>처리 중...</span>
        </span>
      ) : (
        children
      )}
    </button>
  );
}
```

### 2-4. 버튼 텍스트 가이드

```
예측 가능한 힌트 원칙 적용
```

| 상황 | ❌ Bad | ✅ Good |
| --- | --- | --- |
| 설문 시작 | 확인 | 설문 시작하기 |
| 다음 질문 | 다음 | 다음 질문 |
| 설문 제출 | 완료 | 제출하고 결과 보기 |
| 팀 편성 | 실행 | 팀 편성 시작 |
| 모임 생성 | 만들기 | 모임 만들기 |
| 로그인 | 로그인 | Google로 시작하기 |

---

## 3. 입력 필드 (Input)

### 3-1. 디자인 원칙

```
포스텔의 법칙: 한 화면에 하나의 입력만 요청
테슬러의 법칙: 자동 완성, 형식 자동 변환
밀러의 법칙: 긴 입력은 청킹 (예: 전화번호 하이픈)
```

### 3-2. 구현 예시

```tsx
interface InputProps {
  label: string;
  placeholder?: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  hint?: string;
  maxLength?: number;
}

export default function Input({
  label,
  placeholder,
  value,
  onChange,
  error,
  hint,
  maxLength,
}: InputProps) {
  return (
    <div className="flex flex-col gap-2">
      <label className="text-sm font-medium text-gray-700">
        {label}
      </label>

      <input
        type="text"
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder={placeholder}
        maxLength={maxLength}
        className={`
          h-12 px-4 rounded-xl border-2 transition-colors
          focus:outline-none focus:border-blue-500
          ${error ? 'border-red-500' : 'border-gray-200'}
        `}
      />

      {error && (
        <p className="text-sm text-red-500">{error}</p>
      )}

      {hint && !error && (
        <p className="text-sm text-gray-500">{hint}</p>
      )}

      {maxLength && (
        <p className="text-xs text-gray-400 text-right">
          {value.length} / {maxLength}
        </p>
      )}
    </div>
  );
}
```

### 3-3. 입력 힌트 가이드

```
말하기 쉽게 원칙 + 예측 가능한 힌트 원칙
```

| 필드 | Placeholder | Hint |
| --- | --- | --- |
| 모임 이름 | 예: 2025 신입사원 OT | 2-50자 |
| 참가자 이름 | 이름을 입력해 주세요 | 다른 참가자에게 보여요 |
| 참가 코드 | TB-XXXX | 운영자에게 받은 코드 |

---

## 4. 카드 (Card)

### 4-1. 디자인 원칙

```
심미적 사용성: 둥글고 부드러운 모서리
힉의 법칙: 카드당 하나의 정보 덩어리
밀러의 법칙: 한 화면에 5-9개 카드
```

### 4-2. 구현 예시

```tsx
interface MeetingCardProps {
  meeting: {
    name: string;
    code: string;
    status: 'collecting' | 'matching' | 'completed';
    participantCount: number;
    createdAt: string;
  };
  onClick: () => void;
}

export default function MeetingCard({ meeting, onClick }: MeetingCardProps) {
  const statusConfig = {
    collecting: { label: '응답 수집 중', color: 'bg-blue-100 text-blue-700' },
    matching: { label: '매칭 중', color: 'bg-yellow-100 text-yellow-700' },
    completed: { label: '완료', color: 'bg-green-100 text-green-700' },
  };

  const status = statusConfig[meeting.status];

  return (
    <button
      onClick={onClick}
      className="
        w-full p-4 rounded-2xl border border-gray-200
        bg-white text-left
        hover:border-blue-200 hover:shadow-md
        transition-all duration-200
        active:scale-[0.98]
      "
    >
      {/* 상단: 이름 + 상태 */}
      <div className="flex justify-between items-start mb-3">
        <h3 className="font-semibold text-gray-900 line-clamp-1">
          {meeting.name}
        </h3>
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${status.color}`}>
          {status.label}
        </span>
      </div>

      {/* 하단: 메타 정보 */}
      <div className="flex items-center gap-4 text-sm text-gray-500">
        <span className="flex items-center gap-1">
          <UserIcon className="w-4 h-4" />
          {meeting.participantCount}명
        </span>
        <span>{meeting.code}</span>
        <span>{formatDate(meeting.createdAt)}</span>
      </div>
    </button>
  );
}
```

---

## 5. 설문 질문 (Survey Question)

### 5-1. 디자인 원칙

```
포스텔의 법칙: 1 Thing / 1 Page - 한 화면에 한 질문
힉의 법칙: 선택지 4-5개 이하
피츠의 법칙: 선택지 버튼 충분한 터치 영역
도허티 임계: 선택 시 즉각적 피드백 (색상 변화)
```

### 5-2. 질문 유형

| 유형 | 설명 | UI |
| --- | --- | --- |
| **Single** | 단일 선택 | 라디오 버튼 스타일 |
| **Multiple** | 복수 선택 | 체크박스 스타일 |
| **Scale** | 1-5점 척도 | 숫자 버튼 그리드 |

### 5-3. 구현 예시

```tsx
interface QuestionCardProps {
  question: {
    id: string;
    text: string;
    type: 'single' | 'multiple' | 'scale';
    options: Array<{ id: string; label: string; emoji?: string }>;
  };
  currentIndex: number;
  totalCount: number;
  selectedOptions: string[];
  onSelect: (optionId: string) => void;
}

export default function QuestionCard({
  question,
  currentIndex,
  totalCount,
  selectedOptions,
  onSelect,
}: QuestionCardProps) {
  return (
    <div className="flex flex-col h-full">
      {/* 진행률 표시 */}
      <div className="mb-6">
        <div className="flex justify-between text-sm text-gray-500 mb-2">
          <span>{currentIndex + 1} / {totalCount}</span>
        </div>
        <div className="h-1 bg-gray-200 rounded-full overflow-hidden">
          <div
            className="h-full bg-blue-500 transition-all duration-300"
            style={{ width: `${((currentIndex + 1) / totalCount) * 100}%` }}
          />
        </div>
      </div>

      {/* 질문 텍스트 */}
      <h2 className="text-xl font-bold text-gray-900 mb-8">
        {question.text}
      </h2>

      {/* 선택지 */}
      <div className="flex flex-col gap-3">
        {question.options.map((option) => {
          const isSelected = selectedOptions.includes(option.id);

          return (
            <button
              key={option.id}
              onClick={() => onSelect(option.id)}
              className={`
                w-full p-4 rounded-xl border-2 text-left
                transition-all duration-200
                ${isSelected
                  ? 'border-blue-500 bg-blue-50'
                  : 'border-gray-200 hover:border-blue-200'
                }
              `}
            >
              <span className="flex items-center gap-3">
                {option.emoji && (
                  <span className="text-2xl">{option.emoji}</span>
                )}
                <span className={isSelected ? 'font-medium' : ''}>
                  {option.label}
                </span>
              </span>
            </button>
          );
        })}
      </div>
    </div>
  );
}
```

### 5-4. 질문 텍스트 가이드

```
말하기 쉽게 + 간결하게 원칙
```

| ❌ Bad | ✅ Good |
| --- | --- |
| 귀하의 MBTI 유형을 선택해 주시기 바랍니다 | 당신의 MBTI는? |
| 팀 프로젝트에서 선호하는 역할을 선택하세요 | 팀에서 맡고 싶은 역할은? |
| 경력 기간을 아래에서 선택해 주세요 | 개발 경력이 어느 정도인가요? |

---

## 6. 모달 (Modal)

### 6-1. 디자인 원칙

```
힉의 법칙: 모달당 하나의 결정
피크엔드의 법칙: 확인/취소 시 명확한 피드백
심미적 사용성: 부드러운 진입 애니메이션
```

### 6-2. 구현 예시

```tsx
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  description?: string;
  children?: React.ReactNode;
  primaryAction?: {
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'destructive';
  };
  secondaryAction?: {
    label: string;
    onClick: () => void;
  };
}

export default function Modal({
  isOpen,
  onClose,
  title,
  description,
  children,
  primaryAction,
  secondaryAction,
}: ModalProps) {
  if (!isOpen) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      onClick={onClose}
    >
      {/* 배경 딤 */}
      <div className="absolute inset-0 bg-black/50 animate-fade-in" />

      {/* 모달 본체 */}
      <div
        className="
          relative z-10 w-[90%] max-w-md
          bg-white rounded-2xl p-6
          animate-slide-up
        "
        onClick={(e) => e.stopPropagation()}
      >
        <h2 className="text-lg font-bold text-gray-900 mb-2">
          {title}
        </h2>

        {description && (
          <p className="text-gray-600 mb-4">{description}</p>
        )}

        {children}

        {/* 액션 버튼 */}
        <div className="flex gap-3 mt-6">
          {secondaryAction && (
            <Button
              variant="ghost"
              className="flex-1"
              onClick={secondaryAction.onClick}
            >
              {secondaryAction.label}
            </Button>
          )}

          {primaryAction && (
            <Button
              variant={primaryAction.variant || 'primary'}
              className="flex-1"
              onClick={primaryAction.onClick}
            >
              {primaryAction.label}
            </Button>
          )}
        </div>
      </div>
    </div>
  );
}
```

### 6-3. 모달 문구 가이드

```
강요 대신 권유 + 숨겨진 감정 원칙
```

| 상황 | Title | Description |
| --- | --- | --- |
| 설문 이탈 | 지금 나가시겠어요? | 응답이 저장되지 않아요 |
| 모임 삭제 | 모임을 삭제할까요? | 참가자 응답도 함께 삭제돼요 |
| 팀 편성 시작 | 팀 편성을 시작할까요? | 진행 중에는 새 응답을 받을 수 없어요 |

---

## 7. 토스트 (Toast)

### 7-1. 디자인 원칙

```
도허티 임계: 즉각적 피드백 제공
피크엔드의 법칙: 성공/실패 경험에 감정 담기
```

### 7-2. 토스트 유형

| 유형 | 아이콘 | 색상 | 지속시간 |
| --- | --- | --- | --- |
| **Success** | ✓ | 초록 | 3초 |
| **Error** | ! | 빨강 | 5초 (더 오래) |
| **Info** | i | 파랑 | 3초 |
| **Warning** | ⚠ | 노랑 | 4초 |

### 7-3. 토스트 메시지 가이드

```
간결하게 + 숨겨진 감정 원칙
```

| 상황 | 메시지 |
| --- | --- |
| 모임 생성 성공 | 모임을 만들었어요! |
| 설문 제출 성공 | 응답을 제출했어요 |
| 팀 편성 완료 | 팀 편성이 완료됐어요! |
| 네트워크 에러 | 인터넷 연결을 확인해 주세요 |
| 서버 에러 | 잠시 후 다시 시도해 주세요 |
| 코드 오류 | 코드를 다시 확인해 주세요 |

---

## 8. 로딩 상태 (Loading States)

### 8-1. 디자인 원칙

```
도허티 임계: 0.4초 내 피드백
심미적 사용성: 지루하지 않은 애니메이션
```

### 8-2. 로딩 유형

| 상황 | UI | 예시 |
| --- | --- | --- |
| 페이지 로딩 | 스켈레톤 UI | 카드 목록, 프로필 |
| 버튼 액션 | 버튼 내 스피너 | 제출, 저장 |
| 데이터 처리 | 풀스크린 로딩 | 팀 편성 |

### 8-3. 스켈레톤 UI 예시

```tsx
function MeetingCardSkeleton() {
  return (
    <div className="p-4 rounded-2xl border border-gray-200 animate-pulse">
      {/* 제목 */}
      <div className="h-5 bg-gray-200 rounded w-3/4 mb-3" />

      {/* 메타 정보 */}
      <div className="flex gap-4">
        <div className="h-4 bg-gray-200 rounded w-16" />
        <div className="h-4 bg-gray-200 rounded w-20" />
        <div className="h-4 bg-gray-200 rounded w-24" />
      </div>
    </div>
  );
}

export default function MeetingListSkeleton() {
  return (
    <div className="grid gap-4">
      {[1, 2, 3].map((i) => (
        <MeetingCardSkeleton key={i} />
      ))}
    </div>
  );
}
```

### 8-4. 팀 편성 로딩 (특별 처리)

```
피크엔드의 법칙: 특별한 경험으로 기억에 남기기
```

```tsx
import { useState, useEffect } from 'react';
import Lottie from 'lottie-react';
import matchingAnimation from '@/assets/animations/matching.json';

export default function MatchingLoader() {
  const messages = [
    '참가자들을 분석하고 있어요',
    '최적의 팀 조합을 찾고 있어요',
    '거의 다 됐어요!',
  ];

  const [messageIndex, setMessageIndex] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setMessageIndex((prev) => (prev + 1) % messages.length);
    }, 2000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div className="fixed inset-0 bg-white flex flex-col items-center justify-center">
      {/* 애니메이션 아이콘 */}
      <div className="w-24 h-24 mb-8">
        <Lottie animationData={matchingAnimation} loop />
      </div>

      {/* 상태 메시지 */}
      <p className="text-lg font-medium text-gray-900 animate-fade-in">
        {messages[messageIndex]}
      </p>

      {/* 프로그레스 바 */}
      <div className="w-48 h-1 bg-gray-200 rounded-full mt-6 overflow-hidden">
        <div className="h-full bg-blue-500 animate-progress" />
      </div>
    </div>
  );
}
```

---

## 9. 빈 상태 (Empty State)

### 9-1. 디자인 원칙

```
숨겨진 감정: 허전함 대신 친근하게
예측 가능한 힌트: 다음 행동 안내
```

### 9-2. 구현 예시

```tsx
interface EmptyStateProps {
  icon?: React.ReactNode;
  title: string;
  description?: string;
  action?: {
    label: string;
    onClick: () => void;
  };
}

export default function EmptyState({
  icon,
  title,
  description,
  action,
}: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 px-4">
      {icon && (
        <div className="w-20 h-20 mb-4 text-gray-300">
          {icon}
        </div>
      )}

      <h3 className="text-lg font-medium text-gray-900 mb-2">
        {title}
      </h3>

      {description && (
        <p className="text-gray-500 text-center mb-6">
          {description}
        </p>
      )}

      {action && (
        <Button variant="primary" onClick={action.onClick}>
          {action.label}
        </Button>
      )}
    </div>
  );
}
```

### 9-3. 빈 상태 문구 가이드

| 상황 | Title | Description | Action |
| --- | --- | --- | --- |
| 모임 없음 | 아직 모임이 없어요 | 새 모임을 만들어 보세요! | 모임 만들기 |
| 참가자 없음 | 아직 응답이 없어요 | 참가 코드를 공유해 보세요 | 코드 복사하기 |
| 검색 결과 없음 | 검색 결과가 없어요 | 다른 검색어로 시도해 보세요 | - |

---

## 10. 3D 시각화 (Visualization)

### 10-1. 디자인 원칙

```
피크엔드의 법칙: 결과 확인의 "와!" 순간 만들기
폰 레스토프 효과: 내 위치 강조
피츠의 법칙: 터치/클릭 가능한 충분한 구체 크기
```

### 10-2. 인터랙션

| 동작 | 구현 | 피드백 |
| --- | --- | --- |
| 회전 | 드래그 | 부드러운 관성 |
| 줌 | 핀치/휠 | 제한 범위 내 |
| 참가자 선택 | 탭/클릭 | 정보 팝업 표시 |
| 내 위치 | 자동 | 글로우 효과 + 별 아이콘 |
| 팀 필터 | 범례 클릭 | 다른 팀 투명도 0.2 |

### 10-3. 팀 색상

```typescript
export const TEAM_COLORS = [
  '#3B82F6', // Blue
  '#10B981', // Green
  '#F59E0B', // Amber
  '#EF4444', // Red
  '#8B5CF6', // Purple
  '#EC4899', // Pink
  '#06B6D4', // Cyan
  '#F97316', // Orange
] as const;
```

### 10-4. 참가자 구체 구현

```tsx
import { useRef, useState } from 'react';
import { useFrame } from '@react-three/fiber';
import { Html } from '@react-three/drei';
import type { Mesh, Vector3 } from 'three';

interface ParticipantSphereProps {
  position: [number, number, number];
  color: string;
  name: string;
  isMe: boolean;
  isHighlighted: boolean;
  onClick: () => void;
}

export default function ParticipantSphere({
  position,
  color,
  name,
  isMe,
  isHighlighted,
  onClick,
}: ParticipantSphereProps) {
  const meshRef = useRef<Mesh>(null);
  const [hovered, setHovered] = useState(false);

  // 호버 시 크기 변화 애니메이션
  useFrame(() => {
    if (!meshRef.current) return;

    const targetScale = hovered ? 1.3 : 1;
    meshRef.current.scale.lerp(
      { x: targetScale, y: targetScale, z: targetScale } as Vector3,
      0.1
    );
  });

  return (
    <group position={position}>
      {/* 글로우 효과 (내 위치) */}
      {isMe && (
        <mesh>
          <sphereGeometry args={[0.35, 32, 32]} />
          <meshBasicMaterial color={color} transparent opacity={0.3} />
        </mesh>
      )}

      {/* 메인 구체 */}
      <mesh
        ref={meshRef}
        onClick={onClick}
        onPointerOver={() => setHovered(true)}
        onPointerOut={() => setHovered(false)}
      >
        <sphereGeometry args={[0.2, 32, 32]} />
        <meshStandardMaterial
          color={color}
          transparent
          opacity={isHighlighted ? 1 : 0.2}
        />
      </mesh>

      {/* 이름 라벨 (호버 시) */}
      {hovered && (
        <Html center>
          <div className="px-2 py-1 bg-gray-900 text-white text-xs rounded whitespace-nowrap">
            {name} {isMe && '(나)'}
          </div>
        </Html>
      )}

      {/* 별 아이콘 (내 위치) */}
      {isMe && (
        <Html center position={[0, 0.4, 0]}>
          <span className="text-yellow-400 text-lg">⭐</span>
        </Html>
      )}
    </group>
  );
}
```

---

## 11. 반응형 디자인

### 11-1. 브레이크포인트

| 이름 | 크기 | 대상 |
| --- | --- | --- |
| sm | 640px | 큰 모바일 |
| md | 768px | 태블릿 |
| lg | 1024px | 작은 데스크톱 |
| xl | 1280px | 데스크톱 |

### 11-2. 적용 예시

```tsx
// 그리드 레이아웃
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  {meetings.map((m) => <MeetingCard key={m.id} meeting={m} />)}
</div>

// 텍스트 크기
<h1 className="text-xl font-bold md:text-2xl lg:text-3xl">
  팀 편성 결과
</h1>

// 패딩
<div className="px-4 py-6 md:px-8 md:py-10 lg:px-12">
  {/* content */}
</div>
```

---

## 12. 접근성 (Accessibility)

### 12-1. 체크리스트

| 항목 | 구현 |
| --- | --- |
| 색맹 대응 | 색상 외 아이콘/텍스트로 상태 표시 |
| 키보드 탐색 | tabIndex, focus 스타일 |
| 스크린 리더 | aria-label, role 속성 |
| 터치 영역 | 최소 44x44px |
| 대비 | WCAG AA (4.5:1) 이상 |

### 12-2. 구현 예시

```tsx
// 버튼 접근성
<button
  type="button"
  aria-label="모임 삭제"
  aria-disabled={isLoading}
  className="focus:ring-2 focus:ring-blue-500 focus:outline-none"
>
  <TrashIcon className="w-5 h-5" aria-hidden="true" />
</button>

// 설문 선택지 접근성
<div role="radiogroup" aria-label="MBTI 선택">
  {options.map((opt) => (
    <button
      key={opt.id}
      role="radio"
      aria-checked={selected === opt.id}
      onClick={() => onSelect(opt.id)}
    >
      {opt.label}
    </button>
  ))}
</div>

// 이미지 접근성
<img src={qrCode} alt="참가 QR 코드" />

// 로딩 상태 알림
<div role="status" aria-live="polite">
  {isLoading && <span className="sr-only">로딩 중</span>}
</div>
```

---

## 참고 문서

- [TOSS_UI_UX_DESIGN.md](./TOSS_UI_UX_DESIGN.md) - 토스 UX 법칙 원문
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 마이크로카피 규칙
- [04_CODE_GENERATION_GUIDE.md](./04_CODE_GENERATION_GUIDE.md) - 컴포넌트 생성 프롬프트
- [Apple HIG](https://developer.apple.com/design/) - iOS 터치 가이드라인
