# THE MAN — 남성 쇼핑몰

> 카카오 소셜 로그인, 이메일 인증, 주소 찾기 API를 활용해 보안성을 높인 회원 서비스를 구현한 남성 쇼핑몰

![Java](https://img.shields.io/badge/Java-007396?style=flat&logo=java&logoColor=white)
![Spring](https://img.shields.io/badge/Spring-6DB33F?style=flat&logo=spring&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Oracle](https://img.shields.io/badge/Oracle-F80000?style=flat&logo=oracle&logoColor=white)

<br>

## 기술 스택

| 분류 | 기술 |
|---|---|
| 백엔드 | Java · Spring Framework |
| 프론트엔드 | JavaScript · jQuery · Ajax |
| 데이터베이스 | Oracle DB · MyBatis |
| 외부 API | 카카오 소셜 로그인 · 다음 주소 찾기 |
| 인프라 | AWS |

<br>

## 아키텍처

```
Client (JSP)
    ↕ Ajax
Spring Controller
    ↕
Service → MyBatis Mapper → Oracle DB
         ↕
  Kakao OAuth API
```

<br>

## 핵심 코드

### 카카오 소셜 로그인

인가코드로 토큰을 요청하고, 토큰으로 사용자 정보를 받아온다. DB에 존재하는 회원이면 로그인 처리, 없으면 회원가입 화면으로 이동시킨다.

```java
@GetMapping("code")
public ModelAndView code(String code, HttpSession session, ModelAndView mv)
    throws IOException, ParseException {

  String accessToken = kakaoService.getToken(code);
  Member checkUser = kakaoService.getUserInfo(accessToken);
  Member loginUser = memberService.selectUser(checkUser);

  if (loginUser != null) {
    session.setAttribute("loginUser", loginUser);
    mv.setViewName("redirect:/");
  } else {
    session.setAttribute("checkUser", checkUser);
    mv.setViewName("member/memberJoin");
  }
  return mv;
}
```

### 이메일 인증

정규식으로 형식 검증 → 중복 확인 → 인증번호 발송 → 코드 일치 시 가입 버튼 활성화 순서로 처리한다.

```javascript
function codeCheck() {
  if ($('#codeInfo').val() != $('#code').val()) {
    alert('인증번호가 일치하지 않습니다!');
    emailSend();
  } else {
    alert('인증번호가 일치합니다');
    $('#enroll-form button[type=submit]').removeAttr('disabled');
  }
}
```

### 로그인 인터셉터

로그인이 필요한 페이지 접근 시 세션을 확인해 비로그인 사용자를 차단한다.

```java
public class LoginInterceptor extends HandlerInterceptorAdapter {
  @Override
  public boolean preHandle(HttpServletRequest request,
                           HttpServletResponse response, Object handler) throws Exception {
    HttpSession session = request.getSession();
    if (session.getAttribute("loginUser") != null) return true;
    session.setAttribute("alertMsg", "로그인이 필요합니다");
    response.sendRedirect(request.getContextPath());
    return false;
  }
}
```
