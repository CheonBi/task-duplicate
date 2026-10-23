# Spring MVC와 백엔드 패키지 구성

- 문서 상태: 설계 초안
- 작성일: 2026-10-22
- 대상: GG마켓 백엔드

## MVC와 디렉토리 구성은 다른 문제

Spring MVC는 `DispatcherServlet`을 중심으로 HTTP 요청을 처리한다. `@RestController`는 반환값을 응답 본문에 쓰므로, GG마켓의 JSON API에는 서버에서 렌더링하는 화면 템플릿이 필요하지 않다. `controller`, `service`, `repository`라는 디렉토리 구분은 MVC가 강제하는 구조가 아니라 코드의 책임을 나누는 선택이다. [Spring MVC 요청 처리](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html), [`@ResponseBody`와 `@RestController`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responsebody.html)

따라서 MVC의 이름을 그대로 `model/`, `view/`, `controller/` 디렉토리로 옮기지 않는다. 특히 API 응답 객체와 DB 엔티티를 모두 `model/`에 모으는 방식은 각각의 책임을 흐릴 수 있다.

Spring Boot도 특정 패키지 구성을 요구하지 않는다. 다만 `@SpringBootApplication`이 있는 클래스를 하위 패키지들의 상위에 두면 컴포넌트와 JPA 엔티티를 찾는 기준이 명확해진다. 현재 `GgmarketApplication`은 `task.duplicate.ggmarket`에 있으므로 이 패키지를 루트로 유지한다. [Spring Boot 코드 구성](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)

## 패키지를 나누는 두 가지 방법

| 방법 | 예시 | 작업할 때의 차이 |
| --- | --- | --- |
| 역할별 | `controller/`, `service/`, `repository/` | 같은 역할의 파일을 찾기 쉽지만, 찜 기능 하나를 수정할 때 여러 패키지를 오간다. |
| 기능별 | `favorite/` 안에 컨트롤러·서비스·저장소를 함께 둔다. | 찜 기능에 관한 파일을 한곳에서 다룰 수 있다. 기능이 커지면 그 안에서 역할별로 다시 나눌 수 있다. |

## GG마켓의 초기 선택

찜 기능은 `task.duplicate.ggmarket.favorite`에 모은다. `GgmarketApplication`은 루트 패키지에 그대로 둔다. 아래 흐름은 구현 방향이며, 아직 클래스가 만들어졌다는 뜻은 아니다.

```text
HTTP 요청 → FavoriteController → FavoriteService → FavoriteRepository → PostgreSQL
HTTP 응답 ← JSON 응답         ← 찜 상태          ← 저장 결과
```

- 컨트롤러: HTTP 경로·입력·응답을 다룬다.
- 서비스: 저장 여부와 중복 요청 같은 찜 규칙을 다룬다.
- 저장소: 찜 상태를 DB에서 읽고 쓴다.

필요한 클래스가 생길 때 패키지에 추가한다. 여러 기능이 실제로 같은 코드를 사용할 때까지 `common` 같은 공용 패키지는 만들지 않는다.

## 다시 검토할 기준

- 찜 이외의 기능이 추가되었을 때, 한 기능의 변경 파일이 여러 최상위 패키지에 흩어지는가?
- 기능 사이에 서비스나 엔티티 참조가 늘어 순환 의존이 생기는가?
- 회사 프로젝트에 적용할 때는 기존 팀의 패키지 규칙과 변경 비용을 먼저 확인했는가?

회사 프로젝트에서는 최근 변경 사례를 보고 실제로 함께 수정되는 파일과 팀의 소유 범위를 확인한다. 기존 역할별 구조에서 불편이 반복되지 않는다면 유지하고, 한 기능의 변경이 계속 흩어지거나 순환 의존이 생긴다면 기능별 묶음을 제안한다. GG마켓의 구조를 바꾸게 되면 이유와 영향을 이 문서에 기록한다.
