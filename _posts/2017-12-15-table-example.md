---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 5
subtitle: 로그인 구현
categories: Project
tags: [Spring, Java, Web]
---

## 도메인
사용자가 로그인을 하기 위해서 아이디와 비밀번호를 입력하면 그 정보를 담을 주머니가 필요하고, 입력된 정보와 회원정보가 일치하는지 확인하는 작업이 필요하다.

### 도메인 클래스 설계
```domain/login/LoginForm.class
@Data
public class LoginForm {
    @NotEmpty
    private String loginId;

    @NotEmpty
    private String password;
}
```
로그인 정보를 담을 클래스를 만들어 아이디와 비밀번호 변수를 생성하고, 아이디와 비밀번호는 모두 로그인에 반드시 필요한 정보이므로 @NotEmpty로 필수적으로 입력하도록 설계하였다. 


### 서비스 개발
```domain/login/LoginService.class
@Service
@RequiredArgsConstructor
public class LoginService {
    private final MemberRepository memberRepository;

    public Member login(String loginId, String password) {
        return memberRepository.findByLoginId(loginId)
                .filter(m -> m.getPassword().equals(password))
                .orElse(null);
    }
}
```
로그인의 핵심기능은 회원정보 입력을 통해 넘어온 아이디로 회원정보를 찾고, 그 회원정보의 비밀번호와 비교해서 같으면
회원정보를 반환하고, 다르면 null을 반환한다.


## 컨트롤러
```web/login/LoginController.class
@Controller
@RequiredArgsConstructor
@Slf4j
public class LoginController {
    private final LoginService loginService;

    @GetMapping("login")
    public String loginForm(@ModelAttribute("loginForm") LoginForm loginForm) {
        return "page/loginPage";
    }

    @PostMapping("login")
    public String login(@ModelAttribute LoginForm loginForm, BindingResult bindingResult, HttpServletRequest request) {
        if (bindingResult.hasErrors()) {
            return "page/loginPage";
        }

        Member loginMember = loginService.login(loginForm.getLoginId(), loginForm.getPassword());
        log.info("login? {}", loginMember);

        //로그인 실패
        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "page/loginPage";
        }

        //로그인 성공, 세션 생성
        HttpSession session = request.getSession();
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

        return "redirect:/";
    }

    @PostMapping("logout")
    public String logout(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate(); //세션 삭제
        }
        return "redirect:/";
    }
}
```
로그인화면을 보여주고, 로그인, 로그아웃을 할 수 있도록 컨트롤러를 개발할 것이다.

### 로그인
Get 방식으로 /login 주소를 받으면 로그인 화면을 띄우도록 하였다. 회원가입과 마찬가지로 로그인 검증 오류 발생시 로그인 화면에 입력정보를 담기도록 loginForm 모델을 함께 넘겨준다.
Post 방식으로 /login 주소를 받으면 로그인 기능이 실행된다. 오류 발생시 다시 로그인화면으로 이동하는데 입력정보를 담아 이동한다. 입력정보에 오류가 없다면 LoginService의 로그인기능이 실행된다. 만약 입력한 정보와 회원정보가 일치하지 않는다면 오류 메세지와 함께 다시 로그인화면으로 이동한다. 로그인에 성공했다면, 세션이 생성된다. 서블릿을 통해 HttpSession을 생성하면 쿠키를 생성한다. 쿠키의 값은 완전한 랜덤 값으로 지정되어 추정을 할 수 없도록 하여 보안에 강력하다. 이렇게 생성된 세션에 회원정보를 담는다. 그리고 메인화면으로 이동한다.

### 로그아웃
Post방식으로 /logout주소를 받으면 session.invalidate()로 회원정보가 담긴 세션을 삭제시키고 로그인화면으로 이동시킨다. 로그인과 로그아웃은 모두 return값으로 'redirect:/'을 사용하지만, 세션이 있을 땐, 메인화면으로, 세션이 없을 땐, 로그인화면으로 이동시키기 때문에 다른 화면을 출력한다.


## 홈화면
```loginPage.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
  <title>러브쉐어</title>
  <meta charset="utf-8">
  <link th:href="@{/css/bootstrap_lux.css}" href="../css/bootstrap_lux.css" rel="stylesheet">
  <style>
    .container {
      max-width: 560px;
    }
    .field-error {
      border-color: #dc3545;
      color: #dc3545;
    }
  </style>
</head>
<body>
<div class="container">
  <div class="py-5 text-center"><br><br><br><br><br><br><br><br><br>
    <h2>러브쉐어 로그인</h2><br>
    <form th:action="login" th:object="${loginForm}" method="post" class="form-group">
      <div th:if="${#fields.hasGlobalErrors()}">
        <p class="field-error" th:each="err : ${#fields.globalErrors()}"
           th:text="${err}">전체 오류 메시지</p>
      </div>
      <div class="form-floating mb-3">
        <input type="id" class="form-control" id="loginId" name="loginId" th:field="*{loginId}" placeholder="ID">
        <label for="loginId">ID</label>
      </div>
      <div class="form-floating">
        <input type="password" class="form-control" id="password" name="password" th:field="*{password}" placeholder="Password">
        <label for="password">Password</label>
      </div><br>
      <button type="submit" class="btn btn-primary bg-dark" style="width: 535px; font-size: 20px">로그인</button>
    </form><br>
    <a href="join">회원가입</a>
  </div>
</div>
</body>
</html>
```
회원가입화면과 마찬가지고 loginForm 객체를 받아와 각 입력 필드에 변수를 담고, 필드 오류 발생시 오류입력 정보가 출력되도록 하였다. 그리고, 입력정보와 회원정보가 일치하지 않으면, 오류 메시지를 출력하도록 하였다.

- 로그인 화면
![image](https://user-images.githubusercontent.com/71585151/219945967-ff33b247-a7e9-4ff4-8bdb-7438b9e3f01e.png)

- 입력정보와 일치한 회원정보가 없을 때
![image](https://user-images.githubusercontent.com/71585151/219946002-429df618-a4ff-45d1-8d62-187da0cabb90.png)
