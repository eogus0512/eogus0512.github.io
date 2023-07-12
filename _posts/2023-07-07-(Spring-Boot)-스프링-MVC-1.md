---
layout: post
title: (Spring Boot) 스프링 MVC - 1
subtitle: 서블릿
categories: Study
tags: [Spring]
---

## 서블릿

**@ServeletComponentScan**

이 애노테이션을 추가하면 스프링 부트에서 서블릿을 직접 등록해서 사용 가능



**서블릿 등록방법**

서블릿 클래스에 서블릿 애노테이션 (@WebServlet(name = "서블릿 이름", urlPatterns = "URL 매핑"))을 추가하면 HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 서블릿 클래스의 service메서드를 실행



**Http 요청 메시지 로그로 확인하는 법**

설정 파일인 application.properties 를 생성하고, 다음 코드를 추가

```
logging.level.org.apache.coyote.http11=debug
```

이 코드를 추가하면 서버가 받은 HTTP 요청 메시지를 출력



**서블릿 컨테이너 동작 방식**

내장 톰캣 서버 생성

![](C:\Users\Daehyun\AppData\Roaming\marktext\images\2023-07-07-20-25-43-image.png)

스프링 부트는 내장 톰캣 서버를 생성하여 서블릿 컨테이너에 서블릿을 생성하여 추가



**HTTP 요청, HTTP 응답 메시지**

![](C:\Users\Daehyun\AppData\Roaming\marktext\images\2023-07-07-20-27-25-image.png)



웹 어플리케이션 서버의 요청 응답 구조

![](C:\Users\Daehyun\AppData\Roaming\marktext\images\2023-07-07-20-30-50-image.png)

웹 브라우저의 HTTP 요청을 통해 서블릿 클래스에 매핑된 URL을 요청하면, 웹 어플리케이션 서버는 컨테이너에서 해당 서블릿 클래스를 HTTP 요청 메시지를 기반으로 생성된 request, response객체를 매개변수로 하여 실행하여 반환된 response 객체 정보로 HTTP 응답 생성하여 웹 브라우저에 전달



 

## HTTP 요청

**HttpServletRequest 역할**

서블릿은 HTTP 요청 메시지를 파싱하고, 그 결과를 HttpServletRequest 객체에 담아 제공



#### HTTP 요청 데이터

**3가지 방법**

GET - 쿼리 파라미터

- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달

Post - HTML Form

- 메시지 바디에 쿼리 파라미터 형식으로 전달

HTTP message body에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용, JSON, XML, TEXT



**HTTP 요청 데이터 - GET 쿼리 파라미터**

메시지 바디 없이, URL의 쿼리 파라미터를 사용해서 데이터 전달

예) http://localhost:8080/request-param?username=hello&age=20

- request.getParameter("파라미터 이름") : 단일 파라미터 조회, 중복된 이름의 파라미터가 존재할 때 첫 번째 값을 반환, 이때, request.getParameterValues()를 사용

- request.getParameterNames() : 모든 파라미터 이름 조회, 반환형은 Enumeration<String>

- request.getParameterMap() :  파라미터를 Map으로 조회, 반환형은 Map<String, String[]>



**HTTP 요청 데이터 - POST HTML Form**

메시지 바디에 쿼리 파라미터 형식으로 데이터를 전달

- 요청 URL : http://localhost:8080/basic/hello-form.html

- content-type : application/x-www-form-urlencoded

- username=hello&age=20

GET과 쿼리 파라미터 형식이 같기 때문에 GET과 같은 방식의 쿼리 파라미터 조회 메서드 사용



**HTTP 요청 데이터 - API 메시지 바디 - 단순 텍스트**

HTTP message body에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용, JSON, XML, TEXT

- 데이터 형식은 주로 JSON 사용

- POST, PUT, PATCH

InputSream을 사용하여 HTTP 메시지 바디의 데이터를 직접 읽을 수 있음. InputStream은 byte 코드를 반환하기 때문에 String으로 보기 위해선 문자표(예) UTF_8 Charset)를 지정해주어야 함

InputStream으로 읽은 데이터인 JSON 결과를 사용할 수 있는 자바 객체로 변환하려면 JSON 변환 라이브러리를 추가하여 사용해야한다. (스프링 부트는 기본으로 Jackson라이브러리(ObejctMapper)를 제공한다.)





## HTTP 응답

**HTTPServletResponse 역할**

- HTTP 응답 메시지 생성 : HTTP 응답코드 지정, 헤더 생성, 바디 생성

- 편의 기능 제공 : Content-Type, 쿠키, Redirect



**HTTP 응답 데이터**

- 단순 텍스트 응답 

```
write.println("ok");
```

- HTML 응답

```
PrintWriter writer = response.getWriter();
writer.println("<html>");
writer.println("<body>");
writer.println(" <div>안녕?</div>");
writer.println("</body>");
writer.println("</html>");
```

- HTTP API - MessageBody JSON 응답

```
response.setHeader("content-type", "application/json");
response.setCharacterEncoding("utf-8");
HelloData data = new HelloData();
data.setUsername("kim");
data.setAge(20);

String result = objectMapper.writeValueAsString(data);
response.getWriter().write(result);
```
