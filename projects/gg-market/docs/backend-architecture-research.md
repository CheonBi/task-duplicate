# Spring Boot 백엔드 구조 비교

- 작성일: 2026-10-23
- 범위: MVC와 관련 UI 패턴, 계층형·기능별·헥사고날·클린·모듈형 백엔드 구조
- 목적: GG마켓을 구현하면서 구조를 선택하고, 회사 프로젝트의 패키지 구성도 판단할 기준을 만든다.

## 먼저 내린 결론

Spring Boot로 만드는 일반적인 업무용 웹 애플리케이션에서는 **HTTP 입구에 컨트롤러를 두고, 업무 규칙과 데이터 접근을 분리하는 계층형 구성**이 익숙한 출발점이다. [Fowler는 프레젠테이션·도메인·데이터 접근의 분리를 널리 쓰이는 구성](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)으로 설명한다. [Spring Boot 공식 문서의 전형적인 예시](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)는 `customer`, `order`처럼 기능별 패키지를 먼저 만들고, 그 안에 Controller·Service·Repository를 둔다. 따라서 **기능별로 묶고 내부에서 필요한 책임만 나누는 방식**이 GG마켓의 합리적인 첫 선택이다.

여기서 말하는 '대중적'은 공식 문서의 예시와 널리 인용되는 실무 문헌에서 확인되는 **관행**이라는 뜻이다. 구조별 채택률을 정확한 숫자로 비교할 만한 대표성 있는 Spring Boot 조사는 확인하지 못했다. 헥사고날·클린·수직 슬라이스 중 무엇이 '몇 위'라고 단정하지 않는다.

## 비교할 때 보는 기준

구조의 이름보다 다음 네 가지를 먼저 본다.

1. 기능 하나를 바꿀 때 함께 수정하는 코드가 어디에 모이는가?
2. 업무 규칙이 HTTP·DB 코드 없이 이해되고 검증될 필요가 있는가?
3. 다른 기능이나 외부 시스템으로 향하는 의존을 어디에서 통제하는가?
4. 그 통제로 얻는 이익이 새 인터페이스·객체 변환·패키지 수의 비용보다 큰가?

MVC·MVP·MVVM은 주로 **사용자 입력과 화면의 관계**를 다룬다. 계층형·헥사고날·클린은 **업무 규칙과 외부 기술의 관계**를, 기능별 구성·수직 슬라이스는 **변경할 코드를 묶는 단위**를 다룬다. 서로 하나만 고르는 목록이 아니므로, 한 애플리케이션에서 기능별 패키지와 계층형을 함께 쓸 수 있다.

## 1. MVC와 가까운 UI 구조

**MVC(Model–View–Controller)**는 사용자의 입력을 받는 Controller, 화면을 표현하는 View, 상태와 규칙을 가진 Model의 책임을 나눈다. 원래 UI를 다루기 위해 만들어졌고 여러 UI 프레임워크에 영향을 주었다. 화면 처리와 업무 규칙이 뒤섞이는 것을 줄여 각각을 변경하고 검증하기 쉬워진다. 다만 `model/`, `view/`, `controller/`를 프로젝트의 최상위 디렉터리로 만들어야 한다는 규칙은 아니다. [Fowler, Model View Controller](https://martinfowler.com/eaaCatalog/modelViewController.html), [Microsoft, MVC 구성 요소와 책임](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview)

서버가 HTML을 렌더링하는 앱에서는 View가 서버 코드에 보인다. GG마켓처럼 프런트엔드가 별도이고 백엔드가 JSON API를 제공한다면, 백엔드에는 주로 HTTP Controller와 업무·데이터 코드가 있다. [Spring의 REST 예시](https://spring.io/guides/gs/rest-service/)도 `@RestController`가 객체를 JSON 응답으로 쓰는 형태를 보여준다. 이 사실이 MVC의 가치나 백엔드 패키지 선택을 자동으로 결정하지는 않는다.

**MVP(Model–View–Presenter)**는 Presenter가 입력 처리와 View 갱신을 더 많이 맡는다. View를 수동적으로 두면 화면 없이 Presenter를 검증하기 쉬워지지만, 화면과 Presenter 사이의 연결 코드가 늘어난다. **MVVM(Model–View–ViewModel)**은 View의 표시 상태와 동작을 ViewModel에 두고 데이터 바인딩으로 동기화한다. 바인딩 기능이 강한 클라이언트 UI에서 유용하지만, 상태 동기화 자체가 복잡해질 수 있다. 둘 다 GG마켓의 **Spring Boot 서버 패키지 구조를 고르는 직접적인 대안은 아니다.** [Fowler, Passive View/MVP 계열](https://martinfowler.com/eaaDev/PassiveScreen.html), [Fowler, Presentation Model/MVVM](https://martinfowler.com/eaaDev/PresentationModel.html)

## 2. 계층형 구조: Controller → Service → Repository

HTTP 처리, 업무 수행, 데이터 접근을 나눈다. Controller는 요청·응답과 입력 형식을, Service는 업무 작업과 트랜잭션 경계를, Repository는 저장·조회 방법을 맡는 구성이 흔하다. 업무 규칙을 HTTP나 SQL 세부사항과 분리해 읽고 검증하기 쉬운 것이 채택 이유다. [Fowler, Presentation Domain Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html), [Fowler, Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)

문제는 `controller/`, `service/`, `repository/`를 **전체 프로젝트의 최상위 패키지**로만 만들면 기능 하나의 변경이 여러 디렉터리에 흩어지기 쉽다는 점이다. 또한 모든 요청에 Service를 기계적으로 추가하면 단순 전달 메서드만 쌓인다. 계층은 필요한 책임을 드러내기 위한 도구다. 반대로 Service 한 클래스가 여러 기능의 규칙을 독점하면 계층을 나눈 이점도 줄어든다. Fowler도 큰 시스템에서는 최상위 모듈을 도메인별로 나누고 그 안에서 계층화하는 편을 권한다. [Fowler, Presentation Domain Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)

## 3. 기능별 패키지와 수직 슬라이스

**기능별 패키지(package by feature)**는 `favorite/`, `order/`처럼 업무 영역을 먼저 나누고 관련 Controller·Service·Repository를 가까이 둔다. 기존 계층의 책임은 유지할 수 있다. Spring Boot 공식 문서의 `customer/`, `order/` 예시가 바로 이 형태다. 애플리케이션 시작 클래스를 이 패키지들의 상위에 두는 것도 공식 권장 사항이다. [Spring Boot, Structuring Your Code](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)

**수직 슬라이스(vertical slice)**는 여기서 더 나아가 `favorite/add/`, `favorite/list/`처럼 **요청 또는 사용 사례**마다 입력·처리·출력을 묶는다. 기능 하나의 변경 범위를 좁히고, 단순 조회와 복잡한 명령에 같은 계층 구조를 강요하지 않는 것이 장점이다. 대신 슬라이스 간 중복을 어느 정도 허용하고, 공통 규칙을 언제 추출할지 팀이 판단해야 한다. [Jimmy Bogard, Vertical Slice Architecture](https://www.jimmybogard.com/vertical-slice-architecture/)

두 방식은 같은 말이 아니다. `favorite/` 안에 한 Service가 찜 추가·삭제·목록을 모두 처리하면 기능별 패키지다. 각 요청의 코드를 독립적으로 묶고 의존도 최소화하면 수직 슬라이스에 가깝다. GG마켓은 첫 기능부터 요청별 패키지를 만들 필요는 없지만, 찜 기능이 커져 한 Service에서 독립적인 작업들이 얽히면 검토할 수 있다.

## 4. 헥사고날과 클린 아키텍처

**헥사고날(Ports and Adapters)**은 핵심 업무 코드가 HTTP·DB·메시징 같은 외부 기술을 직접 알지 않도록 경계를 둔다. 바깥의 Controller나 DB 구현은 어댑터이고, 안팎의 약속은 포트다. 같은 업무를 HTTP뿐 아니라 배치에서도 호출하거나, 저장소·외부 연동을 격리해 검증해야 할 때 효과가 있다. 포트와 어댑터, 데이터 변환이 추가되는 만큼 단순한 CRUD에는 비용이 더 클 수 있다. [Alistair Cockburn, 원본 Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture)

**클린 아키텍처**도 업무 규칙을 안쪽에 두지만, 핵심 규칙·사용 사례·인터페이스 어댑터·프레임워크 등 동심원 계층과 **소스 코드 의존은 안쪽으로 향해야 한다**는 규칙을 강조한다. 업무 모델이 복잡하고 프레임워크와 독립적으로 검증할 가치가 클 때 검토할 만하다. 이 경계를 지키려면 인터페이스와 변환 코드가 늘어난다. 헥사고날과 목표가 겹치지만 도식과 강조점은 다르다. [Robert C. Martin, The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

`Repository`를 인터페이스로 선언했다는 사실만으로 헥사고날이나 클린 구조가 되지는 않는다. **누가 그 인터페이스를 소유하고, 업무 코드가 Spring Data·JPA 타입에 의존하는지**가 경계의 실질적인 판단 기준이다. 이 부분은 두 원문의 의존 규칙을 Spring 코드에 적용한 해석이다.

## 5. 모듈형 모놀리스와 마이크로서비스

**모듈형 모놀리스**는 하나의 애플리케이션으로 실행하면서 `favorite`, `order` 같은 모듈의 공개 API와 내부 구현을 구분한다. 기능별 패키지보다 **기능 간 참조 규칙**을 더 엄격하게 적용하는 단계다. [Spring Modulith](https://docs.spring.io/spring-modulith/reference/fundamentals.html)는 최상위 하위 패키지를 모듈로 인식하고, [모듈 순환 의존과 내부 패키지 접근을 검증](https://docs.spring.io/spring-modulith/reference/verification.html)할 수 있다. 기능이 늘고 서로의 구현을 직접 참조하는 일이 반복될 때 값어치가 있다.

**마이크로서비스**는 독립 배포와 팀별 소유권이 필요할 때 고려하는 실행·배포 구조다. 서비스 간 호출, 장애 대응, 데이터 일관성, 운영 도구의 비용이 생긴다. 패키지 정리가 목적이라면 서비스를 쪼개는 것으로 해결하려 하지 않는다. [Fowler, Microservices와 독립 배포](https://martinfowler.com/articles/microservices.html), [Fowler, Microservice Trade-Offs](https://martinfowler.com/articles/microservice-trade-offs.html)

## 대중성에 대한 판단과 GG마켓의 선택

문헌과 공식 예시에서 가장 확실히 확인되는 조합은 **MVC 계열의 HTTP 처리 + 업무·데이터 접근 계층 + 기능별 최상위 패키지**다. 앞의 두 요소는 오래 쓰인 책임 분리 방식이고, 마지막 요소는 Spring Boot가 제시하는 전형적인 코드 배치다. 이는 '세 구조의 채택률이 가장 높다'는 통계가 아니라, 현재 근거로 말할 수 있는 **실무 기본형**이라는 판단이다. 헥사고날·클린·수직 슬라이스·모듈형은 특정 문제가 나타났을 때 경계나 변경 단위를 더 강하게 만드는 선택지다.

GG마켓은 아직 기능 수가 적고 찜 기능부터 구현한다. 다음 구조를 **첫 구현 가설**로 둔다. 클래스와 하위 패키지는 실제 책임이 생길 때만 만든다.

```text
task.duplicate.ggmarket/
├── GgmarketApplication.java
└── favorite/
    ├── FavoriteController.java      # HTTP 요청·응답
    ├── FavoriteService.java         # 찜 규칙·작업 경계가 필요할 때
    └── FavoriteRepository.java      # 저장·조회가 필요할 때
```

이 선택은 Spring Boot 공식 예시와 맞고, 찜 관련 변경을 한곳에서 찾을 수 있다. 중복 찜 방지처럼 여러 단계가 묶인 규칙이 생기면 Service에 둘 근거가 분명해진다. 단순 조회까지 무조건 Service를 통과시키지는 않는다. 외부 알림·검색 엔진·여러 입력 채널이 붙어 업무 코드가 인프라 세부사항에 끌려가면 헥사고날 경계를, 기능 사이의 참조가 늘어 통제가 어렵다면 모듈 검증을 검토한다.

회사 프로젝트에는 현재 패키지 이름보다 **최근 기능 변경 이력**을 먼저 대입한다. 변경이 역할별 패키지 전반에 반복해서 흩어지면 기능별 묶음을, 업무 규칙의 검증이 DB·HTTP 준비에 매번 막히면 포트 경계를, 모듈 사이에 순환 의존이 생기면 모듈 공개 API를 검토한다. 불편이 실제로 나타나지 않았다면 기존 팀의 구조를 바꿀 이유는 약하다.
