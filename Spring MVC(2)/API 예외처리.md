## 시작
![image](https://github.com/Jurioh0603/study/assets/148063470/e8de1756-883d-49c4-a2b0-9702ea41b3d2)<br>
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
스프링 부트가 기본으로 제공하는 `ExceptionResolver`<br>
`HandlerExceptionResolverComposite`에 다음 순서로 등록<br>
1. `ExceptionHandlerExceptionResolver`<br>
2. `ResponseStatusExceptionResolver`<br>
3. `DefaultHandlerExceptionResolver` -> 우선 순위가 가장 낮음<br><br>

`ResponseStatusExceptionResolver` 예외에 따라 HTTP 상태코드를 지정<br>
1. `@ResponseStatus` 가 달려있는 예외<br><br>
BadRequestException 클래스
```
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
public class BadRequestException extends RuntimeException {} // RuntimeException 오류가 발생하는 클래스 
```
> 이때 `BAD_REQUEST` 말고도 다양한 에러가 상수로 정의되어있어 사용할 수 있음<br>
> `ResponseStatusExceptionResolver` 코드를 확인해보면 이전에 직접 개발할 때처럼 ` response.sendError(statusCode, resolvedReason)`를 호출하는 것 확인할 수 있음<br>
> `sendError(400)`을 호출했기 때문에 WAS에서 다시 오류 페이지 `(/error)`를 내부 요청함<br><br>

ApiExceptionController - 추가
```
@GetMapping("/api/response-status-ex1")
public String responseStatusEx1() {
  throw new BadRequestException(); //위에서 만든 오류 던짐
}
```

메시지 기능 제공<br>
BadRequestException 클래스 수정
```
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException {}
```
messages.properties 파일
```
error.bad=잘못된 요청 오류입니다.
```

결과는 properties 파일 내 메시지 출력됨<br><br>

2. ResponseStatusException 예외<br><br>
`@ResponseStatus`는 직접 만든 예외가 아니거나 개발자가 직접 변경할 수 없는 예외 적용할 수 없음 ex)수정할 수 없는 라이브러리의 예외 코드 같은 곳<br>
이때 ResponseStatusException 예외 사용하면 됨<br><br>

ApiExceptionController - 추가
```
@GetMapping("/api/response-status-ex2")
public String responseStatusEx2() {
  throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
}
```
지정한 오류`HttpStatus.NOT_FOUND`에 진짜 발생한 오류 `new IllegalArgumentException()`를 지정할 수 있음<br>

## 스프링이 제공하는 ExceptionResolver2
`DefaultHandlerExceptionResolver`는 스프링 내부에서 발생하는 스프링 예외를 해결<br>
파라미터 바인딩 시점에 타입이 맞지 않을 때 내부에서 `TypeMismatchException`가 발생하는데 이 때 그냥 두면 서블릿 컨테이너까지 오류가 올라가 500 오류 발생<br>
스프링에서 `DefaultHandlerExceptionResolver`는 HTTP 상태 코드 400오류로 변경해줌<br><br>

코드를 보면 다음과 같음<br>
` response.sendError(HttpServletResponse.SC_BAD_REQUEST)` (400)<br>
결국 `response.sendError()`를 통해 해결<br><br>

ApiExceptionController - 추가
```
 @GetMapping("/api/default-handler-ex")
 public String defaultException(@RequestParam Integer data) {
   return "ok";
 }
```

이때 `data`에 문자 입력하면 `TypeMismatchException`이 발생<br>
실행해보면 400 상태 코드 인것 확인 가능<br><br>

정리<br>
> ResponseStatusExceptionResolver -> HTTP응답 코드 변경(직접 설정해줘야 함)<br>
> DefaultHandlerExceptionResolver -> 스프링 내부 예외 처리(스프링이 이미 설정해줌)<br><br>

그러나 `ModelAndView`를 반환하는 것은 API에 맞지 않음<br>
이 문제를 해결하기 위해 스프링은 `ExceptionHandlerExceptionResolver`를 제공<br>

## API 예외 처리 - @ExceptionHandler
API 예외 응답을 위해 `HttpServletResponse`에 직접 Map형태로 데이터를 넣어주어야 했음, 이는 효율적이지 않음<br>
`@ExceptionHandler`라는 애노테이션을 사용해 편리하게 API 예외를 해결해줌(실무에서 대부분 사용하는 방법)<br><br>

예외 발생시 API 응답으로 사용하는 객체 정의<br>
ErrorResult
```
@Data
 @AllArgsConstructor
 public class ErrorResult {
   private String code;
   private String message;
 }
```
ApiExceptionV2Controller
```
@Slf4j
@RestController
public class ApiExceptionV2Controller {
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  @ExceptionHandler(IllegalArgumentException.class)
  public ErrorResult illegalExHandle(IllegalArgumentException e) {
    log.error("[exceptionHandle] ex", e);
    return new ErrorResult("BAD", e.getMessage());
  }
  @ExceptionHandler
  public ResponseEntity<ErrorResult> userExHandle(UserException e) {
    log.error("[exceptionHandle] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
  }
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  @ExceptionHandler
  public ErrorResult exHandle(Exception e) {
    log.error("[exceptionHandle] ex", e);
    return new ErrorResult("EX", "내부 오류");
  }
  @GetMapping("/api2/members/{id}")
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
설명<br>
> `restController` 애노테이션을 달아서 ResponseBody가 생성되도록 해 JSON 객체로 반환할 수 있도록 함<br>
```
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegalExHandle(IllegalArgumentException e) {
  log.error("[exceptionHandle] ex", e);
  return new ErrorResult("BAD", e.getMessage());
}
```
-> `@ExceptionHandler`를 선언, ()안에 해당 컨트롤러에서 처리하고 싶은 예외 지정<br>

우선순위
```
@ExceptionHandler(부모예외.class)
public String 부모예외처리()(부모예외 e) {}
@ExceptionHandler(자식예외.class)
public String 자식예외처리()(자식예외 e) {}
```
예를 들어 자식 예외가 발생하면 더 구체적인 자식 예외가 먼저 처리됨<br>

여러가지 예외 설정
```
 @ExceptionHandler({AException.class, BException.class})
 public String ex(Exception e) {
    log.info("exception e", e);
 }
```

예외 생략
```
 @ExceptionHandler
 public ResponseEntity<ErrorResult> userExHandle(UserException e) {}
```

`@ExceptionHandler`에는 마치 스프링 컨트롤러의 파라미터 응답처럼 다양한 파라미터와 응답 지정 가능 (링크 참고)<br>
<https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annexceptionhandler-args><br>

IllegalArgumentException 처리
```
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegalExHandle(IllegalArgumentException e) {
  log.error("[exceptionHandle] ex", e);
  return new ErrorResult("BAD", e.getMessage());
}
```
실행 흐름<br>
> 컨트롤러 호출 결과 `IllegalArgumentException` 예외가 컨트롤러 밖으로 던져짐<br>
> 예외 발생해 `ExceptionResolver`가 작동, 우선순위 높은 `ExceptionHandlerExceptionResolver`가 실행됨<br>
> `ExceptionHandlerExceptionResolver`는 `@ExceptionHandler`가 있는지 찾음<br>
> `illegalExHandle()`을 실행, `@RestController`이므로 `illegalExHandle()`에도 `@ResponseBody가 적용`<br>
> 따라서 HTTP 컨버터 사용되고 응답이 JSON으로 반환됨<br>
> `@ResponseStatus(HttpStatus.BAD_REQUEST)`를 지정했으므로 HTTP 상태코드 400으로 반환(지정하지 않으면 resolver가 잘 해결했다고 인식해서 200번대로 출력됨)<br><br>

직접 만든 UserException 처리
```
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {
  log.error("[exceptionHandle] ex", e);
  ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
  return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
```
설명<br>
> 예외 따로 지정 안했으니 파라미터에 있는 `UserException` 적용<br>
> `ResponseEntity` 사용해 HTTP 메시지 바디에 직접 응답<br>
> `ResponseEntity` HTTP응답 코드를 프로그래밍 해 동적으로 변경 불가<br>
> `@ResponseStatus` 는 애노테이션이므로 HTTP 응답코드를 동적으로 변경 불가<br><br>

기본 Exception 처리
```
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
@ExceptionHandler
public ErrorResult exHandle(Exception e) {
  log.error("[exceptionHandle] ex", e);
  return new ErrorResult("EX", "내부 오류");
}
```
설명<br>
> `RuntimeException`은 `Exception`의 자식 클래스이기 때문에 이 메서드 호출됨<br>
> `@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)`로 HTTP 상태 코드를 500으로 응답<br><br>

HTML 오류 화면 응답
```
@ExceptionHandler(ViewException.class)
public ModelAndView ex(ViewException e) {
  log.info("exception e", e);
  return new ModelAndView("error");
}
```
위와 같이 오류 화면 html 도 응답할 수 있음<br>
다른 컨트롤러에도 적용하려면 내용을 또 작성해야하는데 이를 해결해주는 방법은 다음장에서 알아보자<br><br>

## API 예외 처리 - @ControllerAdvice
예외 처리 코드와 컨트롤러 코드를 분리<br>
ExControllerAdvice
```
@Slf4j
@RestControllerAdvice
public class ExControllerAdvice {
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  @ExceptionHandler(IllegalArgumentException.class)
  public ErrorResult illegalExHandle(IllegalArgumentException e) {
    log.error("[exceptionHandle] ex", e);
    return new ErrorResult("BAD", e.getMessage());
  }
  @ExceptionHandler
  public ResponseEntity<ErrorResult> userExHandle(UserException e) {
    log.error("[exceptionHandle] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
  }
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  @ExceptionHandler
  public ErrorResult exHandle(Exception e) {
    log.error("[exceptionHandle] ex", e);
    return new ErrorResult("EX", "내부 오류");
  }
}
```
ApiExceptionV2Controller 코드에 있는 @ExceptionHandler 모두 제거
```
 @Slf4j
 @RestController
 public class ApiExceptionV2Controller {
    @GetMapping("/api2/members/{id}")
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

@ControllerAdvice<br>
대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler`, `@InitBinder` 기능을 부여해주는 역할을 함<br>
대상을 지정하지 않으면 모든 컨트롤러에 적용됨(글로벌)<br>
`@RestController`는 `@ControllerAdvice`와 같고, `@ResponseBody`가 추가되어 있음<br>
`@Controller`, `@RestController`의 차이와 같음<br><br>

대상 컨트롤러 지정 방법
```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, 
AbstractController.class})
public class ExampleAdvice3 {}
```
> `ExampleAdvice1`은 `@RestController`라는 애노테이션이 달린 컨트롤러에 적용<br>
> `ExampleAdvice2`는 지정한 패키지 내 컨트롤러에 적용<br>
> `ExampleAdvice3`은 특정 컨트롤러 클래스를 지정한것<br><br>

**결론 : `@ExceptionHandler`와 `@ControllerAdvice`를 조합하면 예외를 깔끔히 해결할 수 있음**<br>
