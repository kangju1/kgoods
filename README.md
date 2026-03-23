# Koreal 프로젝트 문서

> 웬즈데이의 소싱 역량을 기반으로 한 K-콘텐츠 B2B 도매 플랫폼

---

## 문서 구조

```
docs/
├── README.md                          ← 이 파일 (문서 인덱스)
│
├── overview/
│   ├── project-overview.md            ← 프로젝트 개요 및 비전
│   └── benchmark-analysis.md          ← 벤치마킹 사이트 분석
│
├── business/
│   ├── process-flow.md                ← 핵심 업무 프로세스 (신보 입수 → 정산)
│   ├── product-categories.md          ← 상품 카테고리 및 특성
│   └── partner-directory.md           ← 거래처 목록 및 코드 체계
│
├── technical/
│   ├── architecture.md                ← 시스템 아키텍처 (Django API + Next)
│   ├── mvp-stack.md                   ← MVP 스택·JWT·API·Next 구조
│   ├── data-model.md                  ← 핵심 데이터 모델
│   └── automation.md                  ← 자동화 대상 및 우선순위
│
└── strategy/
    ├── key-considerations.md          ← 핵심 고려사항 및 리스크
    └── roadmap.md                     ← 개발 로드맵

documentation-guide.md                 ← 문서 구조·관리·PR 습관 (유지보수용)
```

**문서를 어떻게 나누고 언제 고칠지**는 [문서 구조·관리 가이드](./documentation-guide.md)를 본다.

---

## 빠른 참조

| 문서 | 설명 |
|------|------|
| [프로젝트 개요](./overview/project-overview.md) | 프로젝트 배경, 목표, 핵심 가치 |
| [업무 프로세스](./business/process-flow.md) | 신보안내부터 정산까지 전체 흐름 |
| [상품 카테고리](./business/product-categories.md) | 국내반/수입반/MD 등 상품 유형별 특성 |
| [거래처 목록](./business/partner-directory.md) | 매입/매출 거래처 코드 및 정보 |
| [시스템 아키텍처](./technical/architecture.md) | 플랫폼 구성 방향 |
| [MVP 스택·API 설계](./technical/mvp-stack.md) | Django/Next, JWT, 앱 분리, API·가격·장바구니 |
| [데이터 모델](./technical/data-model.md) | 상품, 주문, 거래처 데이터 구조 |
| [자동화 전략](./technical/automation.md) | 자동화 가능 영역 및 우선순위 |
| [핵심 고려사항](./strategy/key-considerations.md) | 리스크, 주의사항, 전략적 제언 |
| [로드맵](./strategy/roadmap.md) | 단계별 개발 계획 |
| [문서 구조·관리 가이드](./documentation-guide.md) | docs 운영, API 정본, ADR, 모노레포 시 확장 |

---

## 프로젝트 핵심 한 줄 요약

> 웬즈데이의 소싱·운영 역량을 바탕으로 K-콘텐츠 B2B 도매 플랫폼을 구축한다. **1차에는 검색으로 유입되는 중·소 잠재 바이어**가 상품을 발견하고 **이메일 인증 가입 후 주문**까지 도달하는 경로를 검증하고, **기존 거래처·내부 8단계 운영**과 병행해 단계적으로 자동화한다.
