## 시작
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
