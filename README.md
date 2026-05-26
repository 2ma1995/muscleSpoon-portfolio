# 💪 MuscleMemory

**헬스장 통합 관리 SaaS 플랫폼 — 관리자 웹 + 회원 앱**

![1인 풀스택](https://img.shields.io/badge/1인_풀스택_개발-000?style=for-the-badge)
![기간](https://img.shields.io/badge/2026.05_~_진행중-333?style=for-the-badge)
![SaaS](https://img.shields.io/badge/Multi--tenant_SaaS-6366f1?style=for-the-badge)

> 헬스장 운영에 필요한 **회원 관리, 수업 스케줄, PT, 회원권/결제, 락커, 실시간 채팅, 통계**까지  
> 하나의 플랫폼으로 제공하는 풀스택 SaaS 프로젝트입니다.  
> **멀티테넌트 아키텍처**로 다수의 헬스장을 하나의 시스템에서 격리 운영합니다.

---

## 📸 화면 미리보기

### 관리자 웹

|      로그인 (관리자)      |           대시보드            |
| :-----------------------: | :---------------------------: |
| ![로그인](img/로그인.png) | ![대시보드](img/대시보드.png) |

|          수업 스케줄          |       오늘의 운동 (에디터)        |
| :---------------------------: | :-------------------------------: |
| ![로그인](img/수업스케줄.png) | ![대시보드](img/오늘운동작성.png) |

|            회원 관리            |           회원 등록           |           회원 수정           |
| :-----------------------------: | :---------------------------: | :---------------------------: |
| ![회원선택](img/회원관리_1.png) | ![회원생성](img/회원추가.png) | ![회원수정](img/회원수정.png) |

|         락커 관리         |          통계 (매출)          |
| :-----------------------: | :---------------------------: |
| ![락커](img/락커관리.png) | ![통계화면](img/통계화면.png) |

|             공지 사항             |           피티 관리           |
| :-------------------------------: | :---------------------------: |
| ![공지사항](img/공지사항화면.png) | ![피티관리](img/피티관리.png) |

|        설정 — 강사 관리         |       설정 — 수업 템플릿        |
| :-----------------------------: | :-----------------------------: |
| ![설정_강사](img/설정_강사.png) | ![설정_수업](img/설정_수업.png) |

|        설정 — 회원권 템플릿         |        설정 — 운영 정책        |
| :---------------------------------: | :----------------------------: |
| ![설정_회원권](img/설정_회원권.png) | ![설정_운영정책](img/설정.png) |

### 회원 앱 (PWA)

|             로그인             |             수업 예약              |             예약 완료              |             예약 변경              |
| :----------------------------: | :--------------------------------: | :--------------------------------: | :--------------------------------: |
| ![로그인](img/user/로그인.png) | ![수업예약](img/user/수업예약.png) | ![예약완료](img/user/예약완료.png) | ![예약변경](img/user/예약변경.png) |

|                 마이페이지                 |           설정 화면            |           만료 회원권            |
| :----------------------------------------: | :----------------------------: | :------------------------------: |
| ![마이페이지](img/user/마이페이지수정.png) | ![설정화면](img/user/설정.png) | ![만료회원권](img/user/만료.png) |

|             이전 수업              |       예약 불가 (오픈 전)       |         휴일 안내          |
| :--------------------------------: | :-----------------------------: | :------------------------: |
| ![이전수업](img/user/지난수업.png) | ![예약불가](img/user/1주전.png) | ![휴일](img/user/휴일.png) |

---

## ⚙️ 기술 스택

| 영역           | 기술                                                                                         |
| :------------- | :------------------------------------------------------------------------------------------- |
| **Backend**    | Java 17 · Spring Boot 3.5 · Spring Security 6 · Spring Data JPA · Flyway · WebSocket (STOMP) |
| **Frontend**   | Next.js 16 · TypeScript 5 · Tailwind CSS 4 · shadcn/ui · Recharts · PWA (Service Worker)     |
| **Database**   | MySQL 8.0 (Google Cloud SQL)                                                                 |
| **Infra**      | Google Cloud Run · Cloudflare (DNS/CDN/SSL) · Firebase FCM (Web Push)                        |
| **Auth**       | JWT (Access + Refresh Token) · RBAC (SUPER_ADMIN / STAFF / MEMBER)                           |
| **Rate Limit** | Bucket4j 8.10 (Token-bucket)                                                                 |

---

## 🏗️ 시스템 아키텍처

```mermaid
graph LR
  subgraph Clients
    A["🖥️ Admin Web<br/>Next.js 16"]
    B["📱 Member App<br/>Next.js PWA"]
  end

  subgraph Security
    C{"🔐 Spring Security 6<br/>JWT Filter · RBAC"}
  end

  subgraph Backend
    D["⚙️ Spring Boot 3.5<br/>REST API · 25 Controllers"]
    E["💬 WebSocket<br/>STOMP Chat"]
  end

  subgraph Data
    F[("🗄️ MySQL 8.0<br/>Cloud SQL · gym_id 격리")]
  end

  subgraph External
    G["🔔 Firebase FCM<br/>Web Push"]
    H["☁️ Cloud Run + Cloudflare"]
  end

  A -->|HTTPS / REST| C
  B -->|HTTPS / REST| C
  B -.->|STOMP| E
  C --> D
  C --> E
  D --> F
  D -.->|Push API| G
  G -.->|Web Push| B
  D --> H
```

### 멀티테넌트 설계

```
┌─ Gym A (gymId=1) ─────────────┐  ┌─ Gym B (gymId=2) ─────────────┐
│  Members │ Tickets │ Lessons   │  │  Members │ Tickets │ Lessons   │
│  Bookings│ Lockers │ Settings  │  │  Bookings│ Lockers │ Settings  │
└───────────────────────────────┘  └───────────────────────────────┘
              ▲                                  ▲
              └──── 단일 MySQL DB, gym_id 컬럼으로 논리적 격리 ────┘
                    Hibernate @Filter → 모든 쿼리에 WHERE gym_id = ? 자동 적용
```

> 📐 **[인터랙티브 아키텍처 다이어그램 보기 →](https://2ma1995.github.io/muscleSpoon-portfolio/architecture.html)**  
> 8개 핵심 플로우(수업 예약, 로그인, 채팅, 푸시 알림 등)의 단계별 데이터 흐름을 시각적으로 확인할 수 있습니다.

---

## 🚀 주요 기능

### 관리자 웹 (STAFF / SUPER_ADMIN)

| 기능            | 설명                                               | 핵심 기술          |
| :-------------- | :------------------------------------------------- | :----------------- |
| **대시보드**    | 오늘의 수업 현황, 실시간 예약/취소 활동 피드       | WebSocket STOMP    |
| **수업 스케줄** | 주간 뷰 관리, 원클릭 수업 개설/삭제                | 커스텀 WeekPicker  |
| **회원 관리**   | 회원 CRUD, 인바디 기록, PR 기록, 회원 검색         | JPA Dynamic Query  |
| **회원권/결제** | 회원권 발급, 홀드, 미수금 처리, 결제 내역          | @Transactional     |
| **PT 관리**     | PT 일정, 운동 기록(세트/무게/횟수), 코칭 일지      | 중첩 엔티티 CRUD   |
| **오늘의 운동** | HTML 리치 에디터로 운동 메뉴 작성                  | DOMPurify XSS 방어 |
| **통계**        | 월별 매출, 회원 증감, 수업 참여율 차트             | Recharts           |
| **채팅**        | 회원 ↔ 관리자 실시간 1:1 채팅                      | WebSocket STOMP    |
| **공지사항**    | 공지 작성 → 회원 앱 푸시 알림 자동 발송            | Firebase FCM       |
| **락커 관리**   | 락커 배정/반납/고장 처리, 수량 조정                | 상태 머신          |
| **설정**        | 강사, 수업/회원권/부가상품 템플릿, 운영 정책, 휴일 | 다중 설정 관리     |
| **플랫폼**      | 헬스장(테넌트) 생성/계약 관리, 플랫폼 메시지       | SUPER_ADMIN 전용   |

### 회원 앱 (PWA)

| 기능            | 설명                                        | 핵심 기술            |
| :-------------- | :------------------------------------------ | :------------------- |
| **수업 예약**   | 주간 스케줄 확인, 예약/취소, 잔여 횟수 표시 | 회원권 차감 트랜잭션 |
| **오늘의 운동** | 당일 운동 메뉴 확인 (HTML 렌더링)           | DOMPurify            |
| **마이페이지**  | 회원권 현황, 예약 내역, 프로필 수정         | JWT 소유권 검증      |
| **푸시 알림**   | 수업 리마인더, 공지사항 실시간 알림         | FCM + Service Worker |

---

## 🔐 인증 및 보안

```
┌─ 로그인 ──────────────────────────────────────────────────────┐
│                                                               │
│  Client  ──POST /login──▶  AuthService                        │
│                              ├─ BCrypt 비밀번호 검증           │
│                              ├─ TokenVersion 확인              │
│                              └─ JWT 발급 ──────────▶ Client    │
│                                                               │
│  JWT Payload: { sub, gymId, role, version }                    │
│  ├─ Access Token  (15분)                                      │
│  └─ Refresh Token (7일)                                       │
│                                                               │
├─ 강제 로그아웃: TokenVersion 증가 → 기존 토큰 즉시 무효화 ────┤
├─ 소유권 검증: JWT email ↔ Booking.member.email 비교 ──────────┤
├─ Rate Limiting: Bucket4j token-bucket (로그인 시도 제한) ─────┤
└─ XSS 방어: DOMPurify 클라이언트 사이드 산화 ─────────────────┘
```

| 보안 요소         | 구현 방식                                                                |
| :---------------- | :----------------------------------------------------------------------- |
| **인증**          | JWT Access + Refresh Token (jjwt 0.12.6)                                 |
| **강제 로그아웃** | TokenVersion 필드 — 비밀번호 변경/관리자 차단 시 기존 토큰 즉시 무효화   |
| **RBAC**          | `SUPER_ADMIN` → 플랫폼 관리, `STAFF` → 헬스장 관리, `MEMBER` → 예약/조회 |
| **소유권 검증**   | 예약 취소 시 JWT 이메일 ↔ 예약자 이메일 비교                             |
| **Rate Limiting** | Bucket4j 토큰 버킷 알고리즘 (로그인, API 호출 제한)                      |
| **XSS 방어**      | DOMPurify로 리치 텍스트 HTML 산화 후 렌더링                              |

---

## 📊 핵심 구현 포인트

### 1. 실시간 활동 피드 (대시보드)

당일 수업에 한해 회원별 최신 예약 상태를 중복 제거하여 표시.  
동일 회원이 **취소 → 재예약**한 경우 `CHANGED` 라벨로 병합 처리합니다.

```java
// 회원별 최신 예약 1건만 추출 (중복 제거)
Map<Long, Booking> latestByMember = bookings.stream()
    .collect(Collectors.toMap(
        b -> b.getMember().getMemberId(),
        b -> b,
        (a, b) -> a.getCreatedAt().isAfter(b.getCreatedAt()) ? a : b
    ));
```

### 2. 회원권 예약 차감 시스템

```
예약 요청 → 수업 정원 확인 → 회원권 잔여 횟수 확인
         → [트랜잭션 시작]
         → 잔여 횟수 -1 (비관적 잠금)
         → Booking 엔티티 생성
         → [트랜잭션 커밋]
         → 취소 시: 잔여 횟수 +1 복구
```

동시 예약 방지를 위해 `@Transactional` + 비관적 잠금(Pessimistic Lock)을 적용합니다.

### 3. 실시간 채팅 (WebSocket STOMP)

```
회원 앱 ──STOMP CONNECT──▶ /ws (JWT 인증)
       ──SUBSCRIBE──────▶ /topic/chat/{roomId}
       ──SEND────────────▶ /app/chat/{roomId}
                              │
                    ChatService: 메시지 저장 (MySQL)
                              │
                    STOMP BROADCAST ──▶ 관리자 웹 (실시간 수신)
```

### 4. 커스텀 DatePicker / WeekPicker

외부 라이브러리 없이 직접 구현한 날짜 선택 컴포넌트.  
월 뷰 ↔ 주 뷰 전환, 계약 기간 범위 선택 등 다양한 컨텍스트에 재사용합니다.

### 5. 멀티테넌트 데이터 격리

```java
// Hibernate @Filter: 모든 엔티티 쿼리에 자동 적용
@FilterDef(name = "gymFilter", parameters = @ParamDef(name = "gymId", type = Long.class))
@Filter(name = "gymFilter", condition = "gym_id = :gymId")
public class Member { ... }

// Service 레이어에서 gymId 자동 주입 (JWT에서 추출)
session.enableFilter("gymFilter").setParameter("gymId", currentGymId);
```

### 6. PWA 푸시 알림 (Firebase FCM)

```
관리자: 공지 작성 → NoticeService → PushSubscription 조회 (gymId 기준)
                                  → Firebase FCM API 호출
                                  → Service Worker가 백그라운드 수신
                                  → 회원 앱에 시스템 알림 표시
```

---

## 📁 프로젝트 구조

```
muscle-spoon/
├── gym-api/                    # Spring Boot 백엔드
│   └── src/main/java/com/musclespoon/gymapi/
│       ├── controller/         # 25개 REST Controller
│       ├── service/            # 비즈니스 로직 (29개 Service)
│       ├── entity/             # 28개 JPA 엔티티
│       ├── repository/         # Spring Data JPA Repository
│       ├── security/           # JWT Filter, SecurityConfig, RBAC
│       ├── dto/                # 요청/응답 DTO
│       ├── config/             # WebSocket, CORS, Flyway 설정
│       └── scheduler/          # 회원 상태 동기화, 수업 리마인더
│
└── gym-web/                    # Next.js 16 프론트엔드
    └── app/
        ├── (admin)/            # 관리자 페이지 (9개 라우트)
        │   ├── dashboard/      # 대시보드
        │   ├── members/        # 회원 관리
        │   ├── lessons/        # 수업 스케줄
        │   ├── pt/             # PT 관리
        │   ├── notices/        # 공지사항
        │   ├── statistics/     # 통계
        │   ├── workout/        # 오늘의 운동
        │   ├── lockers/        # 락커 관리
        │   └── settings/       # 헬스장 설정
        ├── member/             # 회원 앱 (PWA)
        ├── super-admin/        # 플랫폼 관리 (SUPER_ADMIN)
        ├── login/              # 관리자 로그인
        ├── member-login/       # 회원 로그인
        └── components/ui/      # 재사용 UI 컴포넌트 (shadcn/ui)
```

---

## 🔢 프로젝트 규모

| 항목                  | 수치 |
| :-------------------- | :--- |
| REST Controllers      | 25개 |
| API Endpoints         | 100+ |
| JPA Entities (Tables) | 28개 |
| Service Classes       | 29개 |
| Frontend Pages        | 15개 |
| Business Domains      | 14개 |

---

## 📅 개발 기간

**2026.05 ~ 진행 중** · 1인 풀스택 개발

## 🎮 서비스 시연 (Demo)

MuscleSpoon의 주요 기능(관리자 웹)을 직접 체험해 보실 수 있습니다. 아래 버튼을 클릭하여 테스트 계정으로 로그인해 보세요!

[![사이트 방문하기](https://img.shields.io/badge/MuscleMemory_웹사이트_방문하기-38bdf8?style=for-the-badge&logo=googlechrome&logoColor=white)](https://musclememory.kr)

> **💡 테스트 관리자 계정**
>
> - **접속 링크:** [https://musclememory.kr](https://musclememory.kr)
> - **ID:** `test@gym.com`
> - **PW:** `asd123123`

_※ 누구나 접근할 수 있는 테스트 환경이므로, 실제 개인정보나 민감한 데이터 입력은 삼가주시기 바랍니다._
