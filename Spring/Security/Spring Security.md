## Spring Security Structure

![image](https://github.com/seoinee/TIL/assets/96633718/01c2cf51-1de4-4015-b487-748db0c67266)

1. 사용자가 로그인 정보와 함께 인증 요청 (HttpRequest)
1. AuthenticationFilter가 요청을 가로챔. 이때 가로챈 정보를 통해 UsernamePasswordAuthenticationToken 객체 (사용자가 입력한 데이터를 기반으로 생성, 즉 현 상태는 미검증 Authentication) 생성
3. ProviderManager (AuthenticationManager 구현체) 에게 UsernamePasswordAuthenticationToken 객체를 전달
4. AuthenticationProvider에 UsernamePasswordAuthenticationToken 객체를 전달
5. 실제 DB로 부터 사용자 인증 정보를 가져오는 UserDetailsService에 사용자 정보를 넘겨줌
6. 넘겨받은 정보를 통해 DB에서 찾은 사용자 정보인 UserDetails 객체를 생성
7. AuthenticationProvider는 UserDetails를 넘겨받고 사용자 정보를 비교
8. 인증이 완료되면, 사용자 정보를 담은 Authentication 객체를 반환
9. 최초의 AuthenticationFilter에 Authentication 객체가 반환됨
10. Authentication 객체를 SecurityContext에 저장

- 여기서 주의깊게 살펴봐야 할 부분은 UserDetailsService와 UserDetails
- 실질적인 인증 과정은 사용자가 입력한 데이터 (ID, PW 등) 와 UserDetailsService의 loadUserByUsername() 메서드가 반환하는 UserDetails 객체를 비교함으로써 동작.
- 따라서 UserDetailsService와 UserDetails 구현을 어떻게 하느냐에 따라서 인증의 세부 과정이 달라짐.

- 아래는 스프링 시큐리티에서 동작하는 기본 필터들의 목록 및 순서
- 만약 OAuth 2.0 로그인을 사용한다면, UsernamePasswordAuthenticationFilter 대신 OAuth2LoginAuthenticationFilter 가 호출. 두 필터의 상위 클래스는 AbstractAuthenticationProcessingFilter
- 사실 스프링 시큐리티는 AbstractAuthenticationProcessingFilter를 호출하고, 로그인 방식에 따라 구현체인 UsernamePasswordAuthenticationFilter 와 OAuth2LoginAuthenticationFilter 가 동작하는 방식.
![image](https://github.com/seoinee/TIL/assets/96633718/746c53bd-2559-4a22-85a7-ff3cb27d55d9)

- 조금 더 상세히 메서드 및 부가적인 과정.
![image](https://github.com/seoinee/TIL/assets/96633718/da781561-500d-4189-9b05-832ed1f2aaff)

## OAuth 2.0 (OpenID Authentication)
- 타사의 사이트에 대한 접근 권한을 업고 그 권한을 이용하여 개발할 수 있도록 도와주는 프레임워크
- 구글, 카카오, 네이버 등과 같은 사이트에서 로그인을 하면 직접 구현한 사이트에서도 로그인 인증을 받을 수 있도록 하는 구조.
- 물론 구글에서 로그인한 경우, 개발한 웹 사이트에서 구글 ID/PW를 그대로 전달해주면 안되므로, Access Token을 발급 받고, 그 토큰을 기반으로 원하는 기능을 구현해야 함.
- Access Token은 로그인을 하지 않고 인증을 할 수 있도록 해주는 인증 토큰 정도의 개념

### 주요 용어
-  Access Token을 발급 받기 위한 일련의 과정들을 인터페이스로 정의해둔 것이 OAuth.
  - Resource Owner: 개인 정보의 소유자를 가리킨다. 유저 A가 이에 해당한다.
  - Client: 제 3의 서비스로부터 인증을 받고자 하는 서버다. 직접 개발한 웹 사이트 X가 이에 해당한다.
  - Resource Server: 개인 정보를 저장하고 있는 서버를 의미한다. 구글이 이에 해당한다.
- 유저 A가 구글에서 제공해주는 서비스를 이용하는 셈이므로 타 사의 서비스를 이용하기 위해서는 신청을 해야 한다. 신청 방법은 구글, 카카오, 네이버, 페이스북 등 각각 모두 방식이 다르지만, 반드시 필요로 하는 내용은 ID, PW, 본인 인증 방법 이렇게 세 가지 정도다. 각 사이트의 개발자 Docs를 참고하면 쉽게 등록하고 발급받을 수 있다.

  - Client ID: Resource Server에서 발급해주는 ID. 웹 사이트 X에 구글이 할당한 ID를 알려주는 것이다.
  - Client Secret: Resource Server에서 발급해주는 PW. 웹 사이트 X에 구글이 할당한 PW를 알려주는 것이다.
  - Authorized Redirect Uri: Client 측에서 등록하는 Url. 만약 이 Uri로부터 인증을 요구하는 것이 아니라면, Resource Server는 해당 요청을 무시한다.

### 예시 동작 설명
- Client, 즉 직접 개발한 웹 사이트 X가 Resource Server에 등록이 완료되었다면, 이제 Access Token을 발급 받을 수 있다. 구글을 예시로 계속 설명하자면, 유저 A가 웹 사이트 X에서 특정 기능을 이용하기 위해서 로그인이 요구되고, 구글 인증 Access Token을 받기 위해 구글 로그인 링크로 연결된다.
- 예시 링크(https://accounts.google.com/?client_id=123&scope=profile,email&redirect_uri=http://localhost)의 쿼리 스트링을 살펴보면, client_id는 123, scope는 profile과 email, redirect_uri는 http://localhost임을 알 수 있다.
- 유저 A가 구글 로그인을 정상적으로 했다면, 구글은 이전에 등록되었던 client_id=123인 서버의 redirect_uri와 동일한지 확인한다.
- 일치하는 경우, 유저 A에게 scope=profile,email 기능을 넘겨줄 것인지에 대한 승인 여부를 불어보고, 동의한다면 구글은 이에 해당하는 authorization_code라는 임시 PW를 발급한다.
- 이후 http://localhost/?authorization_code=2으로 리다이렉트되며, 웹 사이트 X의 서버는 이 authorization_code를 가지고 구글에게 Access Token을 요청한다. 그리고 유저 A의 인증이 필요할 때마다 Access Token을 이용하여 접근한다.

## JWT 도입
- 스프링 부트에서 OAuth 2.0 로그인을 할 때 사용하는 여러 방법 중, JWT를 사용하게 된 이유

### 서버 기반 인증 방식
-  서버 기반 인증 방식의 핵심은 서버 측에 사용자 정보를 저장하는 것
- Spring Security에서는 별도의 설정이 없다면 세션을 이용하여 처리
- 사용자가 로그인을 하면 서버는 해당 사용자의 세션을 만들고, 서버의 메모리와 데이터베이스에 저장
- 만약 마이크로 서비스 개발을 진행하거나 서버를 확장하게 된다면, 모든 서버에게 세션의 정보를 공유해야 하므로 이를 위한 별도의 중앙 세션 관리 서버를 두곤 한다.

### Access Token과 Refresh Token
- Access Token은 리소스 (사용자 정보) 에 직접 접근할 수 있도록 해주는 정보만을 가지고 있고 Refresh Token에 비해 짧은 만료 기간을 가지며, 주로 세션에 담아 관리
- Refresh Token은 새로운 Access Token을 발급받기 위한 정보를 담고 있음. 클라이언트가 Access Token이 없거나 만료된 상태라면, Refresh Token을 통해 Auth Server에 요청하여 새로운 Access Token을 발급 받을 수 있음. Refresh Token은 외부에 노출되지 않도록 하기 위해 보통은 DB에 저장하곤 함.

### OAuth 2.0
- 위에서 설명했듯이 구글, 카카오 등에서 제공하는 Authorization Server를 통해 회원 정보를 인증하고 Access Token을 발급 받음. 그리고 발급 받은 Access Token을 이용해 직접 개발한 서버의 API 서비스를 이용 및 호출함. 마이크로 서비스이거나 서버 간의 통신이 잦은 경우, Access Token을 자주 주고 받을 수 밖에 없음.
- 그리고 각 서버는 API 호출 요청에 대해서 전달 받은 Access Token이 유효한 지를 확인해야 하는데, 이는 서버에서 클라이언트의 상태 (Access Token의 유효성) 를 관리하게끔 하며, 또 API를 호출할 때마다 Access Token이 유효한지 매번 DB에서 조회하고 새로 갱신 시 업데이트 작업을 해주어야 함. 즉, 클라이언트 상태를 관리 및 공유할 추가적인 저장 공간과 매번 요청마다 Access Token의 검증 및 업데이트를 위한 DB 호출이 발생하는 구조.
- 마이크로 서비스 개발처럼, 서버의 수가 많은 경우에는 각각의 서버가 Access Token의 유효성 및 권한 확인을 Auth Server에 요청하기 때문에 병목 현상 등이 발생해, 서버의 부하로 이어질 수 있음. 이 문제점을 해소하기 위해 JWT 기반 인증을 도입함.

### JWT
- JWT는 Claim 기반 방식을 사용
- 여기서 Claim이란 사용자에 대한 속성 값들을 가리킴.
- 즉, JWT은 의미있는 토큰 (사용자의 상태를 포함) 으로 구성되어 있기 때문에, Auth Server에 검증 요청을 보내야만 했던 과정을 생략하고 각 서버에서 수행할 수 있게 되어 비용 절감 및 Stateless 아키텍처를 구성할 수 있음.
  - 클라이언트 (사용자) 는 Auth Server에 로그인을 한다.
  - Auth Server에서 인증을 완료한 사용자는 JWT 토큰을 전달 받는다.
  - 클라이언트는 특정 애플리케이션 서버에 리소스 (서비스에 필요한 데이터) 를 요청할 때, 앞서 전달 받은 JWT 토큰을 Authorization Header에 넣어 전달한다.
  - 애플리케이션 서버는 전달 받은 JWT 토큰의 유효성을 직접 검사하여 사용자 인증을 할 수 있다.
- JWT 방식은 확장성에 큰 강점을 가짐.
- JWT 방식은 한 번 만들어 클라이언트에게 전달하면 제어가 불가능하기 때문에 만료 시간을 필수적으로 넣어 주어야 함.

### Filter Chain
- cors 사용
- 세션 매니저 무상태성으로 변경 (세션 사용안함)
- csrf 사용안함
- header <frame></frame> 끄기 -> h2-console 을 위함
- form login 사용안함
- http basic login 사용안함
- 인증 또는 인가에 대한 exception 핸들링 클래스를 정의
- 접근 허용 url 설정 (auth 는 이메일로 가입, oauth 는 social 로 가입할 때 사용)
- authorizationEndpoint 는 각 플랫폼으로 리다이렉트를 하기위한 기본 url
   -> http://localhost:8080/oauth2/authorize/naver -> 네이버 로그인화면 이동
- redirectEndpoint
   -> 네이버로 부터 accessToken 을 받아왔을 때 우리 서버에서 처리 할 url
   -> http://localhost:8080/oauth2/callback/naver -> naver 는 registrationId 가 된다
- userInfoEndpoint
   -> accessToken 을 가지고 플랫폼(네이버)에서 해당 유저에 대한 정보를 가져온다 (restTemplate 이용)
   -> userService 를 통해 가져온 정보를 가공한다
- successHandler
   -> Client 에서 요청한 redirect_uri 파라미터가 서버 application.yml 에 authorizedRedirectUris 에 설정과 같은 지
       매칭하는 작업과 토큰을 생성하여 해당 리다이렉트 주소로 토큰을 파라미터로 넘겨주는 일을 한다
   -> { 해당 주소 } ?token={ JWT Token }
- failureHanlder
   -> 에러를 처리하는 일을 하며, 해당 리다이렉트 주소 리다이렉트 하며 에러메시지를 전달한다
   -> { 해당 주소 } ?error={ message }
- TokenAuthenticationFilter 를 UsernameAuthenticationFilter 앞에 놓아 인증을 시도하는 필터로 사용된다
   -> 클라이언트에서 Header 에 Authorization Bearer {JWT} 을 넘겨주어야 인증이 된다

-  .oauth2Login()
1. authorizationEndpoint : 프론트엔드에서 백엔드로 소셜로그인 요청을 보내는 URI입니다
기본 URI는 /oauth2/authorization/{provider} 입니다. ex) /oauth2/authorization/google
URI를 변경하고 싶으면 baseUri(uri)를 사용하여 설정합니다.
위에 같이 설정하면 /oauth2/authorize/{provider}가 됩니다. ex) /oauth2/authorize/google
authorizationRequestRepository : Authorization 과정에서 기본으로 Session을 사용하지만 Cookie로 변경하기 위해 설정합니다
2. redirectionEndpoint : Authorization 과정이 끝나면 Authorization Code와 함께 리다이렉트할 URI입니다
기본 URI는 /login/oauth2/code/{provider} 입니다. ex) /login/oauth2/code/google
URI를 변경하고 싶으면 마찬가지로 baseUri(uri)를 사용하여 설정합니다.
위에 같이 설정하면 /oauth2/callback/{provider}가 됩니다. ex) /oauth2/callback/google
3. userInfoEndPoint : Provider로부터 획득한 유저정보를 다룰 service class를 지정합니다
4. successHandler : OAuth2 로그인 성공시 호출할 handler
5. failureHandler : OAuth2 로그인 실패시 호출할 handler
- exceptionHandling() : JWT를 다룰 때 생길 excepion을 처리할 class를 지정합니다
authenticationEntryPoint : 인증 과정에서 생길 exception을 처리
accessDeniedHandler : 인가 과정에서 생길 exception을 처리
- addFilterBefore : 모든 request에서 JWT를 검사할 filter를 추가합니다
UsernamePasswordAuthenticationFilter에서 클라이언트가 요청한 리소스의 접근권한이 없을 때 막는 역할을 하기 때문에 이 필터 전에 jwtAuthenticationFilter를 실행






https://velog.io/@tmdgh0221/Spring-Security-%EC%99%80-OAuth-2.0-%EC%99%80-JWT-%EC%9D%98-%EC%BD%9C%EB%9D%BC%EB%B3%B4
