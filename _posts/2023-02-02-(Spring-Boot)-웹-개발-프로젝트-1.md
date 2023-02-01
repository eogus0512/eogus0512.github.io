---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 1
subtitle: 프로젝트 생성 및 구축
categories: Project
tags: [Spring, Java, Web]
---

## 프로젝트 생성 

먼저 개발을 하기 위해서 프로젝트를 생성해야한다. 

Spring은 **spring initializr** 이라는 프로젝트 생성 사이트를 제공한다. 

바로가기 - 
[spring initializr](https://start.spring.io/)

이 프로그램을 이용하면 Spring 프로젝트 파일을 매우 간편하게 생성할 수 있다. 

![springInitialzr](https://user-images.githubusercontent.com/71585151/215978714-f72d0022-b5d3-43d6-8ddc-63d0a2b52e48.png)

필자는 위와 같이 프로젝트를 설정하였다.
언어는 Java를 사용하고 Spring Boot는 제일 최신버전을 선택하였다. (SNAPSHOT 버전 같은 경우 베타버전이라고 생각하면 된다.)
Packaging을 Jar를 선택하고 자바버전은 11을 사용할 것이다.

그리고 spring initializr은 Dependencies를 추가할 수 있도록 해주는데 필자는 웹 개발에 필요한 'Spring Web', 검증에 필요한 'Validation', 코드를 간결하게 만들어주는 'Lombok', 서버사이트 자바템플릿인 'Thymeleaf'를 추가하였다.

나중에 build.gradle을 통하여 Dependency를 더 추가할 수 있다.

설정이 완료가 되었으면 왼쪽 하단 'GENERATE' 버튼을 클릭하면 프로젝트의 압축파일 다운로드가 시작된다.

다운로드 후에 압축을 풀어 원하는 파일에 저장한다.


## IntelliJ에서 프로젝트 실행

![1](https://user-images.githubusercontent.com/71585151/216102547-3e6bc4db-7ef0-4fb9-8b05-cd7f6101175e.png)

IntelliJ를 실행하여 열기 버튼을 클릭한다.


![2](https://user-images.githubusercontent.com/71585151/216102550-0b11a816-c22c-47e1-b1ce-6f17b526a6ad.png)

다운로드 받은 프로젝트 파일을 찾아 파일 내에 있는 'buil.gradle'을 선택하고, 확인 버튼을 클린한다.


![3](https://user-images.githubusercontent.com/71585151/216102553-f85be366-9b60-42db-8e43-edad65f3896d.png)

프로젝트로 열기를 클릭하고 프로젝트 신뢰 버튼을 클릭하면 프로젝트가 실행된다.
초기 실행 때는 필요한 라이브러리들을 다운로드 받기 위해 시간이 다소 걸릴 수 있다.

다운로드가 완료되면 모든 준비는 끝났다. 이제 실행해보자.

![4](https://user-images.githubusercontent.com/71585151/216103916-daf3a949-694b-41af-a4e5-b255ae9fa2e9.png)

LoveShareApplication을 실행하고, 웹브라우저에서 'http://localhost:8080/'에 접속해보자

![5](https://user-images.githubusercontent.com/71585151/216104738-ed7301f2-8865-48c3-8e20-315e178fc413.png)

위와 같은 화면에 보이면 정상 실행된 것이다. 지금은 프로젝트에 아무 소스코드가 존재하지 않기 때문에 이러한 오류화면이 뜨는 것은 정상이다. 
