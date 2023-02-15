---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 4
subtitle: 회원가입 구현
categories: Project
tags: [Spring, Java, Web]
---

## 패키지 구조
패키지는 크게 domain과 web으로 나눌 것이다. domain에는 domain, repository, service 등의 핵심 비지니스 업무 영역을 포함할 것이고, web에는 controller 등 웹페이지 관련 영역을 포함할 것이다. 따라서, web이 다른 것으로 대체되더라도 domain에는 영향을 끼치지 않아야한다. 즉, web -> domain 의존관계가 형성되도록 나눌것이다.



## 도메인 설계
```domain/member/Member.java
@Data
public class Member {
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
회원정보에 필요한 요소로 ID, 비밀번호, 이름, 닉네임, 나이, 성별, 연인이름을 선택하였고, 연인이름을 제외한 모든 요소는 회원가입시에 반드시 입력해야만 가입이 가능하도록 하려고 한다. 이때, 필요한 annotation은 @NotEmpty이다. 요소에 @NotEmpty를 추가해주면 해당 요소가 비어있으면 가입이 불가능(오류발생)하도록 설정할 수 있다. 그리고 오류발생시 message를 설정해주었다.
ID, 비밀번호는 기입을 하였더라도 해당 패턴에 맞게 입력하지 않으면 오류가 발생하도록 @Pattern을 추가해주었다. 그리고 각자에 맞는 패턴을 regexp에 정규식으로 설정해주었다. 마지막으로 클래스에 @Data를 추가하여 getter, setter를 생략해주었다.

```domain/member/GenderType.enum
public enum GenderType {
    MEN("남자"), WOMEN("여자");

    private final String description;

    GenderType(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}
```
나중에 회원가입화면에서 성별을 radio로 설계하기 위해서 성별 종류를 enum type으로 만들어주었다.



## 레포지토리 구현
나중에는 DB와 연결하여 레포지토리를 DB로 관리할 것이지만 일단, 메모리로 회원정보가 저장되도록 레포지토리를 구현할 것이다.
```domain/member/MemberRepository.java
@Repository
@Slf4j
public class MemberRepository {
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    public Member save(Member member) {
        member.setId(++sequence);
        log.info("save: member={}", member);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }
    public Optional<Member> findByLoginId(String loginId) {
        return findAll().stream()
                .filter(m -> m.getLoginId().equals(loginId))
                .findFirst();
    }
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
    public void clearStore() {
        store.clear();
    }
}
```
레포지토리 기능은 크게 회원정보를 저장하는 기능, ID로 레포지토리에서 회원정보를 찾는 기능, 레포지토리에서 회원정보를 모두 가져오는 기능으로 나뉜다.
- save메소드는 회원에 고유 ID(로그인 ID와는 별개)를 저장 순서대로 부여하여 해시맵에 저장한다. 회원가입에 사용된다.
- findByLoginId 메소드는 loginId를 가지고 loginId에 해당하는 회원정보를 가져온다. 로그인에 사용된다.
- findAll 메소드는 모든 회원정보를 가져온다.
- clearStore 메소드는 이름그대로 메모리에 저장되어 있는 회원정보를 모두 삭제한다.



## 회원가입 기능 구현
```web/member/MemberController.java
@RequiredArgsConstructor
@Controller
@RequestMapping("/join")
public class MemberController {
    private final MemberRepository memberRepository;

    @GetMapping
    public String joinForm(@ModelAttribute("member") Member member) {
        return "page/joinPage";
    }

    @PostMapping
    public String saveMember(@Valid @ModelAttribute Member member, BindingResult result) {
        if (result.hasErrors()) {
            return "page/joinPage";
        }

        memberRepository.save(member);
        return "redirect:/";
    }

    @ModelAttribute("genderType")
    public GenderType[] gender() {
        return GenderType.values();
    }
}
```
/join을 Get요청으로 받으면 회원가입화면으로 이동하도록 하였다. 이때, 회원가입 정보 검증때 오류가 발생하면 회원정보를 저장해두기 위해서 @ModelAttribute로 member모델을 만들어주어 함께 넘겨주도록 하였다. 
회원가입화면에서 회원정보 입력 후 가입버튼을 누르면 Post요청으로 /join으로 넘어간다. 회원정보에 이전에 설정한 검증요소에 부합하지 않은 정보가 있으면 오류가 발생하게 되고, 회원가입 페이지로 이동시킨다. 이때, 입력했던 회원정보도 같이 넘어가 회원가입화면에 나타나도록 하였다. 오류가 발생하지 않았다면 MemberRepository의 save메소드를 사용하여 회원정보를 저장하고, 메인화면으로 이동시킨다.
추가적으로 genderType 모델을 만들어 회원가입화면에서 사용할 수 있도록 하였다.



## 회원가입 화면 설계
```page/joinPage.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <title>러브쉐어</title>
  <meta charset="utf-8">
  <link th:href="@{/css/bootstrap_lux.css}" href="../css/bootstrap_lux.css" rel="stylesheet">
  <style>
    .container {
      max-width: 700px;
    }
    .field-error {
      border-color: #dc3545;
      color: #dc3545;
    }
  </style>
</head>
<body>
<div class="container">
  <div class="py-5"><br><br><br><br><br>
    <h2 class="text-center">회원가입</h2>
    <form th:action th:object="${member}" method="post">
      <fieldset>
        <div class="form-group">
          <label for="loginId" class="form-label mt-4">ID</label>
          <input type="text" class="form-control" th:errorclass="field-error" id="loginId" th:field="*{loginId}" aria-describedby="IdHelp" placeholder="ID">
          <small id="IdHelp" class="form-text text-muted">아이디는 필수입력사항입니다.</small>
          <div th:class="field-error" th:errors="*{loginId}" />
        </div>
        <div class="form-group">
          <label for="password" class="form-label mt-4">비밀번호</label>
          <input type="password" class="form-control" th:errorclass="field-error" id="password" th:field="*{password}" placeholder="Password">
          <div class="field-error" th:errors="*{password}" />
        </div>
        <div class="form-group">
          <label for="userName" class="form-label mt-4">이름</label>
          <input type="text" class="form-control" th:errorclass="field-error" id="userName" th:field="*{userName}" placeholder="이름을 입력하세요.">
          <div class="field-error" th:errors="*{userName}" />
        </div>
        <div class="form-group">
          <label for="nickName" class="form-label mt-4">닉네임</label>
          <input type="text" class="form-control" th:errorclass="field-error" id="nickName" th:field="*{nickName}" placeholder="닉네임을 입력하세요.">
          <div class="field-error" th:errors="*{nickName}" />
        </div>
        <div class="form-group">
          <label for="age" class="form-label mt-4">나이</label>
          <select class="form-select" th:errorclass="field-error" id="age" th:field="*{age}">
            <option value="" selected="selected">나이를 선택하세요.</option>
            <th:block th:each="num : ${#numbers.sequence(14, 50)}">
              <option th:text="${num}" th:value="${num}"></option>
            </th:block>
          </select>
          <div class="field-error" th:errors="*{age}" />
        </div>
        <label class="form-label mt-4">성별</label>
        <div class="form-check" th:each="type : ${genderType}">
          <input class="form-check-input" th:errorclass="field-error" type="radio" th:field="*{gender}" th:value="${type.name()}">
          <label class="form-check-label"  th:for="${#ids.prev('gender')}" th:text="${type.description}">
            남자
          </label>
          <div class="field-error" th:errors="*{gender}" />
        </div>
        <br>
        <button type="submit" class="btn btn-primary text-center bg-dark" style="width: 680px; font-size: 20px">확인</button>
      </fieldset>
    </form>
  </div>
</div>
</body>
</html>
```
부트스트랩을 사용하여 깔끔하게 회원가입화면을 제작했다. 
form 태그에 th:object="${member}"을 삽입하여 회원가입화면에서 member객체를 사용할 수 있도록 하였다. 그리고 객체의 변수를 각각 해당 회원가입 요소 input태그에 넣어줬다. 타임리프는 이 th:field를 인식하여 id와 name을 field이름에 맞게 자동 생성해준다. 따라서 id와 name 속성을 입력해줄 필요가 없다. 그런데 필자는 input마다 label을 넣었기 때문에 확실한 인식을 위해서 id속성만 입력하였다. 회원가입화면으로 처음이동했을 때는 field가 비어있지만 회원가입 오류 발생시에 회원가입화면으로 이동했을 때는 field에 회원가입 정보가 저장되어 있다. 그리고 회원가입 각 요소마다 오류 발생시 오류 메세지를 출력이 되도록 div태그를 삽입해주었다. 오류 메시지는 th:errors속성으로 나타난다.

- 회원가입 화면
![image](https://user-images.githubusercontent.com/71585151/219054832-88978ff9-f0c2-4ae0-bcb0-a87b02ed7a5b.png)

- @NotEmpty오류 발생시 화면
![image](https://user-images.githubusercontent.com/71585151/219055386-f4690d71-a924-4169-b126-721b67733bf3.png)

- @Pattern 오류 발생시 화면
![image](https://user-images.githubusercontent.com/71585151/219055530-009986ca-4db7-4a88-a3ce-509f1a64e5d4.png)

