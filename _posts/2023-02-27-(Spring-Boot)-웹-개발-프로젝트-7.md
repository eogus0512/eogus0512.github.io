---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 7
subtitle: 프로필 생성을 위한 연인 관계 연결
categories: Project
tags: [Spring, Java, Web, DB]
---

## 연인관계 신청 기능 설계
### 개요
이 프로젝트에서 제일 중요한 기술은 연인 프로필을 생성하여 다른 연인들과 서로 자신들의 피드를 공유하는 것이다. 이러한 기술을 구현하기 위해서 연인관계를 연결하여 연인관계인 두 명의 유저가 하나의 프로필을 갖도록 해야한다. 회원가입 기능을 구현할 때 회원생성시 연인필드는 null값으로 두었는데 그 이유는 연인관계가 연인 신청을 통해 연결이 되도록 하기 위해서이다. 따라서 연인신청을 통해 연인관계가 연결이 되면 둘만의 프로필이 생성되도록 구현을 할 것이다.

### 흐름 설계
연인관계신청을 구현하기 위해서 먼저 서비스 흐름을 설계해야한다. 신청인과 피신칭인의 시점으로 나누어 설명해보겠다.

신청인 : 피신청인의 아이디를 입력하여 피신청인과의 연인관계를 신청한다. 그러면, 피신청인의 아이디가 신청인의 연인필드에 입력되고, 피신청인의 연인관계 수락을 기다린다. 이때, 연인관계 신청을 취소할 수 있는데 신청을 취소하게 되면 신청인 필드의 값이 null로 바뀐다.

피신청인 : 신청인의 신청이 들어오면 신청에 대해 수락 또는 거절을 선택할 수 있다. 만약 수락을 선택하면 피신청인의 연인필드에 신청인의 아이디가 입력되고 거절을 선택하면 신청인의 연인필드의 입력되어 있는 자신의 아이디가 삭제되고, 다른 사람을 신청할 수 있게 된다.

### 화면 설계
 - 신청 화면

![image](https://user-images.githubusercontent.com/71585151/221557869-ef0e5f8a-7afe-4649-973e-7b18de41a6b6.png)

 - 대기 화면

![image](https://user-images.githubusercontent.com/71585151/221557989-68717e97-978c-4111-838d-97d65738dbe8.png)


 - 피신청 화면

 ![image](https://user-images.githubusercontent.com/71585151/221558190-e5312591-c297-4c55-8fba-9f79cd202ad6.png)


 - 연인관계 성립 시 화면

![image](https://user-images.githubusercontent.com/71585151/221558354-ffeeb1ae-bb21-4438-b9a1-0bf0c30593d2.png)

(임시화면, 추후에 이 화면에 프로필 구현 예정)



## 연인관계 신청 기능 구현
### 화면 구현
```profilePage.html
<!DOCTYPE html>
<html th:replace="~{layout/common_layout :: layout(~{::title}, ~{::section})}">
<head>
    <title>러브쉐어</title>
</head>
<body>
<section class="text-center">
    <div th:if="${session.loginMember.loverName == null}" style="max-height: 700px;">
        <div th:if="${subscriberName == null}">
            <br><br><br><br><br><br><br><br>
            <form th:action="profile-request" method="post">
                <div class="form-group">
                    <h3>연인의 아이디를 입력하세요.</h3><br>
                    <input type="text" class="form-control" id="loverName" name="loverName">
                    <br><button type="submit" class="btn btn-primary bg-dark" style="width: 200px; font-size: 20px">신청</button>
                </div>
            </form>
        </div>
        <div th:unless="${subscriberName == null}">
            <br><br><br><br><br><br><br><br>
            <h3><span th:text="${subscriberName}"></span>님의 연인관계 요청이 있습니다.</h3>
            <h3>요청을 수락하시겠습니까?</h3>
            <form th:action="profile-accept" method="post">
                <input type="hidden" id="subscriberName1" name="subscriberName1" th:value="${subscriberName}">
                <button type="submit" class="btn btn-primary bg-dark" style="width: 200px; font-size: 20px">수락</button>
            </form><br>
            <form th:action="profile-reject" method="post">
                <input type="hidden" id="subscriberName2" name="subscriberName2" th:value="${subscriberName}">
                <button type="submit" class="btn btn-primary bg-dark" style="width: 200px; font-size: 20px">거절</button>
            </form>
        </div>
    </div>
    <div th:unless="${session.loginMember.loverName == null}">
        <div th:if="${subscriberName == null}">
            <br><br><br><br><br><br><br><br>
            <form th:action="profile-request-cancel" method="post">
                <div class="form-group">
                    <h3>현재 <span th:text="${session.loginMember.loverName}"></span>님의 수락을 기다리는 중입니다.</h3><br>
                    <br><button type="submit" class="btn btn-primary bg-dark" style="width: 200px; font-size: 20px">요청 취소</button>
                </div>
            </form>
        </div>
        <div th:unless="${subscriberName == null}">
            <br><br><br><br><br><br><br><br>
            연인 있음
        </div>
    </div>
</section>
</body>
</html>
```
앞서 설계한대로 흐름에 따라서 화면이 구성되도록 th:if와 th:unless를 사용하여 구현하였다. 조건값으로 'session.loginMember.loverName'과 'subscriberName'로 하였는데 여기서 'session.loginMember.loverName'은 연인필드이고, 'subscriberName'은 자신을 연인으로 신청한 사람의 이름이다. 따라서 이 두 객체의 존재에 따라서 보여지는 화면을 다르게 하였다.

### 기능 구현
```web.profile.ProfileController.class
@Controller
@RequiredArgsConstructor
@Slf4j
@Transactional
public class ProfileController {
    private final MemberRepository memberRepository;

    @GetMapping("/profile")
    public String profile(Model model, HttpServletRequest request) {
        HttpSession session = request.getSession();
        Member member = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
        Optional<Member> subscriber = memberRepository.findByLoverName(member.getLoginId());
        log.info("subscriber? {}", subscriber);
        if (subscriber.isPresent()) {
            model.addAttribute("subscriberName", subscriber.get().getLoginId());
        } else {
            model.addAttribute("subscriberName", null);
        }

        return "page/profilePage";
    }

    @PostMapping("/profile-request")
    public String profile_request(@RequestParam String loverName, HttpServletRequest request) {
        update(loverName, request);
        return "redirect:/profile";
    }

    @PostMapping("/profile-request-cancel")
    public String profile_request_cancel(HttpServletRequest request) {
        update(null, request);
        return "redirect:/profile";
    }

    @PostMapping("/profile-accept")
    public String profile_accept(@RequestParam("subscriberName1") String subscriber, HttpServletRequest request, Model model) {
        update(subscriber, request);
        model.addAttribute("subscriberName", subscriber);
        return "redirect:/profile";
    }

    @PostMapping("/profile-reject")
    public String profile_reject(@RequestParam("subscriberName2") String subscriber) {
        Optional<Member> member = memberRepository.findByLoginId(subscriber);
        Member update = memberRepository.findById(member.get().getId());
        update.setLoverName(null);
        return "redirect:/profile";
    }

    public void update(String data, HttpServletRequest request) {
        HttpSession session = request.getSession();
        Member member = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);

        Member update = memberRepository.findById(member.getId());
        update.setLoverName(data);
        session.setAttribute(SessionConst.LOGIN_MEMBER, update);
        log.info("session? {}", update);
    }
}
```
먼저 이 컨트롤러에선 프로필과 관련한 기능들을 모두 넣을 예정이다. 먼저 프로필 창을 들어가면 GET방식으로 /profile을 요청하게 되는데 이때 화면을 바로 보여주지 않고 연인관계신청기능에서 가장 중요한 'subscriber'을 찾아 그 값을 프로필 화면으로 넘겨준다. 자신을 연인으로 신청한 사람의 객체를 찾아 객체의 loginId값을 subscriber로 넘긴다.

각 화면에서 버튼을 통해 발생하는 이벤트들을 프로필 컨트롤러를 만들어 이벤트별로 각각 구현하였다. 이 컨트롤러에선 주로 loverName필드의 데이터값의 변경이 발생하므로 update메소드를 따로 만들었다. update메소드에선 데이터를 변경할 객체를 찾아 데이터를 변경하면 @Transaction을 통하여 데이터베이스에 변경사항이 적용된다.

그리고 이벤트별로 적절하게 update메소드를 사용하여 loverName필드의 데이터를 사용자의 요구에 맞게 변경하도록 하였다. 그런데 신청인의 연인관계 신청을 거절하였을때 발생하는 'profile_reject'메소드에선 사용자의 loverName필드의 데이터를 변경하는 update메소드를 사용하지 않고, 따로 'subscriber'의 loverName필드의 데이터를 변경하도록 하였다.

### 결과
![image](https://user-images.githubusercontent.com/71585151/221564361-71cda5fe-09eb-48ff-897d-d6445f77a3c3.png)
신청 절차가 모두 완료되면 두 사용자의 loverName에 서로의 아이디가 저장되어 있는것을 확인할 수 있다.
