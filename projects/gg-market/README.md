# GG마켓

기본적인 e-commerce task들을 duplicate 하면서
요구사항을 정의하는 task 부터 개발 task, 테스트, 배포까지 프로세스를 연구한다.

## 소개

- 만들려는 서비스: E-commerce
- 해결하려는 사용자 문제: 지금 구매하지 않을 상품을 저장해 나중에 다시 찾고 싶다.
- 참고 서비스: 네이버쇼핑, 쿠팡, Gmarket

## 기능 및 진행 상태

| 기능 | 요구사항 문서                         | 진행 상태 |
| ---- | ------------------------------------- | --------- |
| 찜   | [찜 기능 요구사항](docs/favorites.md) | 예정      |

<!-- 상태는 예정 / 진행 중 / 완료 등으로 표시한다. -->

## 설계 기록

- [Spring Boot 백엔드 구조 비교](docs/backend-architecture-research.md)

## 실행 방법

`compose.yaml`은 GG마켓의 로컬 개발 환경을 실행한다. 백엔드 이미지를 만들 때 Dockerfile은 `backend/Dockerfile`에 둔다.

### 준비 사항

- Docker Engine과 Docker Compose
- `gg-market/.env.example`을 `.env`로 복사하고 `GGMARKET_DB_PASSWORD`에 로컬 개발용 비밀번호 설정

### 실행 순서

저장소 루트에서 실행한다.

```bash
cd projects/gg-market
docker compose up -d db
docker compose ps
docker compose exec db psql -U ggmarket -d ggmarket -c 'SELECT 1;'
```

WSL에서 실행하는 백엔드는 `localhost:5433`으로 DB에 접속한다. 나중에 백엔드도 같은 Compose 구성에서 실행하면 DB 주소는 `db:5432`를 사용한다.

`docker compose down`은 컨테이너를 중지하고 데이터 볼륨은 남긴다. `docker compose down -v`는 DB 데이터 볼륨도 삭제한다.
