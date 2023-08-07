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


##### https://velog.io/@tmdgh0221/Spring-Security-%EC%99%80-OAuth-2.0-%EC%99%80-JWT-%EC%9D%98-%EC%BD%9C%EB%9D%BC%EB%B3%B4
