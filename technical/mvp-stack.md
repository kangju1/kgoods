# MVP 스택·API 설계

확정 사항: Django REST API 전용, Next.js(바이어+어드민), 도메인별 Django app, `AbstractUser` 기반 User, JWT, **이메일 파싱 없음·수동 카탈로그**.

아래는 **비즈니스 목표**(검색 유입 → 상세 → 이메일 인증 가입 → 주문 → 어드민 확인, 견적 이후 수기)를 달성하기 위한 **권장 설계**다.

---

## 1. Django 프로젝트·앱 구성

| App | 책임 |
|-----|------|
| **`accounts`** | 커스텀 `User(AbstractUser)`, 이메일 인증 토큰/플로, (선택) 프로필 확장 모델 |
| **`catalog`** | `Product`, `ProductVariant`, `ProductImage`, `ProductCategory`, `Artist`, 상품 공개·SEO 필드 |
| **`partners`** | `Partner`, `PartnerPriceTier`, `ProductPrice` |
| **`orders`** | `Cart`, `CartItem`, `Order`, `OrderItem` |
| **`core`** (선택) | 공통 pagination, 예외 핸들러, permissions 베이스 |

원칙: **순환 참조 방지** — 예: `orders`가 `catalog`·`accounts`를 참조, 역은 최소화.

---

## 2. 사용자·거래처·가격 정책 (권장)

### 2-1. User

- 로그인 식별자: **이메일** (`USERNAME_FIELD = 'email'`)
- `AbstractUser` 필드 + MVP 확장 예: `phone` (optional), `company_name` (optional), `email_verified_at` (datetime, null)
- **`partner`**: FK → `Partner`, **null=True**. 잠재 바이어는 null, 기존 거래처 매핑 시 설정
- **어드민 접근**: Django `is_staff` (또는 별도 role 모델은 후속). Next 어드민은 **스태프 전용 API**만 호출

### 2-2. 신규 가입자 가격 티어

- `User`에 `price_tier` FK를 두거나, **시스템 기본 티어**(예: `WEB_DEFAULT`)를 신규에 자동 부여
- 로그인 후 공개가: 해당 티어의 `ProductPrice`, 없으면 **정가만 표시** 또는 **"견적 문의"** (비즈니스 선택)

### 2-3. 비로그인(검색 유입) 노출

- **상품명·발매일·이미지·설명·SKU/JAN(선택)** 공개
- **도매 단가**: 기본은 **비노출** 또는 **정가(`list_price`)만** 노출 — 유출·채널 충돌 방지
- 로그인 후: 티어 기준 공급가 표시

---

## 3. 장바구니 (권장)

**목표**: 가입 전 탐색은 허용하되, **주문은 인증된 사용자만** (문서상 여정과 일치).

| 방식 | 내용 |
|------|------|
| **권장 (MVP)** | 서버 `Cart`/`CartItem`에 **`user`** FK. 장바구니 API는 **인증 필수** — 구현 단순, 스팸·악용 감소 |
| **대안** | 비회원 `session_key` 또는 쿠키 UUID로 장바구니 → 로그인 시 **병합** (퍼널은 유리, 구현 다소 증가) |

첫 릴리스는 **인증 필수 장바구니**를 권장하고, 전환율 이슈가 보이면 병합형 비회원 카트를 추가한다.

---

## 4. 주문·결제 (MVP)

- `Order`: **`user`** FK 필수, **`partner`** FK는 null 허용(잠재 바이어)
- `payment_status`: PG 없을 때 **`QUOTE_PENDING`** / **`NOT_APPLICABLE`** 등 도메인 값으로 정의해 혼동 방지
- `status`: MVP는 **`RECEIVED` → `CONFIRMED`(운영 확정)** 정도만 써도 됨; 나머지는 후속
- 주문 확정 후 실제 결제·세금계산서는 **수기** — 플랫폼은 **접수·이력**에 집중

---

## 5. JWT 운영 (권장)

- 라이브러리: **`djangorestframework-simplejwt`** (일반적 선택)
- **액세스**: 짧은 TTL(예: 15~60분), **리프레시**: 길게(예: 7~14일), 가능하면 **rotation** 활성화
- **클라이언트(Next)**:
  - **이상적**: 리프레시 토큰을 **HttpOnly·Secure·SameSite** 쿠키로만 보관하기 위해 **Next Route Handler(BFF)** 가 로그인/리프레시를 프록시하고 쿠키 설정(백엔드와 쿠키 도메인 협의 필요)
  - **MVP 단순안**: 액세스·리프레시를 JSON으로 받아 **메모리+짧은 액세스** 또는 클라이언트 저장 — **XSS 대비** CSP·입력 검증·HTTPS 필수; 이후 쿠키 방식으로 강화
- **어드민**: 동일 JWT라도 **`is_staff`**인 경우만 ` /api/v1/admin/...` 접근 허용

---

## 6. API 네이밍·버전

- 접두사: **`/api/v1/`**
- 권장 그룹:
  - **`/api/v1/auth/`** — 회원가입, 로그인, 토큰 리프레시, 이메일 인증, 비밀번호 재설정(후속)
  - **`/api/v1/catalog/`** — 공개 카테고리·상품 목록·상세(slug)
  - **`/api/v1/me/`** — 프로필, 내 주문 목록
  - **`/api/v1/cart/`**, **`/api/v1/orders/`** — 인증 필요
  - **`/api/v1/admin/`** — 스태프 전용: 상품 CRUD, 마스터, 주문 목록·상태 변경, Partner·티어

공개 상품 API는 **인증 없이** 가능하되, **가격 필드는 로그인 여부·티어에 따라 직렬화 분기** (DRF serializer context).

---

## 7. Next.js 구조 (권장)

단일 Next 앱 내 **라우트 그룹**으로 권한·레이아웃 분리:

| 그룹 | 예시 경로 | 비고 |
|------|-----------|------|
| 바이어 (공개) | `app/(shop)/`, `products/[slug]` | 메타·JSON-LD로 SEO |
| 바이어 (인증) | `app/(shop)/cart`, `checkout`, `account` | JWT 포함 요청 |
| 어드민 | `app/(admin)/admin/...` | `is_staff` 가드(미들웨어에서 `/api/me` 또는 JWT claim 확인) |

- **상품 상세**: `fetch` API URL은 빌드 타임/ISR용 **공개 엔드포인트** 사용; 가격은 클라이언트에서 로그인 시 추가 요청하거나, 상세 API가 쿠키/헤더에 따라 다른 JSON 반환
- **캐시 무효화**: 상품 publish 시 Django → **revalidate webhook** URL 호출(Next) 권장

---

## 8. 이메일 인증 플로

1. 가입 시 `User` 생성, `is_active=False` **또는** `email_verified_at` null + 주문 API에서 검증 강제  
2. 서명된 uid/token 링크로 `GET` 또는 `POST` 확인 엔드포인트  
3. 검증 완료 후 로그인·주문 허용  

트랜잭션 메일: Django `send_mail` + SMTP(또는 SendGrid 등) — MVP 최소 구성.

---

## 9. 보안·품질 체크리스트 (MVP)

- CORS: Next 오리진만 허용
- CSRF: **JWT Bearer 사용 시** 브라우저가 API에 직접 호출하는 패턴이면 CSRF 부담 감소(쿠키 기반 세션과 혼용하지 않기)
- Rate limit: 가입·로그인·주문 생성 (django-ratelimit 등)
- 파일 업로드: 어드민 이미지 **용량·타입** 제한, 스토리지 비공개 버킷 + 서명 URL

---

## 10. 배포 (권장 방향)

- API: `api.example.com`, 웹: `www.example.com` (또는 동일 도메인 `/api` 리버스 프록시)
- DB: 관리형 PostgreSQL
- 시크릿: 환경 변수·시크릿 매니저

이 문서는 구현이 진행되면 **엔드포인트 표·OpenAPI 링크**로 확장하면 된다.
