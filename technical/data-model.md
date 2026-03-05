# 핵심 데이터 모델

## 1. 엔티티 관계 개요

```
Partner (거래처)
    ├── PartnerPriceTier (공급가 그룹)
    └── Order (주문)
            └── OrderItem (주문 항목)

Product (상품)
    ├── ProductCategory (카테고리)
    ├── Artist (아티스트)
    ├── ProductVariant (버전/옵션)
    ├── ProductStock (재고)
    └── ProductPrice (거래처별 가격)

NewRelease (신보 안내)
    └── NewReleaseSource (원본 이메일/파일)
```

---

## 2. 주요 엔티티 상세

### Product (상품)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | 상품 고유 ID |
| name | string | 상품명 |
| name_en | string | 영문명 (글로벌 대응) |
| name_ja | string | 일문명 (일본 수입 상품) |
| category_id | FK | 카테고리 참조 |
| artist_id | FK | 아티스트 참조 (nullable) |
| jan_code | string | JAN/EAN 바코드 |
| item_code | string | 공급사 품번 (nullable) |
| isbn | string | ISBN (도서/잡지) |
| list_price | decimal | 정가 |
| release_date | date | 발매일 |
| order_deadline | date | 예약 주문 마감일 (nullable) |
| status | enum | NEW_RELEASE / IN_STOCK / OUT_OF_STOCK |
| product_type | enum | ALBUM_KR / ALBUM_JP / BOOK_KR / BOOK_JP / MD |
| sale_condition | string | 판매 조건 (랜덤/세트 조건 등) |
| distribution_limit | string | 유통 제한 조건 (nullable) |
| is_exclusive | boolean | 단독 유통 여부 |
| source_country | enum | KR / JP |
| is_adult | boolean | 성인 콘텐츠 여부 |
| created_at | timestamp | |
| updated_at | timestamp | |

### ProductVariant (상품 버전/옵션)

음반의 버전(Ver. A, B, Digipack 등), 굿즈의 색상/사이즈 등

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| product_id | FK | 상품 참조 |
| variant_name | string | 버전명 (예: "Digipack Ver. A") |
| jan_code | string | 버전별 JAN 코드 |
| item_code | string | 버전별 품번 |
| stock_qty | integer | 재고 수량 |

### ProductPrice (거래처별 가격)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| product_id | FK | 상품 참조 |
| price_tier_id | FK | 가격 그룹 참조 |
| supply_price | decimal | 이 그룹에 적용되는 공급가 |
| supply_rate | decimal | 공급율 (도서류용, nullable) |

### Partner (거래처)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| code | string | 거래처 코드 (예: SLC) |
| name | string | 거래처 정식명 |
| business_no | string | 사업자등록번호 |
| contact_email | string | 담당자 이메일 |
| contact_phone | string | 담당자 연락처 |
| partner_type | enum | BUYER / SUPPLIER / BOTH |
| payment_type | enum | PREPAID / POSTPAID |
| price_tier_id | FK | 적용 가격 그룹 |
| preferred_categories | array | 관심 카테고리 (알림 필터용) |
| status | enum | ACTIVE / INACTIVE / PENDING |

### Order (주문)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| order_no | string | 주문번호 (수주서 번호) |
| partner_id | FK | 거래처 참조 |
| status | enum | RECEIVED / CONFIRMED / PURCHASING / SHIPPED / SETTLED |
| order_type | enum | PRE_ORDER / IN_STOCK |
| payment_status | enum | PENDING / PAID / INVOICED |
| total_amount | decimal | 총 주문금액 |
| notes | text | 비고 |
| created_at | timestamp | 주문 접수일 |
| confirmed_at | timestamp | 주문 확정일 |
| shipped_at | timestamp | 출고일 |

### OrderItem (주문 항목)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| order_id | FK | 주문 참조 |
| product_id | FK | 상품 참조 |
| variant_id | FK | 버전 참조 (nullable) |
| quantity | integer | 주문 수량 |
| unit_price | decimal | 적용된 공급가 |
| subtotal | decimal | 소계 |

### NewRelease (신보 안내)

이메일에서 파싱된 신보 안내 원본을 추적하기 위한 엔티티

| 필드 | 타입 | 설명 |
|------|------|------|
| id | UUID | |
| source_type | enum | EMAIL / KAKAO / MANUAL |
| source_subject | string | 이메일 제목 |
| source_sender | string | 발신자 |
| received_at | timestamp | 수신 시각 |
| raw_content | text | 원문 (파싱 전) |
| parsed_data | JSON | AI 파싱 결과 |
| review_status | enum | PENDING / REVIEWED / REGISTERED |
| product_ids | array | 등록된 상품 ID 목록 |

---

## 3. 코드 체계

### 상품 상태 (status)
| 값 | 설명 |
|----|------|
| `NEW_RELEASE` | 신보 (예약 주문 가능) |
| `IN_STOCK` | 구보 (재고 있음, 즉시 출고) |
| `OUT_OF_STOCK` | 품절 |
| `DISCONTINUED` | 단종 |

### 주문 상태 (status)
| 값 | 설명 |
|----|------|
| `RECEIVED` | 주문 접수 |
| `CONFIRMED` | 주문 확정 (수주서 발행) |
| `PURCHASING` | 발주 완료, 매입 진행 중 |
| `IN_WAREHOUSE` | 창고 입고 완료 |
| `SHIPPED` | 출고 완료 |
| `SETTLED` | 정산 완료 |

---

## 4. 현재 엑셀 데이터 구조와의 매핑

손정욱 대표가 관리하는 엑셀(입출고현황, 상품 DB)의 주요 필드와 플랫폼 데이터 모델 간 매핑:

| 엑셀 필드 | 플랫폼 필드 | 비고 |
|-----------|-----------|------|
| 상품코드 (임의 코드) | `item_code` | |
| JAN 코드 | `jan_code` | |
| 원가 | 별도 매입가 테이블 | |
| 발행일 | `release_date` | |
| 구매처 코드 | `partner.code` (supplier) | |
| 공급율 | `product_price.supply_rate` | 도서류만 해당 |
| 출고기준가 | `product_price.supply_price` | 음반 전용 환산가 |
| 주문현황 (VLOOKUP) | `order` → `order_item` | ERP 연동 또는 플랫폼 자체 관리 |
