# TeamBlend 아키텍처 문서

---

## 1. 시스템 개요

TeamBlend는 **Supabase + FastAPI ML 하이브리드 아키텍처**를 사용합니다.

- **Supabase**: 인증, 데이터베이스, 파일 스토리지 담당
- **FastAPI**: ML 처리(클러스터링, 차원 축소)만 담당

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client (React)                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   React 18      │  │    Three.js     │  │     Zustand     │ │
│  │   + TypeScript  │  │ React Three    │  │   (상태관리)     │ │
│  │   + TailwindCSS │  │   Fiber        │  │                 │ │
│  └────────┬────────┘  └─────────────────┘  └─────────────────┘ │
└───────────┼─────────────────────────────────────────────────────┘
            │ Supabase SDK (REST API)
            ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Supabase Cloud                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │      Auth       │  │   PostgreSQL    │  │     Storage     │ │
│  │  Google/Kakao   │  │   (RLS 적용)    │  │  (QR 이미지)    │ │
│  │     OAuth       │  │                 │  │                 │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
            │ JWT 전달
            ▼
┌─────────────────────────────────────────────────────────────────┐
│                  FastAPI ML Microservice                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  JWT 검증       │  │    K-means      │  │     t-SNE       │ │
│  │  (Supabase)     │  │   클러스터링    │  │   3D 좌표       │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 핵심 설계 원칙

### 2-1. Supabase First

| 원칙 | 설명 |
| --- | --- |
| **인증은 Supabase Auth** | 직접 JWT 발급 X, Supabase OAuth 사용 |
| **데이터는 Supabase DB** | PostgreSQL + RLS로 보안 처리 |
| **파일은 Supabase Storage** | QR 이미지 저장 |
| **FastAPI는 ML만** | 클러스터링, 차원 축소 연산만 담당 |

### 2-2. RLS (Row Level Security) First

```sql
-- RLS가 접근 제어를 담당
-- Frontend에서 owner_id 검증 코드 불필요

CREATE POLICY "Users can view own meetings" ON meetings
  FOR SELECT USING (auth.uid() = owner_id);
```

- ✅ 데이터베이스 레벨에서 접근 제어
- ❌ Frontend에서 `if (meeting.owner_id !== user.id)` 같은 코드 불필요

### 2-3. Stateless ML Service

- FastAPI는 **상태를 저장하지 않음**
- 입력: 설문 응답 배열
- 출력: 팀 배정 + 3D 좌표
- DB 접근 없음 (Supabase에 저장은 Frontend가 담당)

---

## 3. 데이터 흐름

### 3-1. 운영자 플로우

```
[1. 로그인]
    │
    │ supabase.auth.signInWithOAuth({ provider: 'google' })
    ▼
[Supabase Auth] → 세션 발급 (자동 관리)
    │
    │ onAuthStateChange → authStore 동기화
    ▼
[2. 모임 생성]
    │
    │ supabase.from('meetings').insert({ name, template_id, code })
    ▼
[Supabase DB] → meetings 테이블에 저장
    │
    ▼
[3. 참가자 응답 수집 (대기)]
    │
    │ supabase.from('participants').select().eq('meeting_id', id)
    ▼
[4. 팀 편성 실행]
    │
    │ POST /api/matching (JWT 포함)
    ▼
[FastAPI ML] → K-means + t-SNE 처리
    │
    │ { teams: [...], positions_3d: [...] }
    ▼
[5. 결과 저장]
    │
    │ supabase.from('participants').update({ team_id, position_3d })
    ▼
[6. 3D 시각화 렌더링]
```

### 3-2. 참가자 플로우

```
[1. 코드 입력 / QR 스캔]
    │
    │ supabase.from('meetings').select().eq('code', 'TB-XXXX')
    ▼
[Supabase DB] → 모임 정보 조회
    │
    ▼
[2. 설문 진행]
    │
    │ (클라이언트 상태로 응답 수집)
    ▼
[3. 설문 제출]
    │
    │ supabase.from('participants').insert({
    │   meeting_id, name, participant_code, survey_responses
    │ })
    ▼
[Supabase DB] → participants 테이블에 저장
    │
    ▼
[4. 결과 대기 / 확인]
    │
    │ supabase.from('participants')
    │   .select('team_id, position_3d')
    │   .eq('participant_code', 'P-XXXXXX')
    ▼
[3D 시각화에서 내 위치 확인]
```

### 3-3. ML 매칭 플로우

```
[Frontend]
    │
    │ 1. 참가자 목록 조회
    │    supabase.from('participants').select('id, survey_responses')
    │
    │ 2. JWT 추출
    │    const { session } = await supabase.auth.getSession()
    │
    │ 3. ML API 호출
    │    POST /api/matching
    │    Authorization: Bearer {access_token}
    │    Body: { meeting_id, participants, num_teams, strategy }
    ▼
[FastAPI ML]
    │
    │ 1. JWT 검증 (Supabase SDK 사용)
    │ 2. 설문 응답 → 특성 벡터 변환
    │ 3. K-means 클러스터링 (팀 배정)
    │ 4. t-SNE 차원 축소 (3D 좌표)
    │
    │ Response: { teams, positions_3d, matching_scores }
    ▼
[Frontend]
    │
    │ 4. 결과 저장
    │    participants.forEach((p, i) => {
    │      supabase.from('participants').update({
    │        team_id: teams[i],
    │        position_3d: positions_3d[i]
    │      }).eq('id', p.id)
    │    })
    ▼
[3D 시각화 렌더링]
```

---

## 4. 인증 아키텍처

### 4-1. Supabase Auth 플로우

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend (React)                          │
│                                                                 │
│  [Google 로그인 버튼 클릭]                                       │
│         │                                                        │
│         ▼                                                        │
│  supabase.auth.signInWithOAuth({                                │
│    provider: 'google',                                          │
│    options: { redirectTo: window.location.origin }              │
│  })                                                             │
│         │                                                        │
└─────────┼────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Supabase Auth Server                         │
│                                                                 │
│  1. Google OAuth 리다이렉트                                      │
│  2. 사용자 인증 (Google 계정)                                    │
│  3. auth.users 테이블에 사용자 생성/업데이트                      │
│  4. JWT 발급 (Access Token + Refresh Token)                     │
│  5. 콜백 URL로 리다이렉트 (토큰 포함)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend (React)                          │
│                                                                 │
│  [콜백 처리]                                                     │
│         │                                                        │
│         ▼                                                        │
│  supabase.auth.onAuthStateChange((event, session) => {          │
│    if (event === 'SIGNED_IN') {                                 │
│      authStore.setUser(session.user)                            │
│      authStore.setSession(session)                              │
│    }                                                            │
│  })                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4-2. 세션 관리

| 항목 | 값 | 설명 |
| --- | --- | --- |
| Access Token | 1시간 | API 요청 인증 |
| Refresh Token | 7일 | Access Token 갱신 |
| 저장 위치 | localStorage + Cookie | Supabase SDK 자동 관리 |
| 갱신 | 자동 | SDK가 만료 전 자동 갱신 |

### 4-3. ML API 인증

```typescript
// Frontend: ML API 호출 시 JWT 전달
async function runMatching(meetingId: string, participants: Participant[]) {
  const { data: { session } } = await supabase.auth.getSession();

  if (!session) {
    throw new Error('인증이 필요합니다');
  }

  const response = await axios.post(
    `${ML_API_URL}/api/matching`,
    { meeting_id: meetingId, participants, num_teams: 5 },
    { headers: { Authorization: `Bearer ${session.access_token}` } }
  );

  return response.data;
}
```

```python
# FastAPI: JWT 검증
from supabase import create_client

supabase = create_client(SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY)

async def verify_token(authorization: str = Header(...)):
    token = authorization.replace("Bearer ", "")
    try:
        user = supabase.auth.get_user(token)
        return user
    except Exception:
        raise HTTPException(status_code=401, detail="Invalid token")
```

---

## 5. 데이터베이스 설계

### 5-1. ERD (Entity Relationship Diagram)

```
┌─────────────────┐       ┌─────────────────┐
│   auth.users    │       │    templates    │
│  (Supabase)     │       │                 │
├─────────────────┤       ├─────────────────┤
│ id (UUID) PK    │       │ id (TEXT) PK    │
│ email           │       │ name            │
│ ...             │       │ icon            │
└────────┬────────┘       │ questions (JSONB)│
         │                │ matching_strategy│
         │ 1              └────────┬────────┘
         │                         │
         │                         │ 1
         ▼                         │
┌─────────────────┐                │
│    meetings     │◄───────────────┘
├─────────────────┤
│ id (UUID) PK    │
│ owner_id (FK)   │───────┐
│ name            │       │
│ code (UNIQUE)   │       │
│ template_id (FK)│       │
│ status          │       │
│ team_count      │       │
│ matching_strategy│      │
│ created_at      │       │
└────────┬────────┘       │
         │                │
         │ 1              │
         │                │
         ▼                │
┌─────────────────┐       │
│  participants   │       │
├─────────────────┤       │
│ id (UUID) PK    │       │
│ meeting_id (FK) │───────┘
│ name            │
│ participant_code│
│ survey_responses│
│ team_id         │
│ position_3d     │
│ matching_score  │
│ submitted_at    │
└─────────────────┘
```

### 5-2. 테이블 상세

#### meetings

| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | UUID | Primary Key |
| owner_id | UUID | FK → auth.users |
| name | TEXT | 모임 이름 |
| code | TEXT | 설문 코드 (TB-XXXX) |
| template_id | TEXT | FK → templates |
| status | TEXT | collecting / matching / completed |
| team_count | INTEGER | 팀 수 |
| matching_strategy | TEXT | similarity / complementary / balanced |
| created_at | TIMESTAMPTZ | 생성 시간 |
| updated_at | TIMESTAMPTZ | 수정 시간 |

#### participants

| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | UUID | Primary Key |
| meeting_id | UUID | FK → meetings |
| name | TEXT | 참가자 이름 |
| participant_code | TEXT | 참가 코드 (P-XXXXXX) |
| survey_responses | JSONB | 설문 응답 데이터 |
| team_id | INTEGER | 배정된 팀 번호 |
| position_3d | JSONB | [x, y, z] 3D 좌표 |
| matching_score | DECIMAL | 매칭 점수 |
| submitted_at | TIMESTAMPTZ | 제출 시간 |

#### templates

| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | TEXT | Primary Key (hackathon, study 등) |
| name | TEXT | 템플릿 이름 |
| icon | TEXT | 이모지 아이콘 |
| description | TEXT | 설명 |
| questions | JSONB | 질문 배열 |
| matching_strategy | TEXT | 기본 매칭 전략 |

### 5-3. RLS 정책

```sql
-- meetings: 소유자만 접근
ALTER TABLE meetings ENABLE ROW LEVEL SECURITY;

CREATE POLICY "select_own_meetings" ON meetings
  FOR SELECT USING (auth.uid() = owner_id);

CREATE POLICY "insert_own_meetings" ON meetings
  FOR INSERT WITH CHECK (auth.uid() = owner_id);

CREATE POLICY "update_own_meetings" ON meetings
  FOR UPDATE USING (auth.uid() = owner_id);

CREATE POLICY "delete_own_meetings" ON meetings
  FOR DELETE USING (auth.uid() = owner_id);

-- participants: 소유자 조회, 누구나 생성
ALTER TABLE participants ENABLE ROW LEVEL SECURITY;

CREATE POLICY "select_meeting_participants" ON participants
  FOR SELECT USING (
    -- 모임 소유자는 해당 모임의 모든 참가자 조회 가능
    meeting_id IN (SELECT id FROM meetings WHERE owner_id = auth.uid())
  );

-- 참가자 본인 조회용 별도 정책 (RPC 함수 통해 participant_code로 조회)
CREATE POLICY "select_own_participant" ON participants
  FOR SELECT USING (true);  -- 공개 조회 허용 (participant_code는 추측 불가능한 UUID)

CREATE POLICY "insert_participants" ON participants
  FOR INSERT WITH CHECK (true);  -- 누구나 참가 가능

CREATE POLICY "update_meeting_participants" ON participants
  FOR UPDATE USING (
    meeting_id IN (SELECT id FROM meetings WHERE owner_id = auth.uid())
  );
```

---

## 6. ML 매칭 엔진

### 6-1. 알고리즘 파이프라인

```
┌─────────────────────────────────────────────────────────────────┐
│                    설문 응답 수집                                │
│  { mbti: "ENFP", role: "frontend", experience: 3, ... }        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    특성 벡터 변환                                │
│                                                                 │
│  MBTI "ENFP" → [1, -1, 1, -1]  (E/I, S/N, T/F, J/P)           │
│  role "frontend" → [0, 0, 1, 0]  (원핫 인코딩)                  │
│  experience 3 → [0.6]  (정규화 0~1)                            │
│                                                                 │
│  최종 벡터: [1, -1, 1, -1, 0, 0, 1, 0, 0.6, ...]              │
│  가중치 적용: 각 질문의 weight 값 곱하기                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    K-means 클러스터링                            │
│                                                                 │
│  Input: 특성 벡터 배열, k (팀 수)                                │
│  Algorithm: sklearn.cluster.KMeans                              │
│  Output: 각 참가자의 클러스터(팀) 번호                           │
│                                                                 │
│  전략별 거리 함수 조정:                                          │
│  - similarity: 유클리드 거리 (비슷한 사람끼리)                   │
│  - complementary: 역 코사인 유사도 (다양한 사람끼리)             │
│  - balanced: 제약조건 기반 배정                                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    t-SNE 차원 축소                               │
│                                                                 │
│  Input: 고차원 특성 벡터                                         │
│  Algorithm: sklearn.manifold.TSNE(n_components=3)               │
│  Output: 3D 좌표 [x, y, z]                                      │
│                                                                 │
│  Perplexity: 참가자 수에 따라 동적 조정                          │
│  - 30명 이하: perplexity = n_samples / 3                        │
│  - 30명 이상: perplexity = 30                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    결과 반환                                     │
│                                                                 │
│  {                                                              │
│    "teams": [0, 2, 1, 0, 2, 1, 3, 4],                          │
│    "positions_3d": [[0.12, -0.34, 0.56], ...],                 │
│    "matching_scores": [0.92, 0.87, ...]                        │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
```

### 6-2. 매칭 전략

| 전략 | 설명 | 사용 케이스 |
| --- | --- | --- |
| **similarity** | 비슷한 성향끼리 묶음 | 스터디 (목표 일치) |
| **complementary** | 다양한 성향끼리 묶음 | 해커톤 (역할 다양성) |
| **balanced** | 골고루 분배 | MT (에너지 균형) |
| **diversity** | 최대 다양성 확보 | 워크샵 (부서 믹스) |

### 6-3. API 명세

**Endpoint**: `POST /api/matching`

**Request Headers**:
```
Authorization: Bearer {supabase_access_token}
Content-Type: application/json
```

**Request Body**:
```json
{
  "meeting_id": "550e8400-e29b-41d4-a716-446655440000",
  "participants": [
    {
      "id": "uuid-1",
      "survey_responses": {
        "mbti": "ENFP",
        "role": "frontend",
        "experience": 3,
        "leadership": 4
      }
    },
    {
      "id": "uuid-2",
      "survey_responses": {
        "mbti": "ISTJ",
        "role": "backend",
        "experience": 5,
        "leadership": 2
      }
    }
  ],
  "num_teams": 5,
  "strategy": "complementary"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "teams": [0, 2, 1, 0, 2],
    "positions_3d": [
      [0.12, -0.34, 0.56],
      [0.78, 0.23, -0.45],
      [-0.33, 0.67, 0.12],
      [0.45, -0.12, 0.89],
      [-0.67, 0.45, -0.23]
    ],
    "matching_scores": [0.92, 0.87, 0.91, 0.85, 0.89]
  },
  "meta": {
    "processing_time_ms": 1234,
    "algorithm": "kmeans",
    "strategy": "complementary"
  }
}
```

---

## 7. 3D 시각화 아키텍처

### 7-1. 기술 스택

| 라이브러리 | 버전 | 용도 |
| --- | --- | --- |
| three | ^0.160.0 | 3D 렌더링 엔진 |
| @react-three/fiber | ^8.15.0 | React 바인딩 |
| @react-three/drei | ^9.96.0 | 헬퍼 (OrbitControls 등) |

### 7-2. 컴포넌트 구조

```
<Canvas>
├── <ambientLight />
├── <pointLight />
├── <Stars />  (배경)
├── <OrbitControls />  (카메라 조작)
└── {participants.map(p => (
      <ParticipantSphere
        position={p.position_3d}
        color={TEAM_COLORS[p.team_id]}
        name={p.name}
        isMe={p.id === currentUserId}
        isHighlighted={p.team_id === selectedTeam}
        onClick={() => onSelectParticipant(p.id)}
      />
    ))}
```

### 7-3. 인터랙션

| 동작 | 구현 |
| --- | --- |
| 회전 | OrbitControls 드래그 |
| 줌 | 마우스 휠 |
| 참가자 선택 | Raycaster + onClick |
| 팀 필터 | 다른 팀 투명도 0.2 |
| 내 위치 강조 | 글로우 효과 + 별 아이콘 |

---

## 8. 배포 아키텍처

### 8-1. 배포 구성

```
┌─────────────────────────────────────────────────────────────────┐
│                          Vercel                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Frontend (React)                          ││
│  │  - Vite 빌드                                                 ││
│  │  - 정적 파일 CDN 배포                                        ││
│  │  - 환경변수: VITE_SUPABASE_URL, VITE_ML_API_URL             ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Supabase Cloud                            │
│  - Auth: Google, Kakao OAuth 설정                               │
│  - Database: PostgreSQL + RLS                                   │
│  - Storage: QR 이미지 버킷                                      │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                          Render                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 FastAPI ML Service                           ││
│  │  - Docker 컨테이너                                           ││
│  │  - Python 3.11 + scikit-learn                               ││
│  │  - 환경변수: SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY        ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### 8-2. 환경변수

#### Frontend (Vercel)

```env
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIs...
VITE_ML_API_URL=https://teamblend-ml.onrender.com
```

#### ML Backend (Render)

```env
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIs...
CORS_ORIGINS=["https://teamblend.vercel.app"]
```

### 8-3. CORS 설정

```python
# FastAPI CORS
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://teamblend.vercel.app"],
    allow_credentials=True,
    allow_methods=["POST"],
    allow_headers=["Authorization", "Content-Type"],
)
```

---

## 9. 보안 고려사항

### 9-1. 인증/인가

| 항목 | 구현 |
| --- | --- |
| OAuth | Supabase Auth (Google, Kakao) |
| JWT 검증 | Supabase SDK (자동 갱신) |
| RLS | PostgreSQL 레벨 접근 제어 |
| ML API | JWT Bearer 토큰 필수 |

### 9-2. 데이터 보호

| 항목 | 구현 |
| --- | --- |
| 전송 암호화 | HTTPS (Vercel, Supabase, Render 기본 제공) |
| 저장 암호화 | Supabase PostgreSQL (at-rest encryption) |
| 민감 정보 | 환경변수로 분리 |

### 9-3. 키 관리

| 키 | 사용처 | 노출 범위 |
| --- | --- | --- |
| `SUPABASE_ANON_KEY` | Frontend | 공개 (RLS로 보호) |
| `SUPABASE_SERVICE_ROLE_KEY` | ML Backend | 비공개 (서버만) |

---

## 10. 성능 최적화

### 10-1. Frontend

| 최적화 | 구현 |
| --- | --- |
| 코드 스플리팅 | React.lazy() + Suspense |
| 3D 렌더링 | useFrame 대신 조건부 렌더링 |
| 상태 관리 | Zustand 선택자로 불필요한 리렌더링 방지 |
| 빌드 최적화 | Vite 트리셰이킹 |

### 10-2. ML API

| 최적화 | 구현 |
| --- | --- |
| 비동기 처리 | FastAPI async |
| 캐싱 | 동일 요청 결과 캐싱 (선택적) |
| 타임아웃 | 30초 타임아웃 설정 |

### 10-3. Database

| 최적화 | 구현 |
| --- | --- |
| 인덱스 | meetings.code, participants.participant_code |
| 쿼리 최적화 | 명시적 select 컬럼 지정 |
| 연결 풀링 | Supabase 기본 제공 |

---

## 참고 문서

- [TEAMBLEND_PRD.md](./TEAMBLEND_PRD.md) - 제품 요구사항 문서
- [02_FOLDER_STRUCTURE.md](./02_FOLDER_STRUCTURE.md) - 폴더 구조
- [03_CODE_CONVENTIONS.md](./03_CODE_CONVENTIONS.md) - 코드 컨벤션
- [Supabase 공식 문서](https://supabase.com/docs)
- [React Three Fiber 문서](https://docs.pmnd.rs/react-three-fiber)
