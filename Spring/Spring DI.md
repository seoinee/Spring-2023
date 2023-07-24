# 스프링 기초정리

## 기본형태
---
<br>

```
// 1. 원격호출 가능한 프로그램으로 등록
@Controller
public class Hello {
	// 2. URL과 메서드를 연결 @RequestMapping
	@RequestMapping("/hello")
	public void main() {
		System.out.println("Hello");
	}
}
```

- @Controller - 원격 호출이 가능한 프로그램으로 등록

- @RequestMapping - URL과 메서드를 연결함

<br>

##  HTTP 요청과 응답
---
###  1. HttpServletRequest
<br>

```
@Controller
public class RequestInfo {
	@RequestMapping("/requestInfo")
	public void main(HttpServletRequest request) {
		System.out.println("request.getMethod()="+request.getMethod());      // 요청 방법
        System.out.println("request.getProtocol()="+request.getProtocol());  // 프로토콜의 종류와 버젼 HTTP/1.1
        System.out.println("request.getScheme()="+request.getScheme());      // 프로토콜
	}
}
```

- 프로그램을 URL 통해 요청시 tomcat이 HttpServletRequest 객체를 만들고 요청한 정보를 객체에 담아 해당 메소드의 매개변수로 넘겨줌
  - ex) http://<hi>요청할URL/requestInfo로 요청시 HttpServletRequest request로 넘겨받음

### 1.1 HttpServletRequest 메서드
<br>
<p align="center">
	<img width="581" alt="HttpServletRequest 메서드" src="https://user-images.githubusercontent.com/46203866/180649881-a3b83e79-d302-4ab6-af09-b290c213058e.PNG">
</p>

<center>출처 : 스프링의 정석</center>

###  2. HttpServletReponse
<br>

```
@Controller
public class RequestInfo {
	@RequestMapping("/requestInfo")
	public void main(HttpServletRequest request, HttpServletResponse response) throws IOException {
		"""
		request 관련처리부분들
		"""

		response.setContentType("text/html");	//전송할 데이터의 형식지정
		response.setCharacterEncoding("UTF-8");	//한글사용을 위한 인코딩
		PrintWriter out = response.getWriter();	//response 객체에서 브라우저로 응답하기 위한 출력 스트림을 얻음
		out.println(year + "년 " + month + "월 " + day + "일은 ");
		out.println(yoil + "요일입니다.");
	}
}
```

- 브라우저에 응답하기위한 HttpServletResponse 객체도 tomcat이 해당 메소드의 매개변수로 넘겨줌
- try - catch를 사용해야하지만 편의를 위해 IOException을 사용함
- 위 코드처럼 단순 출력이 아닌 실제 웹처럼 출력시 html 태그를 사용해야함

<br>

### 3. 프로토콜(Protocol)
---
<br>

- 상호간의 통신을 위한 약속 또는 규칙
- 주고받는 데이터에 대한 형식을 정의한 것

### 3.1 HTTP (Hyper Text Transfer Protocol)
-	텍스트 기반 프로토콜
	-	ex) HTML
-	상태를 유지하지 않음 (Stateless) : 서버가 클라이언트의 정보를 저장하지 않는 네트워크 프로토콜
	-	요청에 대한 응답만을 처리함
	-	이러한 점을 보완하기 위해 쿠기&세션을 통해 클라이언트를 구분할 수 있음
-	확장가능한 프로토콜 (확장성) : 기존에 HTTP에 정의된 기능을 확장할 수 있음
	-	ex) 커스텀 헤더(header)를 통해 기능추가

### 3.4 HTTP 응답메시지
---
<br>
<p align="center">
<img width="658" alt="2 Http응답메시지- 상태코드" src="https://user-images.githubusercontent.com/46203866/180834178-58919c90-1f3d-4956-937c-14ab88d2114c.PNG">
</p>

<center>출처 : 스프링의 정석</center>

|상태코드|의미||
|:------:|:---:|---|
|1xx|Informational|HTTP 1.1에서 추가됨, Client<->Server간 정보교환이 목적|
|2xx|Success|성공|
|3xx|Redirect|다른 URL 요청|
|4xx|Client Error|클라이언트측에서 요청에러|
|5xx|Server Error|서버측 에러, 클라이언트의 요청은 OK|
