---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 3
subtitle: 공통 화면 설계
categories: Project
tags: [Spring, Java, Web]
---

## 템플릿 레이아웃
하나의 웹 사이트의 화면들은 공통 영역이 많이 있다. 보통은 똑같은 navigation bar, footer 등이 웹사이트의 대부분의 화면에 포함된다. 이러한 공통 영역들을 html파일마다 추가해주는 것을 비효율적이기 때문에 타임리프의 '템플릿 레이아웃'기능을 사용하여 하나의 html파일에 공통 영역을 한번에 추가하여 다른 html 파일에서 사용할 수 있도록 할 수 있다.

필자는 기능구현을 중요시하기 위해서 공통 영역으로 navigation bar만 포함할 것이다.

### 공통 영역 html 파일 생성

```common_layout.html
<!DOCTYPE html>
<html th:fragment="layout (title)" xmlns:th="http://www.thymeleaf.org">
<head>
    <title th:replace="${title}">레이아웃 타이틀</title>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap_lux.css}" href="../css/bootstrap_lux.css" rel="stylesheet">
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark"...>
<script src="../js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
공통 영역을 하나의 html파일에 포함한다. 위의 코드에서 head, body 태그 안에 공통적으로 포함될 영역의 코드를 삽입한다.
필자는 <head>태그 안에는 타이틀 이름, 메타데이터, 사용할 bootstrap 테마를 삽입하였고, body태그 안에는 navbar, bootstrap의 javascript파일을 삽입하였다. 

공통 영역이라고 해도 화면마다 조금씩 다르게 구성해야할 때가 있다. 이때, th:fragment와 th:replace를 활용하면 된다. fragment는 다른 화면에 포함될 템플릿 조각으로 이해하면 되는데 필자는 html태그에 fragment를 추가했기 때문에 전체 html을 템플릿 조각으로 만든 것이다. 이 fragment에 대체할 태그를 포함할 수 있는데 템플릿 레이아웃을 사용할 html파일에서 태그를 지정하여 replace를 통해 현재태그를 대체할 수 있다.
  

### 공통 영역 적용
  
이제 공통 영역을 구성하였으니 메인화면에 이를 적용시켜볼 것이다. 메인화면의 자세한 구성은 나중에 하고 일단 공통영역만 적용시켜보도록 하겠다.
  
```main.html
<!DOCTYPE html>
<html th:replace="~{layout/common_layout :: layout(~{::title})}">
<head>
  <title>러브쉐어</title>
</head>
<body>
</body>
</html>
```
th:replace를 사용하여 공통영역을 가져왔다. 
```
~{layout/common_layout :: layout(~{::title})}
```
위 코드는 layout/common_layout.html의 layout템플릿 조각을 가져올 것인데 title 태그를 현재 title태그로 대체한다는 것을 의미한다.
  

### 적용 결과
```HomeController.java
@Controller
public class HomeController {
    @GetMapping("/")
    public String main(
            @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {

        if (loginMember == null) {
            LoginForm loginForm = new LoginForm();
            model.addAttribute("loginForm", loginForm);
            return "page/loginPage";
        }

        model.addAttribute("member", loginMember);
        return "page/main";
    }
}
```  
![image](https://user-images.githubusercontent.com/71585151/219018491-be858b76-013f-4844-83a0-f7dbe98211c5.png)
메인화면에 공통영역이 정상적으로 포함된 것을 확인할 수 있다.
