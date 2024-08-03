## 정적 컨텐츠
### 작동 원리
`웹브라우저 -> 내장 톰캣 서버 -> 스프링 컨테이너`

`localhost:8080/hello-static.html` 요청하면
`hello-static` 관련 컨트롤러 먼저 찾고, 없으면 `resources:static/hello-static.html` 를 찾는다.
있으면 -> `hello-static.html` 반환한다.
<br>
<br>

## MVC,템플릿 엔진 이미지
### 작동 원리
`웹브라우저 -> 내장 톰캣서버 -> helloController -> viewResolver`

`localhost:8080/hello-mvc?name=spring` 요청하면 
컨트롤러 `return hello-templated의 model`의 `name:spring` 임에 따라
`templates/hello-template.html`을 찾아서 
`Thymeleaf` 템플릿 엔진을 처리한다.


결과화면출력 = `spring`
<br>
<br>

## API(@ResponseBody)
1. `@ResponseBody`, `return "hello "+name;` - 데이터를 그대로 출력해준다.
2. `@ResponseBody`, `return hello`; - 객체를 응답하면  {"key":"value"}인 JSON 형태로 전달한다.

### 작동 원리
웹 브라우저 -> 내장 톰캣 서버 -> `helloController` -> 이때 `@ResponseBody` 이면 `HttpMessageConverter`에서 문자열인지 객체타입인지 확인 후 각 `StringConverter` 또는 `JsonConverter` 형태로 응답한다.

- `@ResponseBody`는 HTTP의 BODY에 문자 내용을 직접 반환한다.
- 기본 문자 처리 : `StringHttpMessageConverter`
- 기본 객체 처리 : `MappaingJackson2HttpMessageConverter`

-> 등등 다양한 HttpMessageConverter가 기본으로 등록되어 있다.
