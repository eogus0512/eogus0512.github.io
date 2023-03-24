---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 8
subtitle: 인터셉터 추가
categories: Project
tags: [Spring, Java, Web]
---

## 인터셉터
### 개요
현재 로그인을 하지 않은 상태에서 메인화면과 프로필화면의 주소를 입력하더라도 들어가진다. 이는 기대했던 상황이 아니고, 로그인이 된 사용자만 메인화면과 프로필 화면으로 이동할 수 있도록 변경해야 한다. 컨트롤러수준에서 화면마다 화면에 들어갈 때마다 로그인 여부를 체크할 수도 있겠지만 이는 너무 번거롭고 중복되는 코드가 많아진다. 다행히도 스프링에선 한번에 로그인체크를 할 수 있도록 지원한다. 그것이 바로 인터셉트이다. 스프링 인터셉터는 컨트롤러 호출 직전에 호출되어 컨트롤러를 호출하기 전에 여러가지 상황을 살펴보고 조건에 맞춰 행동한다. 이러한 인터셉터를 통해서 로그인 인증 체크를 하도록 할 것이다.

### 인터셉터 흐름
인터셉터의 기능들은 HandlerIntercepter인터페이스를 구현하여 사용할 수 있다. HandlerIntercepter인터페이스의 메소드는 크게 preHandle, postHandle, afterCompletion으로 나눌 수 있는데 먼저, 인터셉터가 실행되는 순서를 살펴보면 아래와 같다.
```
preHandle() 실행 -> 컨트롤러실행 -> postHandle() -> afterCompletion()
```
preHandle() : 컨트롤러 호출전에 반드시 실행된다. preHandle의 반환값이 true이여야 컨트롤러가 실행되고, false면 진행되지 않고, 컨트롤러가 실행되지 않는다.
postHandle() : 컨트롤러 호출 후에 실행된다. 따라서, preHandle에서 반환값이 false이면 실행되지 않는다.
afterCompletion() : 뷰가 렌더링 된 후에 실행된다. postHandle과는 다르게 항상 실행된다. 따라서 이 메소드를 통해 예외가 발생한 이유를 출력하여 보여줄 수 있다.

### 인증 체크
```web.interceptor.LoginCheckInterceptor.class
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
            response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        log.info("인증 체크 인터셉터 실행 {}", requestURI);

        HttpSession session = request.getSession(false);
        if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            log.info("미인증 사용자 요청");
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }
        return true;
    }
}
```
HandlerIntercetor인터페이스를 상속받아 구현하는 클래스를 만들어 준다. 이 로그인 인증 인터셉터의 경우 postHandle과 afterCompletion이 따로 필요하지 않기 때문에 preHandle만을 구현하였다. 
이 메소드에선 로그인이 안되어 있어도 들어갈 수 있도록 설정한 페이지 이외에 다른 페이지를 들어가려고 시도되었을 때 로그인 여부를 확인하여 로그인이 되어있지 않으면 로그인 페이지로 이동하도록 힌다. 코드를 살펴보면 사용자 세션이 존재하지 않으면 주소를 받아와 로그인창의 주소값과 함께 로그인 후에 들어가려고 시도되었던 주소로 바로 이동할 수 있도록 response.sendRedirect을 사용하였다. 그리고 false를 반환하여 들어가려고 시도하였던 페이지로 갈 수 없도록 하였다. 만약 사용자 세션이 존재한다면 true를 반환하여 정상적으로 페이지를 불러오도록 하였다.

```web.WebConfig.class
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/join", "/login", "/logout", "/css/**", "/*.ico", "/error");
    }
}
```
인터셉터 클래스를 만들었으니 인터셉터를 추가하고 설정하여야한다. 그래서 프로젝트의 환경설정을 위한 클래스를 web패키지에 따로 만들어 인터셉트를 추가, 설정하였다. 인터셉터를 등록 및 설정에 WebMvcConfigurer가 제공하는 addInterceptors()를 사용한다. 그리고 registry.addInterceptor(new LogInterceptor())로 인터셉터를 등록하고, order()의 매개변수에 숫자를 추가하여 호출 순서를 지정한다. 필자는 2로 설정하였는데 추후에 1로 로그인터셉터를 설정하기 위해서이다. 그리고 addPathPatterns로 URL패턴을 지정하고, excludePathPatterns에 로그인 필요없이 들어갈 수 있는 페이지 패턴을 지정한다.

### 결과
![image](https://user-images.githubusercontent.com/71585151/227555583-00f5ea7e-a617-40c0-aee3-93b5ed4c288b.png)

로그인 하기 전에 주소 창에 profile을 입력하여 페이지 접속을 시도했지만 인터셉트에 의해 막아져 redirtURL=/profile과 함께 로그인 페이지로 이동한 것을 확인 할 수 있다. 로그인 후에는 원래 접속하려던 profile페이지로 바로 이동하는 것을 확인 할 수 있다.
