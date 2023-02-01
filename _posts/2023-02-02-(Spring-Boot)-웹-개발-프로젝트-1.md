---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 1
subtitle: 프로젝트 생성 및 구축
categories: Project
tags: [Spring, Java, Web]
---

먼저 개발을 하기 위해서 프로젝트를 생성해야한다. 

Spring은 **spring initializr** 이라는 프로젝트 생성 사이트를 제공한다. 
[spring initializr](https://start.spring.io/)

이 프로그램을 이용하면 Spring 프로젝트 파일을 매우 간편하게 생성할 수 있다. 

바로가기 - 
![springInitialzr](https://user-images.githubusercontent.com/71585151/215978714-f72d0022-b5d3-43d6-8ddc-63d0a2b52e48.png)

필자는 위와 같이 프로젝트를 설정하였다.
언어는 Java를 사용하고 Spring Boot는 제일 최신버전을 선택하였다. (SNAPSHOT 버전 같은 경우 베타버전이라고 생각하면 된다.)
Packaging을 Jar를 선택하고 자바버전은 11을 사용할 것이다.

그리고 spring initializr은 Dependencies를 추가할 수 있도록 해주는데 필자는 웹 개발에 필요한 'Spring Web', 검증에 필요한 'Validation', 코드를 간결하게 만들어주는 'Lombok', 서버사이트 자바템플릿인 'Thymeleaf'를 추가하였다.

나중에 build.gradle을 통하여 Dependency를 더 추가할 수 있다.

설정이 완료가 되었으면 왼쪽 하단 'GENERATE' 버튼을 클릭하면 프로젝트의 압축파일 다운로드가 시작된다.

다운로드 후에 압축을 풀어 

