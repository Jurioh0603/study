## 프로젝트 생성

### Project : Gradle - Groovy
### Language : Java
### Spring Boot : 3.2.5

### Project Metadata
- **Group** : com.ourCom
- **Artifact** : app
- **Name** : app
- **Description** : our project for Spring Boot
- **Package name** : com.ourCom.app
- **Packaging** : Jar
- **Java** : 17

### Dependencies
- Spring Web
- Lombok
- Thymeleaf
- Spring Boot DevTools
- Oracle Driver
- MySQL Driver

---

### Thymeleaf란?
- **용도**: 백엔드 서버에서 HTML을 동적으로 렌더링하는 용도
- **특징**:
  - 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 '네츄럴 템플릿'
  - 스프링의 다양한 기능을 편리하게 사용할 수 있음

---

### 프로젝트 설정 및 가져오기

1. **프로젝트 생성 후 압축 해제**:
   - 프로젝트를 생성한 후, 워크스페이스에 압축을 풉니다.

2. **프로젝트 가져오기**:
   - `import` -> `Gradle` -> `Existing Gradle Project` -> 프로젝트 파일 선택

---

## thymeleaf 사용

html파일에 반드시 아래 코드 작성해야함
<html xmlns:th="http://www.thymeleaf.org">

**변수표현식**
- html 예시
 ```
   <!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h3>ex01.html문서</h3>

	<hr>
	<div th:text="세종대왕"></div>
	<div th:text="${'thymeleaf연습중'}"></div>
	<div>[[${'thymeleaf연습중입니다'}]]</div>
	<!-- thymeleaf는 서버측 코드이기 때문에 웹상에서 사용유무를 알 수 없음 -->
</body>
</html>
  ```

**비교 동등**
 ```
	<div th:text=${100==1}></div>
	<div th:text=${100==100}></div>
	<div>[[${100==100}]]</div>
	<div>[[${100!=100}]]</div>
  ```

**변수 사용**
 ```
	*변수 with="변수명=값"
	<div th:with="a=10">[[${a}]]</div><!-- 지역변수의 성격 -->
	<div>[[${a}]]</div><!-- 출력되지 않음 -->
	<div th:with="a=100,b=123">
		[[${a}]]<br/>
		[[${b}]] <!-- 메서드영역의 지역변수적 성격 -->
	</div>
  ```

### 조건문
**if문, 연결연산자**

 ```
	<h3>thymeleaf의 조건문</h3>
	id문, 연결연산자
	<div th:with="a=100">
		<div>변수a의 값:[[${a} + '이다.']]</div>
		<div th:if="${a==100}">변수a의 값:[[${a} + '은 100과 일치합니다.']]</div>
		<div th:unless="${a==123}">unless:[[${a} + '은 100과 일치하지 않을때만 출력되어요.']]</div>
	</div>
	<if test="조건"></if>
  ```

**3항연산자**

 ```
	*3항연산자 ${ 조건 ? 참일 경우 : 거짓일경우 }
	<div th:with="id='admin'">
		변수 id값: [[${id}]]<br/>
		<span>[[${ id =='admin' ? 'admin입니다' : 'admin 아닙니다' }]]</span><br/>
		<span>[[${ id =='user' ? 'admin입니다' : 'user가 아닙니다' }]]</span>
	</div>
  ```

### 반복문
**each**
- Controller
```
	//반복문연습
	@GetMapping("/view/ex03")
	public String ex03(Model model) {
		ArrayList<String> list = new ArrayList<String>();
		list.add("user1");
		list.add("user2");
		list.add("user3");
		list.add("user4");
		list.add("user5");
		model.addAttribute("list",list);
		return "view/ex03";
	}
  ```
- html
 ```
	<h3>thymeleaf의 반복문</h3>
	th:each="변수명 : ${목록변수(컨트롤러에서 정한 모델속성명)}"<br/>
	[[${변수명}]] 또는 th:text=${변수명}
  *List&lt;String&gt;
	<table border="1">
		<thead>
			<tr>
				<th>이름</th>
			</tr>
		</thead>
		<tbody>
			<tr th:each="temp : ${list}">
				<td>[[${temp}]]</td>
			</tr>
		</tbody>
	</table>
  ```

**Map 지네릭을 가진 List**
- Controller
```
	//반복문연습
	@GetMapping("/view/ex03")
	public String ex03(Model model) {
		//List<Map<K,V>>
		List<Map<String,String>> mapList = new ArrayList<Map<String,String>>();
		Map<String,String> map1 = new HashMap<String,String>();
		map1.put("name", "홍1");
		map1.put("nickName", "홍별명1");
		mapList.add(map1);
		
		Map<String,String> map2 = new HashMap<String,String>();
		map2.put("name", "홍2");
		map2.put("nickName", "홍별명2");
		mapList.add(map2);
		
		Map<String,String> map3 = new HashMap<String,String>();
		map3.put("name", "홍3");
		map3.put("nickName", "홍별명3");
		mapList.add(map3);
		model.addAttribute("mapList",mapList);
		return "view/ex03";
	}
  ```
- html
 ```
	*List&lt;Map&gt;
	mapList = [[${mapList}]]
	<br/><br/>
	<table border="1">
		<thead>
			<tr>
				<th>name</th>
				<th>nickName</th>
			</tr>
		</thead>
		<tbody><!-- 두가지 방식으로 출력 -->
			<tr th:each="map : ${mapList}">
				<td th:text="${map.name}">name</td>
				<td>[[${map.nickName}]]</td>
			</tr>
		</tbody>
	</table>
  ```

**VO객체활용**
- Controller
```
	//반복문연습
	@GetMapping("/view/ex03")
	public String ex03(Model model) {
		//List<SampleVO>
		List<SampleVO> voList = new ArrayList<SampleVO>();
		voList.add(new SampleVO("김1","김별명1"));
		voList.add(new SampleVO("김2","김별명2"));
		voList.add(new SampleVO("김3","김별명3"));
		model.addAttribute("voList", voList);
		return "view/ex03";
	}
  ```
- html
 ```
	*List&lt;SampleVO&gt;
	voList = [[${voList}]]
	<br/><br/>
			<ul th:each="vo : ${voList}">
				<li th:text="${vo.name}">name</li>
				<li>[[${vo.nickName}]]</li>
			</ul>
  ```

**반복문 안에 if문이 있을 때**
- Controller
```
	//반복문연습
	@GetMapping("/view/ex03")
	public String ex03(Model model) {
		List<SampleVO> voList = new ArrayList<SampleVO>();
		voList.add(new SampleVO("김1","김별명1",10));
		voList.add(new SampleVO("김2","김별명2",20));
		voList.add(new SampleVO("김3","김별명3",31));
		model.addAttribute("voList", voList);
		return "view/ex03";
	}
  ```

- html
 ```
	<ul th:each="vo : ${voList}" th:if="${vo.age%2==0}">
		<li>
			<div></div>[[${vo.name}]] / [[${vo.nickName}]] / [[${vo.age}]]
		</li>
	</ul>
  ```

**status(상태코드값)**
 ```
	<ul th:each="vo, status : ${voList}">
		<li>
			[[${vo.name}]] / [[${status}]]
		</li>
	</ul>
  ```
-결과 

> size : 0부터 1씩증가
> size : 데이터의 갯수, VO 객체 하나의 행에 있는 데이터 갯수
> count : 1부터 1씩증가, 데이터를 조회할 때 활용할 수 있음 ex-3번 count부터 줄바꿈하기
> current : 각 데이터 내용

 ```
	김1 / {index = 0, count = 1, size = 3, current = SampleVO(name=김1, nickName=김별명1, age=10)}
	김2 / {index = 1, count = 2, size = 3, current = SampleVO(name=김2, nickName=김별명2, age=20)}
	김3 / {index = 2, count = 3, size = 3, current = SampleVO(name=김3, nickName=김별명3, age=31)}
  ```

**div대신 block 사용해도 같은 결과**
 ```
	<th:block th:each="vo, status : ${voList}">
		<li>
			[[${vo.name}]] / [[${status}]]
		</li>
	</th:block>
  ```

**builder 메서드 체이닝 기법**
> 객체의 메서드를 연속적으로 호출하는 방식

- vo 객체에 Builder 어노테이션
- Controller에 Builder를 이용해 객체를 생성하고 필드값을 설정

 ```
	//Builder 객체를 이용해서 Builder 객체생성 후 필드값 설정
	SampleVO sampleVO5 = SampleVO.builder()
			.name("김5").nickName("김별명").age(39).build();
	voList.add(sampleVO5);
  ```

**주소 파라미터 값 받아오기 (주소 - th:href="@{주소}")**
- Controller
 ```
	@GetMapping("/view/ex04_1/{category}/{no}")
		public String ex04_1(@PathVariable("category") String cate
							,@PathVariable("no") String no) {
			System.out.println("파라미터 category =" + cate);
			System.out.println("파라미터 no=" + no);
			return "view/ex04_1";
		}
  ```
- html
 ```
	문법 태그 th:href="@{경로}" /절대경로   ./../상대경로
	<a th:href="/view/ex04_1/it/1">일반 a 요소(ex01.html응답문서)</a><p/>
  ```
- 결과
 ```
	파라미터 category =it
	파라미터 no=1
  ```

**thymeleaf 이용해서 위에서 받아온 주소 파라미터 데이터 넘기기**
- Controller
 ```
	@GetMapping("/view/ex04_2")
		public String ex04_2() {
			return "view/ex04_2";
		}
  ```
- html
 ```
	<ul>
		<li th:each="vo : ${voList}"><!-- 동적으로 데이터 넘기기 -->
			<!-- <a th:href="@{/view/ex04_2/{변수명}/{변수명}"></a> -->
			<a th:href="@{/view/ex04_2/{n}/{a}(n=${vo.name}, a=${vo.age})}">[[${vo.name}]] / [[${vo.age}]]</a><p/>
		</li>
	</ul>
  ```
- 주소 결과
   ```
	http://localhost:8091/view/ex04_2/김1/40
   	http://localhost:8091/view/ex04_2/김2/41
  ```

> 주소에 값이 따라옴 값으로 다음페이지에서 처리해줄 수 있음
<br>

**위 결과를 받아서 처리하는 방법**
1. 쿼리 파라미터 타입 : 변수를 경로로 사용
- Controller
 ```
	@GetMapping("/view/ex04_2/{category}/{no}")
		public String ex04_2(@PathVariable("category") String cate
					,@PathVariable("no") String no) {
			return "view/ex04_2";
		}
  ```
- html
 ```
	[[${category}]] / [[${no}]]
  ```

- 요청주소 및 결과
 ```
	http://localhost:8091/view/ex04_2/김1/40  => (view페이지) 김1 / 40
	http://localhost:8091/view/ex04_2/김2/41  => (view페이지) 김2 / 41
  ```

2. 쿼리스트링 타입 : thymleaf href 템플릿
- Controller
 ```
	@GetMapping("/view/ex04_3")
		public String ex04_3(@RequestParam("category") String cate
					,@RequestParam("no") String no) {
			return "view/ex04_3";
		}
  ```
- html
 ```
	<ul>
		<li th:each="vo : ${voList}"><!-- 동적으로 데이터 넘기기 -->
			<a th:href="@{/view/ex04_3(name=${vo.name}, age=${vo.age})}">[[${vo.name}]] / [[${vo.age}]]</a><p/>
		</li>
	</ul>
  ```

- 요청주소
 ```
	http://localhost:8091/view/ex04_3?n=김1&a=40
  ```

> 반드시 [[${안에 입력해야만 변수로 인식한다}]] 또는 태그내에서 "${안에 작성해야한다}"
<br>

> 참고사이트
> https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html
