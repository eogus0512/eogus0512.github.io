---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 0
subtitle: 프로젝트 구상
categories: Project
tags: [Spring, Java, Web]
---
## 진행하기에 앞서

이전에도 수많은 웹 개발 프로젝트를 진행했지만, 이전에 했던 프로젝트들은 Spring에 대한 지식이 충분하지 않은 상태로 하나하나 코드를 찾아가면서 진행했었다. 그러면서 들었던 생각이 Spring에 확실하게 공부한 후에 내가 개발하고 싶은 웹을 개발해보면서 Spring을 완전히 나의 기술로 만들고 싶었다. 그래서 Spring에 대한 전반적인 지식을 약 6개월 정도에 걸쳐 습득하였고, 이제는 그 지식들을 웹 프로젝트를 통하여 활용해보고 싶다. 그리고 내가 공부하고 활용하는 지식들을 블로그 포스팅을 통하여 기술들을 정리하고 공유하려한다.

## 개요

수많은 SNS가 있는데 그중에서 현재 사람들이 가장 많이 사용하는 SNS는 인스타그램이다. 많은 사람들이 인스타그램에 본인들의 일상을 올리곤 하는데 그중에서 커플들은 그들의 데이트 일상을 올리는 **'럽스타그램'** 이 인기다. 그래서 나는 여기서 든 생각이 럽스타그램 자체를 웹 서비스로 개발하여 커플들끼리 데이트 장소나 후기를 공유하면서 일상도 공유하고 다른 커플들을 통해 데이트 장소도 추천받을 수 있도록 하면 좋을 것 같다는 생각이 들었다.


## 소개

프로젝트 명은 **러브 쉐어** 로 지었다. 이름 그대로 커플들의 일상을 '공유'하고, 데이트 장소를 '공유'할 수 있다. 인스타그램과 Between(커플 어플리케이션)의 중간 느낌으로 커플들만 사용할 수 있는 프로그램이다. 따라서, 회원가입 후, 커플 등록이 되어야만 서비스 이용이 가능하다. 

## 사용기술
 
* 프레임워크 : Spring
* 웹 컨테이너 : Tomcat 
* 프로그래밍 언어 : Java, HTML, CSS, thymeleaf
* IDE : IntelliJ
* 웹 디자인 프레임워크 : Bootstrap(Jquery)
* 데이터베이스 : MySQL

*Note: 필자는 백엔드 개발을 위주로 할 예정이기 때문에 웹 디자인은 bootstrap에서 제공하는 디자인을 활용하여 웹 디자인을 설계할 예정이다.*

## 개발과정

1. 프로젝트 생성 및 구축
2. 프로그램 설계
3. 웹 디자인
4. 주요 기술 개발 및 테스트
5. 데이터베이스 설계 및 구축
6. 데이터 베이스 연동
7. 배포 및 유지보수

  [1]: https://daringfireball.net/projects/markdown/
  [2]: https://www.fileformat.info/info/unicode/char/2163/index.htm
  [3]: https://www.markitdown.net/
  [4]: https://daringfireball.net/projects/markdown/basics
  [5]: https://daringfireball.net/projects/markdown/syntax
