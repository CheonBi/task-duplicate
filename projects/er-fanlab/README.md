# ER Fan Lab

이터널 리턴 팬메이드 실험체 소개 페이지를 직접 만들어 가기 위한 최소 골격입니다.
공식 프로젝트가 아닙니다.

## 구성

- `backend/`: Spring Boot API
- `frontend/`: React + Vite 페이지

## 실행

Java 17 이상과 Node.js 20.19 이상이 필요합니다.

터미널 1:

```bash
cd backend
./gradlew bootRun
```

터미널 2:

```bash
cd frontend
npm install
npm run dev
```

- 페이지: http://localhost:5173
- API 확인: http://localhost:8080/api/health

프론트엔드에서 `/api`로 시작하는 요청은 Vite가 백엔드로 전달합니다.
화면은 `frontend/src/App.jsx`, 스타일은 `frontend/src/index.css`, API는 `backend/src/main/java/com/erfanlab/`에서 직접 추가하면 됩니다.
