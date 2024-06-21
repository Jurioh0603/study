![image](https://github.com/Jurioh0603/study/assets/148063470/e8de1756-883d-49c4-a2b0-9702ea41b3d2)## 시작
API 예외 처리 결과는 JSON 형태로 받아야 하지만 html 을 반환함<br>
이를 해결하기 위해 여러가지 방법을 살펴볼 것<br>
이 장에서는 서블릿 오류 방식 사용할 것<br>
WebServerCustomizer 클래스
```
작동 될 수 있도록 @Component 주석 풀기
```

ApiExceptionController - API 예외 컨트롤러
```
@Slf4j
@RestController
public class ApiExceptionController {
 @GetMapping("/api/members/{id}")
 public MemberDto getMember(@PathVariable("id") String id) {
   if (id.equals("ex")) {
     throw new RuntimeException("잘못된 사용자");
   }
   return new MemberDto(id, "hello " + id);
 }
 @Data
 @AllArgsConstructor
 static class MemberDto {
   private String memberId;
   private String name;
 }
}
```
단순히 회원 조회 기능, 예외 테스트 위해 URL에 연결된 id의 값이 ex면 예외 발생하도록 함<br><br>

ErrorPageController - API 응답 추가
```
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest
      request, HttpServletResponse response) {
   log.info("API errorPage 500");
   Map<String, Object> result = new HashMap<>();
   Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
   result.put("status", request.getAttribute(ERROR_STATUS_CODE));
   result.put("message", ex.getMessage());
   Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
   return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
}
```
설명<br>
> `MediaType.APPLICATION_JSON_VALUE`는 클라이언트 요청 HTTP Header의 `Accept` 값이 `application/json` 일 때<br>
> 해당 500API 메서드가 호출되는 것<br>
> Map을 이용해 응답 데이터 만들어 결과가 JSON으로 반환<br><br>

## API 예외 처리 - 스프링 부트 기본 오류 처리
스프링 부트가 제공하는 기본 오류 방식인 `BasicErrorController`를 사용할 수 있음<br>
코드 내용
```
 @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
 public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
 @RequestMapping
 public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```
설명<br>
> `errorHtml()` `produces = MediaType.TEXT_HTML_VALUE`는 클라이언트 요청이 `text/html`인 경우 `errorHtml`을 호출해 view를 제공<br>
> `error()` 외 경우에 호출되고 ResponseEntity로 HTTPBody에 JSON 데이터를 반환<br>
> `server.error.path` 로 수정 가능, 기본 경로 `/error`<br><br>

결론<br>
각 API 마다 처리할 error페이지가 다를 수 있기 때문에 `BasicErrorController`로는 HTML 화면을 처리할 때 사용하고 <br>
API 오류 처리는 뒤에서 설명할 `@ExceptionHandler`을 사용하자<br><br>

## API 예외 처리 - HandlerExceptionResolver 시작
API마다 상태코드를 다르게 처리하고 싶을 때<br>
ApiExceptionController - 수정
```
 @GetMapping("/api/members/{id}")
 public MemberDto getMember(@PathVariable("id") String id) {
   if (id.equals("ex")) {
     throw new RuntimeException("잘못된 사용자");
   }
   if (id.equals("bad")) {
     throw new IllegalArgumentException("잘못된 입력 값");
   }
   return new MemberDto(id, "hello " + id);
 }
```
`bad` id 예외 추가<br>
실행해보면 상태코드 500임<br><br>

HandlerExceptionResolver<br>
컨트롤러 밖으로 예외가 던져진 경우 예외를 해결하고 동작을 새로 정의할 수 있는 방법<br>
`HandlerExceptionResolver` 적용 전, 후 비교<br>
![image](https://github.com/Jurioh0603/study/assets/148063470/fae30040-fdd9-4c54-ae84-77ad379c340f)<br>
![image](https://github.com/Jurioh0603/study/assets/148063470/ff762d18-4eec-4d38-8482-f436efa83686)<br><br>

적용 후 5번 부터가 다른 점임<br>
`HandlerExceptionResolver`가 `ModelAndView`를 찾고 그 값이 있으면 해당 view를 보여주고 만약 `null`이면 기본 에러 페이지를 보여줌<br>


HandlerExceptionResolver - 인터페이스 보기
```
public interface HandlerExceptionResolver {
  ModelAndView resolveException(
  HttpServletRequest request, HttpServletResponse response,
  Object handler, Exception ex);
}
```
설명<br>
> `hadler` 핸들러(컨트롤러) 정보<br>
> `Exeption ex` 핸들러에서 발생한 예외<br><br>

MyHandlerExceptionResolver
```
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
  @Override
  public ModelAndView resolveException(HttpServletRequest request, 
         HttpServletResponse response, Object handler, Exception ex) {
    try {
      if (ex instanceof IllegalArgumentException) {
        log.info("IllegalArgumentException resolver to 400");
        response.sendError(HttpServletResponse.SC_BAD_REQUEST, 
        ex.getMessage());
        return new ModelAndView();
      }
    } 
    catch (IOException e) {
      log.error("resolver ex", e);
    }
    return null;
  }
}
```
설명<br>
> 여기서는 `IllegalArgumentException`이 발생하면 `response.sendError(400)`를 호출해서 HTTP 상태 코드를 400으로 지정하고, 빈 `ModelAndView`를 반환<br><br>

반환 값에 따른 동작 방식<br>
> 빈 `ModelAndView` `new ModelAndView()`처럼 빈 `ModelAndView`를 반환하면 뷰를 렌더링 하지 않고 정상 흐름으로 리턴됨<br>
> `ModelAndView`지정하면 해당 뷰를 렌더링 함<br>
> `null` 반환시 다음 `ExceptionResolver` 찾아서 실행, 만약 처리 할 수 없으면 기존 발생한 예외를 서블릿 밖으로 던짐<br>
> 위 코드에서 /bad 이외 요청시 null이 반환되고 다음 `ExceptionResolver`가 없으므로 기본으로 제공하는 예외 페이지 보여줌<br>

ExceptionResolver 활용<br>
> 예외 상태 코드 변환<br>
> > `response.sendError(xxx)` 호출로 변경해 서블릿 상태 코드에 따른 오류 처리 위임<br>
> > 이후 WAS는 서블릿 오류 페이지 찾아 내부 호출 ex) 스프링부트가 기본으로 설정한 error 호출<br>

> 뷰 템플릿 처리<br>
> > `ModelAndView`에 값을 채워 예외에 따른 새로운 오류 화면 렌더링 해 제공<br>

> API 응답 처리<br>
> > `response.getWriter().println({"hello" : "1", "hi" : "2"});` 처럼 HTTP응답 바디에 직접 데이터 넣어주는것 가능, JSON으로 응답하면 API응답 처리 가능<br>

WebConfig - 수정(resolver 등록해주기)
```
@Override
 public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> 
resolvers) {
    resolvers.add(new MyHandlerExceptionResolver());
 }
```
> `configureHandlerExceptionResolvers(...)` 사용하면 스프링이 기본으로 등록하는 `ExceptionResolver`가 제거되므로 주의,<br>
> `extendHandlerExceptionResolvers`를 사용하자<br>

## API 예외 처리 - HandlerExceptionResolver 활용
사용자 정의 예외<br>
UserException
```
public class UserException extends RuntimeException {
  public UserException() {
    super();
  }
  public UserException(String message) {
    super(message);
  }
  public UserException(String message, Throwable cause) {
    super(message, cause);
  }
  public UserException(Throwable cause) {
    super(cause);
  }
  protected UserException(String message, Throwable cause, boolean 
    enableSuppression, boolean writableStackTrace) {
    super(message, cause, enableSuppression, writableStackTrace);
  }
}
```
ApiExceptionController - 예외 추가
```
@Slf4j
@RestController
public class ApiExceptionController {
  @GetMapping("/api/members/{id}")
  public MemberDto getMember(@PathVariable("id") String id) {
    if (id.equals("ex")) {
      throw new RuntimeException("잘못된 사용자");
    }
    if (id.equals("bad")) {
      throw new IllegalArgumentException("잘못된 입력 값");
    }
    if (id.equals("user-ex")) {
      throw new UserException("사용자 오류");
    }
    return new MemberDto(id, "hello " + id);
  }
  @Data
  @AllArgsConstructor
  static class MemberDto {
    private String memberId;
    private String name;
  }
}
```
http://localhost:8080/api/members/user-ex 호출시 UserException이 발생하도록 함<br>
이 예외를 처리하는 UserHandlerExceptionResolver<br>
```
@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {
  private final ObjectMapper objectMapper = new ObjectMapper();
  @Override
  public ModelAndView resolveException(HttpServletRequest request, 
      HttpServletResponse response, Object handler, Exception ex) {
    try {
      if (ex instanceof UserException) {
        log.info("UserException resolver to 400");
        String acceptHeader = request.getHeader("accept");
        response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
        if ("application/json".equals(acceptHeader)) {
          Map<String, Object> errorResult = new HashMap<>();
          errorResult.put("ex", ex.getClass());
          errorResult.put("message", ex.getMessage());
          String result = objectMapper.writeValueAsString(errorResult);
          response.setContentType("application/json");
          response.setCharacterEncoding("utf-8");
          response.getWriter().write(result);
          return new ModelAndView();
        } else {
          //TEXT/HTML
          return new ModelAndView("error/400");
          }
    } catch (IOException e) {
        log.error("resolver ex", e);
     }
     return null;
   }
 }
```
HTTP 요청 해더의 `ACCEPT` 값이 `application/json`이면 JSON으로 오류를 주고 그 외 경우에는 `error/500`에 있는 HTML 오류 페이지를 보여줌<br><br>

WebConfig에 UserHandlerExceptionResolver 추가<br>
`resolvers.add(new UserHandlerExceptionResolver());`<br><br>

지금까지 직접 구현했던 부분을 스프링에서 제공하는 방법으로 더 쉽게 개발하자!<br>
다음장에서 확인하기<br>

## 스프링이 제공하는 ExceptionResolver1
