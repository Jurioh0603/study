## Spring Tool Suite (STS) 설치

1. STS 다운로드 페이지에서 sts.jar 파일을 다운로드합니다.
2. 다운로드한 sts.jar 파일을 더블 클릭하여 실행합니다.
3. 설치 위치를 선택한 후 설치를 진행합니다. (보통 기본값인 C:\Program Files\SpringToolSuite4에 설치됩니다)
4. 설치가 완료되면 sts.exe 파일을 실행하여 Spring Tool Suite를 시작합니다.

## Lombok 설치

1. 먼저 Lombok JAR 파일을 다운로드합니다.
2. 다운로드한 JAR 파일을 더블 클릭하여 실행합니다.
3. Install 버튼을 클릭하여 설치를 진행합니다.
4. 설치가 완료되면 sts.exe(Spring Boot 툴)을 선택하고 Install 버튼을 클릭합니다.
5. 설치가 완료되면 sts.exe 폴더에 Lombok 파일이 생성됩니다.

## Lombok의 역할

1. `@Setter`: Setter 메서드를 자동으로 생성합니다.
2. `@Getter`: Getter 메서드를 자동으로 생성합니다.
3. `@NoArgsConstructor`: 매개변수가 없는 기본 생성자를 자동으로 생성합니다.
4. `@AllArgsConstructor`: 모든 필드를 매개변수로 가지는 생성자를 자동으로 생성합니다.
5. `@Data`: equals, hashCode, Getter, Setter, ToString 메서드를 모두 자동으로 생성합니다.
6. `@Builder`: 빌더 패턴을 자동으로 생성합니다.
7. `@ToString`: toString 메서드를 자동으로 생성합니다.

## 프로젝트 생성

1. 자바 버전 17이상만 지원
2. Packaging: Jar
3. Group (도메인 이름 거꾸로 예: com.ourCom), Artifact (프로젝트 이름)과 package가 같아야 함
   - ex) Group - com.ourCom, Artifact - app / package - com.ourCom.app
4. Next
5. Spring Boot 버전 3.2.5 선택
6. Dependencies 추가 (나중에 추가 가능)
   - 반드시 추가해야 하는 것: Spring Web, Spring Boot Devtools
   - 추가로 선택한 것: Lombok, MySQL Driver, Oracle Driver
7. Finish
<br>



## Spring Boot 구조

- DemoApplication: Spring 시작점
- src/main/java: Java 파일을 저장하는 공간 (Controller, Service, DTO, VO, Entity 등)
- src/main/resources: Java 파일 제외한 파일들을 저장하는 공간
  - templates: HTML 문서
  - static: CSS, JS, 이미지 등
- application.properties: 환경 설정 파일 (환경 변수, Connection pool, DB, MyBatis 등)
- src/test/java: 테스트용 코드
- build.gradle: 라이브러리, 플러그인, 프로젝트 설정 등
  - Dependencies: 라이브러리에 기본으로 제공하는 ojdbc, Jackson, Tomcat 등이 지정된 버전으로 내장되어 있음

    <br>

## Spring vs. Spring Boot의 차이점

1. 설정 방식:
   - Spring: XML 또는 Java Config 파일(pom.xml, applicationContext.xml 등)에서 설정
   - Spring Boot: application.properties 또는 application.yml 파일에서 설정

2. 내장 서버:
   - Spring: 외부 서버(Tomcat, Jetty 등)를 따로 설치하고 설정해야 함
   - Spring Boot: 내장 서버(Tomcat, Jetty, Undertow 등)가 기본으로 제공되며, 별도 설치 없이 바로 사용 가능

3. 의존성 관리:
   - Spring: 개발자가 직접 필요한 의존성을 설정하고 관리해야 함(pom.xml 파일 사용)
   - Spring Boot: Starter 패키지를 사용하여 의존성을 자동으로 관리하며, 간편하게 추가할 수 있음

4. 실행 환경:
   - Spring: WAR 파일로 패키징하여 외부 서버에 배포
   - Spring Boot: JAR 파일로 패키징하여 내장 서버에서 실행하거나, Docker와 같은 컨테이너에 직접 배포 가능

5. 뷰 템플릿:
   - Spring: 주로 JSP 또는 Thymeleaf와 같은 서버 사이드 템플릿 엔진 사용
   - Spring Boot: 주로 정적 HTML 파일 사용, 뷰 템플릿 엔진을 사용할 수 있지만 권장되지 않음

6. 자동 설정:
   - Spring: 개발자가 모든 설정을 수동으로 해야 함
   - Spring Boot: 자동 설정(auto-configuration) 기능을 제공하여 개발자가 작성할 코드의 양을 줄여줌

7. 생산성:
   - Spring: 초기 설정 및 프로젝트 구성에 시간이 소요될 수 있음
   - Spring Boot: 기본 설정이 이미 되어 있으므로 개발자는 바로 개발에 집중할 수 있음
<br>


## 플러그인 설치

1. Eclipse에서 Help 메뉴를 엽니다.
2. Eclipse Marketplace를 선택합니다.
3. 검색 상자에 "web developer"를 입력하여 검색합니다. (java, web developer 둘 다 지원되는 것을 선택)
4. 검색 결과 중 필요한 플러그인을 선택한 후 Install 버튼을 클릭합니다.
5. 설치 과정을 따라가며 동의하고 설치를 완료합니다.
6. Trust Artifacts 창이 나타나면 모두 선택하고 Trust Selected를 클릭합니다.
7. Eclipse를 재시작하려면 Restart Now를 클릭합니다.

> dependencies 추가시 반영하는 방법<br>
> 프로젝트 우클릭 -> gradle -> refresh gradle
<br>
## problems 창에서 경고 표시 처리하기

1. Eclipse에서 Problems 창을 엽니다. (주로 하단에 위치합니다)
2. Problems 창에 표시된 경고 아이콘을 마우스 우클릭합니다.
3. 나타나는 메뉴에서 해결하고자 하는 옵션을 선택합니다. (예: Quick Fix 또는 해당 리소스 열기)
4. 선택한 옵션을 실행하여 경고를 해결합니다.
<br>

## Spring Boot test

**패키지명 작성하는 방법**
1. com.ourCom.app.controller
2. com.ourCom.app.vo 등등 app 하위로 만들기

<br>

**컨트롤러 클래스 만들기**
1. @ResponseBody 어노테이션은 Spring 프레임워크에서 컨트롤러가 HTTP 응답의 본문(body)으로 반환할 데이터를 나타냄 <br>
2. 이 어노테이션을 사용하면 컨트롤러가 뷰 템플릿을 통해 HTML 페이지를 렌더링하는 대신, 직접 응답의 본문으로 데이터를 반환
<br>

> **포트번호 에러**<br>
> @Controller와 @GetMapping와 @ResponseBody을 통해 컨트롤러 생성 후 실행 -> 포트번호 충돌<br>
> application.properties에서 포트번호 수정'server.port=8091' -> 실행 -> 주소입력 화면 결과 확인
<br>
## jsp 사용하기 위한 설정 (maven Repository 사이트)
1. **Tomcat Embeded Jasper 빌드하기**
   - 내장 되어있는 버전과 동일한 버전 다운 (10.1.20) -> gradle 중에서 아무거나 복사 build.gradle에 붙여넣기
2. **jstl**
   - jstl 중 가장 다운로드 수가 많은 버전을 선택하여 Tomcat Embeded Jasper와 동일한 방법으로 의존성을 추가
3. **gradle반영**
   - 프로젝트 우클릭 -> gradle -> gradle refresh
5. **폴더 생성**
   - 스프링과 동일하게 src/main 밑에 webapp/WEB-INF/views 폴더 모두 생성
6. **jsp 페이지 생성**
7. **application.properties 에 다음코드를 추가하여 jsp 파일을 읽어들일 수 있도록 경로를 지정**
   - spring.mvc.view.prefix=/WEB-INF/views<br>
   - spring.mvc.view.suffix=.jsp
<br>

## 웹으로 Spring Boot 프로젝트 생성
> 아래주소에서 프로그램으로 생성한것 처럼 동일하게 설정하고 생성할 수 있음
<https://start.spring.io/>
<br>
