---
layout: post
title: (스프링 기본) Domain, Repository, Service, Controller
subtitle: 스프링 기본
categories: Spring
tags: [Spring]
---

## 웹 애플리케이션 계층 구조
![image](https://github.com/eogus0512/eogus0512.github.io/assets/71585151/69f34114-e526-4f7a-b572-68437c9778a4)

## Domain

### 도메인 모델
- 도메인 모델이란 특정 문제와 관련된 모든 주제의 '개념' 모델이다. (비즈니스 도메인 객체)
- 소프트웨어 개발에서 시스템의 비즈니스 로직과 규칙을 표현하는 핵심 요소이다.

### 구성요소
- 엔티티 (Entity) : 고유 식별자 (ID)를 가진 객체로, 비즈니스의 주요 개념을 나타낸다.
- 값 객체 (Value Object) : 식별자가 없고 불변인 객체로, 엔티티의 속성을 나타내는데 사용된다.
- 어그리게이트(Aggregate) : 엔티티와 값 객체와 관련된 집합으로 하나의 루트 엔티티로 접근하여 일관성을 유지한다.
- 리포지토리 (Repository) : 도메인 객체를 저장하고 검색하는데 사용되는 인터페이스이다.

### 구현
```
public class User {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String nickname;

    private String email;

    public void signUp(String nickname) {
        this.nickname = nickname;
    }
}
```
- 도메인 모델의 예시이다.
- 엔티티를 나타내며, 실제로 저장되는 컬럼 이외에도 컬럼 값을 조작하는 비즈니스 로직을 포함할 수 있다.

### 도메인 주도 설계 (DDD)
- 도메인 주도 설계(Domain-Driven Design)은 도메인의 개념을 중심으로 하여 소프트웨어를 설계하는 방법론이다.
- 복잡한 시스템을 개발하는 경우 도메인의 본질을 최대한 반영하여 모델을 설계하고 구현한다.
- 복잡한 도메인을 이해하고, 도메인에 집중하여 효과적으로 모델링하기 위해 사용된다.

<br>

## Repository

### 리포지토리
- 도메인 객체의 영속성을 관리하는 역할을 담당한다.
- 데이터베이스와 같은 영속성 저장소에 대한 접근을 캡슐화하고, 도메인 객체를 CRUD(Create, Read, Update, Delete)하는 메서드를 제공한다.
- 도메인 모델과 데이터 접근 로직을 분리할 수 있도록 한다.

### 구성 요소
- 인터페이스(Interface) : 리포지토리가 제공해야하는 메서드를 정의한다.
- 구현 클래스(Implemention Class) : 인터페이스를 구현하여 실제 데이터베이스 접근 로직을 포함한다.

### 구현
```
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findAllByNickname(String nickname);
    Optional<User> findByEmail(String email);
}
```
- Spring Data JPA는 JpaRepository인터페이스를 통해 기본적인 CRUD 메서드를 자동으로 제공한다.
- 추가적인 커스텀 쿼리 메서드는 위의 코드와 같이 정의하여 사용할 수 있다.

<br>

## Service

### 서비스
- 소프트웨어 시스템 내에서 특정 작업이나 기능을 수행하는데 중점을 둔 설계 요소이다.
- 서비스에서 시스템의 핵심 비즈니스 로직을 구현한다.
- 일반적으로 엔티티에 포함되지 않는 복잡한 비즈니스 로직을 캡슐화한다. 이를 통해 외부 시스템과의 통신과 같은 작업을 수행할 수 있다.
- 특정 기능을 모듈화하여 코드의 가독성과 유지보수성을 향상시킨다.

### 유형
- 애플리케이션 서비스 : 사용자 인터페이스와 도메인 모델 사이의 조정 역할을 한다.
- 도메인 서비스 : 도메인 로직을 캡슐화하며, 특정 엔티티에 국한되지 않는 비즈니스 로직을 구현한다. 여러 엔티티에 걸친 로직이나 엔티티 간의 상호작용을 처리한다.
- 인프라스트럭쳐 서비스 : 데이터베이스 접근, 메시징, 로깅와 같은 인프라 관련 작업을 처리한다.

### 구현
```
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public void signUp(
            UserSignupRequest userSignupRequest,
            HttpServletResponse response,
            AuthUser authUser) {
        User user = findUser(authUser);
        user.signUp(userSignupRequest.nickname());
    }

    public void logout(String accessToken, AuthUser authUser) {
        User user = findUser(authUser);
        user.logout();
    }

    public List<UserNicknameResponse> search(String nickname) {
        List<User> users = userRepository.findByNicknameContaining(nickname);
        return users.stream()
                .map(user -> UserNicknameResponse.from(user))
                .toList();
    }

    private User findUser(AuthUser authUser) {
        return userRepository.findById(authUser.getId())
                .orElseThrow(() -> new BusinessException(UserErrorCode.USER_NOT_FOUND));
    }
}
```
- 도메인 로직을 캡슐화하고, 사용자 관련된 책임을 명확히 분리하여 비즈니스 로직을 구현한다.
- 서비스 계층으로 분리하고, 모듈화를 통해 유지보수성을 향상시킨다.

<br>

## Controller

### 컨트롤러
- 소프트웨어 아키텍처에서 사용자의 요청을 처리하고, 응답을 생성하는 역할을 담당하는 컴포넌트이다.
- 모델과 뷰를 연결하여 사용자 인터페이스와 도메인 로직 간의 상호작용을 관리한다.

### 주요 역할
1. 요청처리 : 사용자의 입력 (HTTP 요청)을 수신하고 이를 처리한다. URL매핑을 통해 적절한 컨트롤러 메서드가 호출된다.
2. 비즈니스 로직 호출 : 서비스 계층을 호출하여 필요한 비즈니스 로직을 실행한다. 컨트롤러에선 비즈니스 로직을 포함하지 않고 모두 서비스 계층에 위임한다.
3. 모델 업데이트 : 서비스를 통해 생성된 결과를 받아 데이터를 포함한 모델을 업데이트 한다.
4. 응답 생성 : JSON, HTML, XML 등 다양한 형식으로 사용자가 볼 수 있는 응답을 생성한다.

### 구현
```
public class UserController {
    private final UserService userService;

    @PostMapping(value = "/sign-up",
            consumes = {MediaType.APPLICATION_JSON_VALUE, MediaType.MULTIPART_FORM_DATA_VALUE})
    public ResponseEntity<Void> signUp(
            @RequestPart MultipartFile image,
            @RequestPart UserSignupJsonRequest request,
            @AuthenticationPrincipal AuthUser authUser,
            HttpServletResponse response
    ) {

        UserSignupRequest userSignupRequest = UserSignupRequest.of(request, image);
        userService.signUp(userSignupRequest, response, authUser);

        return ResponseEntity.ok()
                .build();
    }

    @PostMapping("/logout")
    public ResponseEntity<Void> logout(
            @AuthenticationPrincipal AuthUser authUser,
            HttpServletRequest request
    ) {
        userService.logout(accessToken, authUser);

        return ResponseEntity.ok()
                .build();
    }
}
```
- `@GetMapping`과 `@PostMapping` 등을 통해 HTTP 요청을 처리한다.
- 서비스 계층의 메서드(비즈니스 로직)을 사용하고 결과를 응답받는다.
- 응답을 생성하여 반환한다.
