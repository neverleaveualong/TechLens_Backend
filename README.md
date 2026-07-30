# TechLens Backend

- 수정일자: 2026-07-31
- 작성자: 심우현
- 멘토: 박효민 선임연구원, 양태인 주임연구원
- 프로젝트: 기업의 특허 검색과 기술 동향 분석을 지원하는 백엔드 API
- 프론트엔드: [TechLens Frontend](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Frontend)
- 공통 기술 문서: [TechLens Docs](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs)
- 데모: [TechLens 서비스](https://techlens-app.vercel.app/login)

![TechLens 서비스 화면](https://raw.githubusercontent.com/Douzone-Keycom-Internship-woohyun-2025/Frontend/main/public/screenshots/02_home.png)

> KIPRIS 특허 데이터를 서비스에 필요한 형태로 가공하고, 인증·검색·분석·사용자 데이터를 일관된 REST API로 제공합니다.

## 목차

- [1. 프로젝트 개요](#1-프로젝트-개요)
- [2. 현황](#2-현황)
- [3. 문제 정의](#3-문제-정의)
- [4. 해결방안](#4-해결방안)
- [5. 기대효과](#5-기대효과)
- [6. 데모](#6-데모)
- [7. 주요 구현 포인트](#7-주요-구현-포인트)
- [8. 실행 환경](#8-실행-환경)
- [9. API 요약](#9-api-요약)
- [10. 프로젝트 구조](#10-프로젝트-구조)
- [11. 검증 기준](#11-검증-기준)
- [12. 상세 기술 문서](#12-상세-기술-문서)
- [13. 라이선스](#13-라이선스)

## 1. 프로젝트 개요

- TechLens Backend는 기업별 특허를 검색하고 기술 분류·출원 추이·등록 상태를 분석하는 서버 애플리케이션입니다.
- KIPRIS Open API의 특허 원천 데이터를 조회하고, XML 응답을 서비스용 JSON 응답으로 변환합니다.
- 회원가입·로그인·로그아웃·토큰 재발급을 제공합니다.
- 기본·고급 특허 검색, 특허 상세 조회, 요약 분석, 기업 비교를 제공합니다.
- 관심 특허와 검색 프리셋을 사용자별로 저장·조회·수정·삭제합니다.

## 2. 현황

- 특허 원천 데이터는 외부 KIPRIS API에서 XML 형태로 제공됩니다.
- 원천 데이터는 필드가 많고 IPC 분류와 특허 상태를 별도로 해석해야 합니다.
- 사용자는 검색 결과만 보는 것이 아니라 기업별 기술 분야와 기간별 변화까지 함께 확인해야 합니다.
- 인증 사용자 데이터와 외부 특허 조회 데이터가 하나의 서비스 흐름으로 연결되어야 합니다.
- 백엔드는 외부 데이터 수집·변환·분석·저장을 프론트엔드와 분리해 제공합니다.

## 3. 문제 정의

- 외부 XML 응답을 프론트엔드가 직접 처리하면 화면과 KIPRIS API의 결합도가 높아집니다.
- IPC 분류와 등록 상태를 화면별로 해석하면 분석 기준과 표시 결과가 달라질 수 있습니다.
- Access Token 만료와 Refresh Token 재발급 흐름이 기능별로 중복되면 인증 동작이 불일치할 수 있습니다.
- 관심 특허와 검색 프리셋은 사용자별 소유권을 보장하지 않으면 데이터가 잘못 노출될 수 있습니다.
- 잘못된 입력, 인증 실패, 외부 API 장애, 검색 결과 없음은 서로 다른 응답과 대응이 필요합니다.
- 검색과 분석 요청이 집중되면 외부 API 호출량과 응답 시간이 증가할 수 있습니다.

## 4. 해결방안

- Route·Middleware·Controller·Service·Repository를 분리해 HTTP 처리와 업무 규칙, 데이터 접근 책임을 나눕니다.
- KIPRIS 연동과 XML 파싱은 Service 계층에서 처리하고 내부 타입과 JSON 응답으로 변환합니다.
- Zod schema로 인증·검색·관심 특허·프리셋 요청을 API 경계에서 검증합니다.
- JWT Access Token과 DB에 저장하는 Refresh Token을 분리해 일반 인증과 재발급을 관리합니다.
- 공통 Error Handler와 도메인 오류 클래스로 입력 오류·인증 오류·리소스 없음·외부 연동 오류를 구분합니다.
- 인증·일반·외부 API 요청에 rate limit을 적용해 요청 폭주와 외부 호출 과다를 제어합니다.
- IPC·기간·등록 상태 분석을 서버에서 계산해 프론트엔드가 같은 기준의 결과를 사용하도록 합니다.

## 5. 기대효과

- 프론트엔드는 외부 XML이 아닌 일관된 JSON API 계약을 사용합니다.
- 외부 데이터 변환과 분석 정책이 서버에 모여 화면 변경의 영향을 줄일 수 있습니다.
- 사용자별 관심 특허와 프리셋의 접근 범위를 인증된 사용자 ID 기준으로 통제할 수 있습니다.
- 입력 검증·오류 처리·요청 제한을 서버 경계에서 수행해 운영 시 예측 가능성을 높입니다.
- 계층별 책임이 분리되어 새로운 API를 추가하거나 기존 기능을 수정할 때 영향 범위를 파악하기 쉽습니다.

## 6. 데모

- [TechLens 서비스 데모](https://techlens-app.vercel.app/login)
- 데모에서 확인할 수 있는 백엔드 연동 흐름
  - 로그인과 Access Token 발급
  - 기본·고급 특허 검색
  - 특허 상세와 IPC 데이터 표시
  - 요약 분석과 기업 비교
  - 관심 특허와 검색 프리셋 관리
- 서버 상태 확인 API: `GET /health`

## 7. 주요 구현 포인트

### 7-1. 기술 스택

| 영역 | 기술 | 역할 |
| :--- | :--- | :--- |
| Runtime | Node.js 20.19.0 | 서버 실행 환경 |
| Framework | Express | REST API와 Middleware 구성 |
| Language | TypeScript | API·서비스·데이터 타입 관리 |
| Database | PostgreSQL | 사용자·토큰·특허·IPC·프리셋·관심 특허 저장 |
| External API | KIPRIS Open API | 특허 원천 데이터 조회 |
| Parsing | `xml2js` | KIPRIS XML 응답 변환 |
| Validation | Zod | 요청 입력 검증 |
| Security | `bcryptjs`, `jsonwebtoken`, `helmet`, `cors` | 비밀번호·인증·HTTP 보안 |
| Traffic Control | `express-rate-limit` | 인증·일반·외부 API 요청 제한 |
| Deployment | Render | 서버 및 데이터베이스 운영 환경 |

### 7-2. 계층별 책임

| 계층 | 책임 | 두지 않는 것 |
| :--- | :--- | :--- |
| Route | URL·HTTP method·Middleware 조합 | 업무 규칙·DB 쿼리 |
| Middleware | 인증·입력 검증·rate limit·공통 오류 | 도메인 응답 조합 |
| Controller | 요청값 추출·Service 호출·HTTP 응답 | XML 파싱·SQL 실행 |
| Service | 업무 규칙·외부 API 조합·분석 계산 | HTTP 객체 직접 처리 |
| Repository | PostgreSQL 조회·저장·수정·삭제 | HTTP 상태 코드·사용자 메시지 |
| Validator·Type | 외부 입력과 내부 데이터 형태 정의 | 데이터 저장 로직 |

### 7-3. 요청 처리 흐름

```text
HTTP 요청
  -> Route
  -> rate limit / 인증 / Zod 검증
  -> Controller
  -> Service
  -> Repository 또는 KIPRIS Open API
  -> 내부 타입 변환
  -> JSON 응답 또는 공통 오류 처리
```

- 특허 검색
  - 검색 조건을 `patentSchemas`에서 검증합니다.
  - Patent Service가 KIPRIS API를 호출하고 XML 응답을 파싱합니다.
  - IPC와 특허 상태를 내부 기준으로 정규화합니다.
  - 검색 결과와 페이지 정보를 JSON으로 반환합니다.
- 요약 분석·기업 비교
  - 기업과 기간 조건을 기준으로 특허 데이터를 수집합니다.
  - IPC 분포·월별 출원 추이·등록 상태를 서버에서 계산합니다.
  - 비교 요청은 기업별 분석 결과를 구분해 반환합니다.
- 관심 특허·프리셋
  - 인증 Middleware에서 확인한 사용자 ID를 기준으로 처리합니다.
  - Service가 사용자 소유권을 확인한 뒤 Repository를 호출합니다.
  - 생성·수정·삭제 결과를 프론트엔드의 cache 갱신에 사용할 수 있도록 반환합니다.

### 7-4. 외부 특허 데이터 처리

- Axios로 KIPRIS Open API를 호출합니다.
- `xml2js`로 XML 응답을 파싱합니다.
- XML의 단일 값과 배열 값 차이를 내부 변환 단계에서 정리합니다.
- 외부 원본 필드를 프론트엔드에 그대로 노출하지 않고 서비스 응답 모델로 변환합니다.
- 외부 API 오류·지연·호출 제한을 내부 서버 오류와 구분합니다.
- API 키와 원본 외부 응답은 로그와 클라이언트 응답에 노출하지 않습니다.

### 7-5. 인증·보안·오류 처리

- `bcryptjs`로 비밀번호를 해시하고 `jsonwebtoken`으로 Access·Refresh Token을 발급합니다.
- Refresh Token은 Repository에서 저장·조회·삭제하며 재발급과 로그아웃 시 상태를 갱신합니다.
- 관심 특허와 프리셋은 인증된 사용자 ID 기준으로 접근 범위를 제한합니다.
- Zod schema가 Controller 이전에 잘못된 입력을 거절합니다.
- 도메인 오류 클래스와 공통 Error Handler가 400·401·404·429·500 응답을 구분합니다.
- `helmet`, `cors`, `express-rate-limit`으로 HTTP 보안과 요청량을 관리합니다.

## 8. 실행 환경

- Node.js: `20.19.0`
- Database: PostgreSQL
- External API: KIPRIS Open API
- Deployment: Render

### 8-1. 환경변수

```bash
DATABASE_URL=
PORT=4000
JWT_SECRET=
KIPRIS_API_KEY=
KIPRIS_BASE_URL=
FRONTEND_URL_DEV=
FRONTEND_URL_PROD=
FRONTEND_URL_VERCEL=
FRONTEND_URL_STAGING=
```

- 실제 값은 `.env` 또는 배포 플랫폼의 환경변수로만 관리합니다.
- 토큰·비밀번호·API 키는 README와 Git에 기록하지 않습니다.
- 운영 DB 스키마 변경과 마이그레이션은 별도 검토 후 진행합니다.

### 8-2. 로컬 실행

```bash
npm ci
npm run dev
```

- 서버 기본 주소: `http://localhost:4000`
- 상태 확인: `GET http://localhost:4000/health`
- 운영 실행: `npm run build && npm start`

## 9. API 요약

| 영역 | 주요 Endpoint |
| :--- | :--- |
| 인증 | `POST /users/signup`, `/users/login`, `/users/refresh`, `/users/logout` |
| 특허 | `POST /patents/search/basic`, `POST /patents/search/advanced`, `GET /patents/:applicationNumber` |
| 분석 | `GET /summary`, `GET /summary/compare` |
| 프리셋 | `POST·GET /presets`, `GET·PATCH·DELETE /presets/:presetId` |
| 관심 특허 | `GET·POST /favorites`, `GET·PATCH·DELETE /favorites/:applicationNumber` |
| 상태 확인 | `GET /health` |

- 정확한 요청·응답 필드와 오류 형식은 [TechLens API 명세서](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs/blob/main/specs/TechLens_API%EB%AA%85%EC%84%B8%EC%84%9CV1.1.md)를 기준으로 합니다.
- 테이블·컬럼·관계는 [TechLens DB 정의서](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs/blob/main/specs/TechLens_DB%EC%A0%95%EC%9D%98%EC%84%9CV1.1.md)를 기준으로 합니다.

## 10. 프로젝트 구조

```text
src/
├── config/              # 환경변수와 PostgreSQL 연결
├── controllers/         # HTTP 요청·응답 처리
├── errors/              # 도메인 오류 타입
├── middlewares/         # 인증·검증·제한·공통 오류
├── repositories/        # PostgreSQL 접근
├── routes/              # 기능별 API 라우팅
├── services/            # 업무 규칙과 외부 API 조합
├── types/               # 도메인 타입
├── utils/               # IPC 등 공통 변환
├── validators/          # Zod 요청 schema
├── app.ts               # Express 앱과 Route 등록
└── server.ts            # 서버 기동과 종료 처리
```

## 11. 검증 기준

- 빌드:

  ```bash
  npm run build
  ```

- 테스트:

  ```bash
  npm test
  ```

  - 현재 테스트 스크립트는 연결 상태를 확인하는 단계이며, 자동화 테스트 보강이 필요합니다.
- 자동화 테스트 보강 전 수동 확인 범위
  - 회원가입·로그인·로그아웃·토큰 재발급
  - 기본·고급 특허 검색과 KIPRIS 오류
  - 요약 분석·기업 비교와 데이터 없음
  - 관심 특허·메모·프리셋의 사용자별 접근
  - 400·401·404·429·500 오류 응답
- 변경 후 확인할 계약
  - Backend API 응답과 Frontend 타입
  - API 명세서와 실제 Route
  - DB 정의서와 Repository 쿼리
  - 환경변수 이름과 배포 설정

## 12. 상세 기술 문서

- [TechLens API 명세서](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs/blob/main/specs/TechLens_API%EB%AA%85%EC%84%B8%EC%84%9CV1.1.md)
- [TechLens DB 정의서·ERD](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs)
- [TechLens 시스템 아키텍처·실행 환경](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Docs)
- [Frontend 저장소](https://github.com/Douzone-Keycom-Internship-woohyun-2025/Frontend)

## 13. 라이선스

- 본 프로젝트의 코드와 문서는 심우현의 포트폴리오 및 기술 검토 목적으로 관리합니다.
- 기업 협업 산출물의 권리와 사용 범위는 별도 협의와 원본 계약을 우선합니다.

<p align="center">
  <img width="180" alt="KICOM 로고" src="https://github.com/user-attachments/assets/3e8b41ac-733c-499a-b49b-bf32eee18ad8" />
</p>
