---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 6
subtitle: 데이터베이스 연결 및 생성
categories: Project
tags: [Spring, Java, Web, DB]
---

## 데이터베이스 연결
## DBMS 설치
많은 DBMS가 존재하지만 그중에서 간단한 h2 database라는 DBMS를 사용할 것이다. 버전은 1.4.200을 사용할 것이다. 최근에 많은 버전이 새로 출시되었지만 일부 오류가 발생하는 곳이 생겨 이전버전인 1.4.200을 사용할 것이다.

다운로드는 아래 주소를 통하여 진행하면 된다.
https://www.h2database.com/

### 환경설정
build.gradle에 다음 내용을 추가한다.
```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
```
자바의 객체와 관계형 DB를 매핑하는 기술로 JPA를 사용할 것이다. JPA는 기존의 일일이 작성해줘야했던 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다. 따라서, JPA를 사용하면 개발이 매우 편리해진다.
'spring-boot-starter-data-jpa'는 jpa뿐만아니라 jdbc 관련 라이브러리를 포함한다. 그리고 h2 database를 사용하도록 한다.

application.properties에 다음 내용을 추가한다.
```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```
이 내용은 h2와 연결하는 것과 관련한 JPA 설정들이다.

이제 프로젝트와 h2의 연결을 끝났다. 기존 코드를 변경하여 기존에 만들었던 회원가입, 로그인 기능에서 데이터베이스를 사용할 수 있도록 하겠다.


## 데이터베이스 생성 및 활용
### 도메인 수정
```domain/member/Member.class
@Data
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotEmpty(message = "아이디는 필수 입력사항입니다.")
    @Pattern(regexp="[a-zA-Z0-9]{8,14}", message = "아이디는 영어와 숫자를 조합하여 8~14자리 이내로 입력해주세요.")
    private String loginId;

    @NotEmpty(message = "비밀번호는 필수 입력사항입니다.")
    @Pattern(regexp="[a-zA-Z0-9]{8,16}", message = "비밀번호는 영어와 숫자를 조합하여 8~16자리 이내로 입력해주세요.")
    private String password;

    @NotEmpty(message = "이름은 필수 입력사항입니다.")
    private String userName;

    @NotEmpty(message = "닉네임은 필수 입력사항입니다.")
    private String nickName;

    @NotEmpty(message = "나이는 필수 입력사항입니다.")
    private String age;

    @NotEmpty(message = "성별은 필수 입력사항입니다.")
    private String gender;

    private String loverName;
}
```
기존의 Member클래스에서 변경된 점은 클래스위치에 @Entity, id 변수에 @Id, @GeneratedValue(strategy = GenerationType.IDENTITY)를 추가해준 것이다.

 - @Entity : 이노테이션이 위치한 클래스가 테이블과 상관 관계가 있다는 것을 나타낸다. 클래스와 같은 이름의 테이블과 클래스를 연결한다.
 - @Id : 이노테이션이 위치한 변수가 테이블에서 기본 키라는 것을 의미한다. 이 이노테이션이 포함된 변수는 null일수 없다.
 - @@GeneratedValue(strategy = GenerationType.IDENTITY) : JPA가 값을 스스로 만들어 테이블에 정보를 저장한다. 이때, 'strategy = GenerationType.IDENTITY'의 의미는 DBMS에서 AUTO_INCREMENT(값을 1씩 증가시켜 저장)와 같다.

### 레포지토리 수정
기존의 MemberRepository.class를 MemoryMemberRepository.class로 이름을 변경하고, Member Repository.interface를 생성한다.
```domain/member/MemberRepository.interface
public interface MemberRepository {
    Member save(Member member);
    Member findById(Long id);
    Optional<Member> findByLoginId(String loginId);

    Optional<Member> findByUserName(String userName);

    Optional<Member> findByNickName(String name);
    List<Member> findAll();
}
```
MemoryMemberRepository.class의 메서드에서 몇개의 필요한 메서드를 추가해주었다. 그리고 MemoryMemberRepository.class에서 'implements MemberRepository'를 추가하여 MemberRepository.interface를 상속받도록 하였다.

그리고 JPA를 활용할 DBMemberRepository.class를 생성해준다.
```domain/member/DBMemberRepository.class
@Primary
@Repository
@Slf4j
public class DBMemberRepository implements MemberRepository{

    private final EntityManager em;

    public DBMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        log.info("save: member={}", member);
        em.persist(member);
        return member;
    }

    @Override
    public Member findById(Long id) {
        Member member = em.find(Member.class, id);
        return member;
    }

    @Override
    public Optional<Member> findByLoginId(String loginId) {
        List<Member> result = em.createQuery("select m from Member m where m.loginId = :loginId", Member.class)
                .setParameter("loginId", loginId)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    @Override
    public Optional<Member> findByUserName(String userName) {
        List<Member> result = em.createQuery("select m from Member m where m.userName = :userName", Member.class)
                .setParameter("userName", userName)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public Optional<Member> findByNickName(String nickName) {
        List<Member> result = em.createQuery("select m from Member m where m.nickName = :nickName", Member.class)
                .setParameter("nickName", nickName)
                .getResultList();
        return result.stream().findAny();
    }
}
```
JPA를 사용하기 위해서 EntityManager이란 객체를 생성해주어야하는데 이 객체를 통해 테이블에 도메인객체를 추가해줄 수 있다. 객체를 사용하여 객체 내부에 있는 다양한 메서드를 통해 JPA를 활용할 수 있게 된다. 기존의 순수 Jdbc와 비교도 안될 정도로 간단한 코드로 데이터베이스와 정보를 주고 받을 수 있게 된다. 여기서 사용되는 메서드를 간단히 설명해보겠다.
 - persist(객체) : 객체를 매개변수로 받아 객체의 이름과 일치한 테이블에 객체 내부의 변수와 일치한 컬럼에 각각 정보를 저장한다. 
 - find(클래스명.class, 기본키) : 해당 기본키의 행 값들을 가져온다. 기본키를 중복값을 가지고 있지 않기때문에 하나의 행정보만 가져온다.
 - createQuery(qlString, 클래스면.class) : qlString에 해당하는 행들을 리스트형식으로 모두 가져온다.


### 추가작업
JPA를 통한 모든 데이터 변경은 Transaction안에서 실행해야 한다. 따라서 데이터베이스의 데이터 변경이 일어나는 모든 클래스에 @Transactional을 추가해줘야한다. 현재까지 기능중에서 데이터변경이 일어나는 기능은 회원가입이므로 MemberController에 @Transactional을 추가해준다. 
@Transactional을 사용하면 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을
커밋한다. 만약 런타임 예외가 발생하면 롤백한다. @Transactional을 추가하지 않는다면 오류가 발생하여 반드시 추가해야한다.


## 결과
![image](https://user-images.githubusercontent.com/71585151/220267025-3209d83a-56a7-49ea-aebf-322b969632e3.png)
기존의 방식은 memory에만 저장되어 프로그램을 종료하면 회원정보가 함께 사라졌다. 하지만, 이제는 데이터베이스의 데이터로서 저장되어 회원정보가 사라지지 않는다.
