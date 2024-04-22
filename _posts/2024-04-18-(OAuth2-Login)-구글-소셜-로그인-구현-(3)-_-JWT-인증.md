---
layout: post
title: (OAuth2 Login) 구글 소셜 로그인 구현 (3) - JWT 인증
subtitle: Spring Security + JWT를 이용한 소셜 로그인 구현
categories: OAuth2
tags: [Spring, Spring Security, JWT]
---

## JWT
### JWT란?
- JSON Web Token의 약자로 인증에 필요한 정보들을 암호화한 JSON 토큰을 의미한다.
- 토큰에는 사용자 관련 여러 정보가 포함되어있다.
- JWT를 통해 Stateless 하게 설계가 가능하다.

### JWT 인증 순서
1. 사용자가 로그인 정보를 가지고 로그인을 서버에 요청한다.
2. 서버는 JWT 토큰을 생성하여 클라이언트에 보낸다.
3. 클라이언트는 받은 JWT 토큰을 사용하여 서비스 요청할 때마다 Http Header에 JWT를 넣어 서버에 요청한다.
4. 서버는 해당 JWT를 검증하고, 유효하다면 요청에 대한 응답을 반환한다.

### JWT 구조
- Header
  - 토큰의 타입과 전자서명 알고리즘이 저장된다.
- Payload
  - Claim운 토큰에서 사용하는 정보를 포함한다.
  - Payload는 이러한 Claim을 여러개 포함한다.
- Signature
  - header와 payload를 암호화한 것과 서버가 가지고 있는 개인 키를 암호화한 정보가 저장된다.
 
### Access Token & Refresh Token
- Access Token
  - 로그인 시 발급 받고, 인증 처리 위해 사용되는 토큰을 말한다.
  - 탈취 위험으로부터 벗어나기 위해 유효 기간을 짧게 설정한다.
  - 처음 로그인 요청 시 서버에서 Access Token을 발급하고, 클라이언트는 Http Header에 이를 저장하여 서비스를 이용하기 위해 요청마다 Access Token을 보낸다.
- Refresh Token
  - 유효시간이 짧은 Access Token으로 인한 trade-off 문제를 해결하기 위한 토큰
  - DB와 같은 저장소에서 저장되어 Access Token을 재발급 해주는 토큰을 말한다.
  - 처음 로그인 요청시 서버에서 Access Token과 함께 Refresh Token을 발급하는데 Refresh Token은 Http Header에 저장되지 않고, 서버 DB에 저장된다.
  - Refresh Token은 자체적으로 인증 용도로 사용되지 않고, Access Token을 재발급하기 위한 용도로만 사용된다.
 
<br>
<br>

## JWT 구현
- Access 토큰과 Refresh 토큰을 모두 사용한다.
- Access 토큰은 HTTP Header에 저장하고, Refresh 토큰은 쿠키에 저장한다.

### 의존성 추가 및 JWT 설정
```
dependencies {
  ...

	//JWT
	implementation 'com.auth0:java-jwt:4.2.1'

  ...
}
```
- JWT를 사용하기 위해 build.gradle에 위 코드를 추가한다.

```
jwt:
  secretKey: 'qwjedi19hrb18odxkcnwgkjkladhnw9182nmsd0s89y12mne123mksnsaujfdhowkrn3k45786zxnmcvwjehrkeljhfjwh1'

  access:
    expiration: 3600000 # 1시간 30분
    header: Authorization

  refresh:
    expiration: 1209600000 # 2주
    header: Authorization-refresh
```
- application.yml에 jwt의 비밀키와 access 토큰과 refresh 토큰의 만료 시간 및 header 이름을 설정한다.

### JWT 생성 서비스
```
@Service
@RequiredArgsConstructor
@Getter
@Slf4j
public class JwtCreateAndUpdateService {
    @Value("${jwt.secretKey}")
    private String secretKey; //jwt 비밀키

    @Value("${jwt.access.expiration}")
    private Long accessTokenExpirationPeriod; // access 토큰 유효 시간

    @Value("${jwt.refresh.expiration}")
    private Long refreshTokenExpirationPeriod; // refresh 토큰 유효 시간

    private static final String ACCESS_TOKEN_SUBJECT = "AccessToken";
    private static final String REFRESH_TOKEN_SUBJECT = "RefreshToken";
    private static final String EMAIL_CLAIM = "email";
    private final UserRepository userRepository;
    private final RedisUtil redisUtil;

    //access 토큰 생성
    public String createAccessToken(String email) {
        Date now = new Date(); // 현재 시간

        return JWT.create() // JWT 토큰 생성 빌더 반환
                .withSubject(ACCESS_TOKEN_SUBJECT) //JWT subject를 access 토큰으로 설정
                .withExpiresAt(new Date(now.getTime() + accessTokenExpirationPeriod)) // 토큰 만료 시간을 access 토큰 유효시간으로 설정
                .withClaim(EMAIL_CLAIM, email) // 클레임을 이메일 값으로 설정
                .sign(Algorithm.HMAC512(secretKey)); //HMAC512 알고리즘을 사용하여 secret키로 암호화하여 access 토큰 생성
    }

    //refresh 토큰 생성
    //대부분 access 토큰 생성과정과 동일하지만 클레임에 이메일 설정 X
    public String createRefreshToken() {
        Date now = new Date();
        return JWT.create()
                .withSubject(REFRESH_TOKEN_SUBJECT)
                .withExpiresAt(new Date(now.getTime() + refreshTokenExpirationPeriod))
                .sign(Algorithm.HMAC512((secretKey)));
    }

    //RefreshToken을 DB에 업데이트
    public void updateRefreshToken(String email, String refreshToken) {
        userRepository.findByEmail(email)
                .ifPresentOrElse(
                        user -> user.updateRefreshToken(refreshToken), //회원이 존재하면 refresh 토큰 업데이트
                        () -> new IllegalStateException("일치하는 회원이 없습니다.") //회원이 존재하지 않으면 exception 발생
                );
    }

    public Long getRemainingExpirationTime(String accessToken) {
        Long expiration = JWT.decode(accessToken).getExpiresAt().getTime();
        Long now = new Date().getTime();

        return (expiration - now);
    }

    //토큰 유효성 검사
    public boolean isTokenValid(String token) {
        try {
            JWT.require(Algorithm.HMAC512(secretKey)).build().verify(token);
            if (redisUtil.hasKeyBlackList(token)) {
                throw new RuntimeException("토그아웃 상태의 토큰입니다.");
            }
            return true;
        } catch (Exception e) {
            log.error("유효하지 않은 토큰입니다. {}", e.getMessage());
            return false;
        }
    }
}
```
- JWT 토큰을 생성하기 위한 서비스를 구현한다.
- `createAccessToken()` : Access 토큰을 생성하는 메서드이다. 클레임을 이메일 값으로 설정하기 위해서 이메일을 매개변수로 받는다.
- `createRefreshToken()` : Refresh 토큰을 생성하는 메서드이다. 클레임에 이메일을 설정하지 않는다.
- `updateRefreshToekn()` : Refresh 토큰을 DB에 저장하는 메서드이다.
- `getRemainingExpirationTime()` : Access 토큰의 남은 만료시간을 반환한다. 로그아웃에 사용된다.
- `isTokenValid()` : 토큰의 유효성을 검사한다. 나중에 다시 얘기하겠지만 Redis에 토큰이 저장되어있는 경우 로그아웃 상태의 토큰으로 인식되어 유효하지 않는 토큰으로 구분된다.

### JWT 추출 서비스
```
@Service
@RequiredArgsConstructor
@Getter
@Slf4j
public class JwtExtractService {
    @Value("${jwt.secretKey}")
    private String secretKey; //jwt 비밀키

    @Value("${jwt.access.header}")
    private String accessHeader; // access 헤더

    @Value("${jwt.refresh.header}")
    private String refreshHeader; // refresh 헤더

    private static final String EMAIL_CLAIM = "email";
    private static final String BEARER = "Bearer ";

    //클라이언트의 요청으로 access 토큰 header에서 추출
    public Optional<String> extractAccessToken(HttpServletRequest request) {
        return Optional.ofNullable(request.getHeader(accessHeader)) // 헤더의 accessHeader의 값 가져옴
                .filter(refreshToken -> refreshToken.startsWith(BEARER)) //Bearer 로 시작하면 통과
                .map(refreshToken -> refreshToken.replace(BEARER, "")); //'Bearer '부분을 삭제해 순수 토큰만 가져옴
    }

    //클라이언트 요청으로 refresh 토큰 header에서 추출
    public Optional<String> extractRefreshToken(HttpServletRequest request) {
        return Arrays.stream(request.getCookies())
                .filter(cookie -> refreshHeader.equals(cookie.getName()))
                .map(Cookie::getValue)
                .findFirst();
    }

    //AccessToken에서 Email추출
    public Optional<String> extractEmail(String accessToken) {
        try {
            //require을 통해 secretKey를 사용하여 HMAC512알고리즘으로 토큰 유효성 검사 설정
            return Optional.ofNullable(JWT.require(Algorithm.HMAC512(secretKey))
                    .build() // 반환된 빌더로 JWT verifier 생성
                    .verify(accessToken) //access토큰을 검증하고 유효하지 않으면 예외 발생
                    .getClaim(EMAIL_CLAIM) // 이메일 가져옴
                    .asString()); //String 형식으로 가져옴
        } catch (Exception e) {
            log.error("Access Token이 유효하지 않습니다.");
            return Optional.empty(); //빈 Optional객체 반환
        }
    }
}
```
- `extractAccessToken()` : Http 헤더에서 access 토큰을 추출한다.
- `extractRefreshToken()` : 쿠키에서 refresh 토큰을 추출한다.
- `extractEmail()` : Access 토큰에서 email을 추출한다.

### JWT 전송 서비스
```
@Service
@RequiredArgsConstructor
@Getter
@Slf4j
public class JwtSendService {

    @Value("${jwt.access.header}")
    private String accessHeader; // access 헤더

    @Value("${jwt.refresh.header}")
    private String refreshHeader; // refresh 헤더

    //Http 헤더로 Access 토큰 보내기
    public void sendAccessToken(HttpServletResponse response, String accessToken) {
        response.setStatus(HttpServletResponse.SC_OK); // 성공 상태
        response.setHeader(accessHeader, accessToken); // Http 헤더에 accessHeader를 키로 access 토큰 저장
        log.info("Access Token : {}", accessToken);
    }

    //Http 헤더에 Access 토큰, 쿠키에 Refresh 토큰 저장하여 전송
    public void sendAccessAndRefreshToken(HttpServletResponse response, String accessToken, String refreshToken) {
        response.setStatus(HttpServletResponse.SC_OK); // 성공 상태
        response.setHeader(accessHeader, accessToken); // Http 헤더에 accessHeader를 키로 access 토큰 저장
        response.addCookie(createCookie(refreshHeader, refreshToken));
        log.info("Access Token : {}, Refresh Token : {}", accessToken, refreshToken);
    }

    private Cookie createCookie(String key, String value) {
        Cookie cookie = new Cookie(key, value);
        cookie.setMaxAge(14*24*60*60);
        cookie.setPath("/");
        cookie.setHttpOnly(true);

        return cookie;
    }
}
```
- `sendAccessToken()` : Http 헤더에 Access 토큰을 담아 보낸다.
- `sendAccessAndRefreshToken()` : Http 헤더에 Access 토큰을 담고, 쿠키에 Refresh 토큰을 담아 전송한다.
- `createCookie()` :  Refresh 토큰을 담은 쿠키를 생성하는 메서드이다.

### 사용자 인증 및 권한 부여 관리 클래스
```
public class AuthUser implements UserDetails {

    private final Long id;
    private final String email;
    private final Role role;

    public static AuthUser createAuthUser(User user) {
        return new AuthUser(user);
    }

    private AuthUser(User user) {
        email = user.getEmail();
        role = user.getRole();
        id = user.getId();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> collection = new ArrayList<>();
        collection.add(() -> role.getKey());
        return collection;
    }

    @Override
    public String getPassword() {
        return null;
    }

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    public Long getId() {
        return id;
    }
}
```
- Spring Security를 사용하여 사용자 인증 및 권한 부여를 관리하는 데 사용되는 사용자 세부 정보(UserDetails)를 구현한 클래스이다.

### JWT 인증 필터
- JWT 서비스를 이용하여 JWT 인증 처리, 인증 실패, 토큰 재발급 등의 JWT 관련 기능을 수행하는 필터이다.
- JWT 인증 관련해서 다음 3가지 경우의 상황이 발생할 수 있다.
  1. Access 토큰이 유효한 경우 - Refresh 토큰이 사용자 요청 쿠키에 없다면 인증에 성공한다.
  2. Access 토큰이 유효하지 않고, 사용자 요청 쿠키에 Refresh 토큰이 있는 경우 - DB에 Refresh 토큰과 비교하여 일치하면 Access 토큰 재발급한다. 그렇지 않다면 인증 실패로 처리한다.
  3. Access 토큰이 없거나 유효하지 않고, Refresh 토큰도 없거나 유효하지 않을 경우 - 인증 실패, 403 ERROR 발생
- 위 상황을 처리하는 인증 필터를 구현한다.

```
@RequiredArgsConstructor
@Slf4j
@Component
public class JwtAuthenticationProcessingFilter extends OncePerRequestFilter {

    private static final String NO_CHECK_URL = "/login"; // '/login'으로 들어오는 요청은 Filter사용 X

    private final JwtCreateAndUpdateService jwtCreateAndUpdateService;
    private final JwtExtractService jwtExtractService;
    private final JwtSendService jwtSendService;
    private final UserRepository userRepository;

    private GrantedAuthoritiesMapper authoritiesMapper = new NullAuthoritiesMapper();

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (request.getRequestURI().equals(NO_CHECK_URL)) { // '/login'으으로 요청이 들어오면 다음 필터를 호출
            filterChain.doFilter(request, response);
            return;
        }

        String refreshToken = jwtExtractService.extractRefreshToken(request) //사용자 요청 header에서 refresh 토큰 추출
                .filter(jwtCreateAndUpdateService::isTokenValid) //토큰 유효성 검사
                .orElse(null); //토큰이 없거나 유효하지 않으면 null 반환

        // refresh토큰이 사용자 header에 존재, access 토큰이 없어서 refresh 토큰을 요청한 경우
        if (refreshToken != null) {
            checkRefreshTokenAndReIssueAccessToken(response, refreshToken); //refresh 토큰이 DB의 refresh토큰과 일치하는 경우 access 토큰 재발급
            return;
        }

        // refresh토큰이 사용자 header에 존재 X, access 토큰이 있는지, 유효한지 검사
        if (refreshToken == null) {
            checkAccessTokenAndAuthentication(request, response, filterChain);
        }
    }

    //refresh 토큰으로 DB에서 유저를 찾고, 유저가 있다면 access, refresh 토큰 재발급 후 DB 업데이트
    private void checkRefreshTokenAndReIssueAccessToken(HttpServletResponse response, String refreshToken) {
        userRepository.findByRefreshToken(refreshToken) //refresh 토큰으로 user 찾음
                .ifPresent(user -> { // user가 존재하면
                    String reIssueRefreshToken = reIssueRefreshToken(user); // refresh 토큰 재발급, DB 업데이트
                    jwtSendService.sendAccessAndRefreshToken(response, jwtCreateAndUpdateService.createAccessToken(user.getEmail()), reIssueRefreshToken);
                    //header에 access 토큰과 refresh 토큰 담아 보냄
                });
    }

    //refresh 토큰 재발급 후 refresh 토큰 DB에 업데이트
    private String reIssueRefreshToken(User user) {
        String reIssuedRefreshToken = jwtCreateAndUpdateService.createRefreshToken();
        updateUserRefreshToken(user, reIssuedRefreshToken);
        return reIssuedRefreshToken;
    }

    //user의 refresh 토큰 업데이트 후 저장
    private void updateUserRefreshToken(User user, String reIssuedRefreshToken) {
        user.updateRefreshToken(reIssuedRefreshToken);
        userRepository.saveAndFlush(user);
    }


    // access 토큰 유효성 검사 및 인증 처리
    private void checkAccessTokenAndAuthentication(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    )  throws ServletException, IOException{
        jwtExtractService.extractAccessToken(request) //사용자 요청 header에서 access 토큰 추출
                .filter(jwtCreateAndUpdateService::isTokenValid) // 토큰 유효성 검사
                .ifPresent(accessToken -> jwtExtractService.extractEmail(accessToken) // 유효한 토큰이 있으면 이메일 추출
                        .ifPresent(email -> userRepository.findByEmail(email) // 추출된 이메일이 존재하면 이메일로 user 검색
                                .ifPresent(this::saveAuthentication))); // 인증 허가 메소드 실행
        filterChain.doFilter(request, response); //다음 필터로 넘어가도록 설정
    }

    //Security를 사용하여 사용자를 인증 및 허가
    private void saveAuthentication(User user) {
        AuthUser authUser = AuthUser.createAuthUser(user);

        //UserDetails 객체를 사용하여 사용자의 인증 정보를 나타내는 토큰 생성
        Authentication authentication = new UsernamePasswordAuthenticationToken(
                authUser, null, authoritiesMapper.mapAuthorities(authUser.getAuthorities()));
        SecurityContextHolder.getContext().setAuthentication(authentication); //SecurityContextHolder에 생성된 인증 객체를 설정하여 사용자의 인증 정보를 저장
    }

}
```
- `doFilterInternal()` : JWT 인증 로직을 수행한다.
- `checkRefreshTokenAndReIssueAccessToken()` : Refresh 토큰이 쿠키에 존재할 때 실행된다. Refresh 토큰으로 DB에서 사용자를 찾고, 사용자가 존재한다면 `reIssueRefreshToken()`메서드를 통해 토큰을 재발급 후 DB 업데이트하고, Access 토큰을 생성하여 Refresh 토큰과 함께 클라이언트로 보낸다. 만약, 사용자가 없다면 오류가 발생한다.
- `reIssueRefreshToken()` : Refresh 토큰 재발급 후 Refresh 토큰을 DB에 업데이트한다.
- `checkAccessTokenAndAuthentication` : Refresh 토큰이 쿠키에 존재하지 않을 때 실행된다. Access 토큰이 유효한지 검사하고 유효하다면 인증 처리한다.
- `saveAuthentication()` : 사용자 인증 및 허가를 위한 메서드이다. Security를 통해 사용자의 인증 정보를 나타내는 토큰을 생성하여 해당 정보를 저장한다.
