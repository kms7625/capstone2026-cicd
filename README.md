# capstone2026-cicd — Spring Boot + React CI/CD 실습

GitHub Actions로 CI(빌드 검증)와 CD(Docker 이미지 빌드·Docker Hub 푸시) 파이프라인을 구성해 본 실습 프로젝트입니다.

## 구성

| 서비스 | 내용 | 포트 |
|---|---|---|
| `backend/` | Spring Boot(Java 17, Maven) · JPA · MariaDB — 회원가입·로그인 API(`/api/members/register`, `/api/members/login`) | 8080 |
| `frontend/` | React(Create React App) → Nginx 정적 서빙(멀티 스테이지 Dockerfile) | 63342 → 80 |
| `db` | MariaDB(docker-compose) | 3326 → 3306 |

## 파이프라인

| 워크플로 | 트리거 | 단계 |
|---|---|---|
| `ci.yml` | `main`·`develop` push / PR | 백엔드 `./mvnw clean package`(테스트 생략) · 프론트엔드 `npm install && npm run build` |
| `cd.yml` | `main` push | Maven 빌드 → 백엔드·프론트엔드 Docker 이미지 빌드(`커밋 SHA`·`latest` 태그) → Docker Hub 푸시 |

Docker Hub 인증은 저장소 Secrets(`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`)를 사용합니다.
작업은 `feature/*` → `develop` → `main` PR 흐름으로 진행했습니다(PR #1~#5).

## 실행

```bash
cd backend && ./mvnw clean package -DskipTests && cd ..
docker compose up --build
# frontend http://localhost:63342 · backend http://localhost:8080
```

## 한계 (실습용)

- 비밀번호를 평문으로 저장·비교합니다(해시 미적용).
- DB 접속 정보가 `docker-compose.yml`·`application.yaml`에 직접 들어 있습니다.
- CI에서 테스트를 생략(`-DskipTests`)합니다.
