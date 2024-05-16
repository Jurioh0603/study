## 프로젝트 생성
1. 자바 버전 17이상만 지원
2. packaging : Jar
3 .Group(회사이름), Artifact(프로젝트이름)과 package가 같아야함
4. ex)Group - com.ourCom, Artifact - app / package - com.ourCom.app
5. next
6. 3.2.5버전
7. dependencies 추가 (나중에 추가 가능) - 
  반드시 추가 해야하는것(Spring Web, Spring Boot Devtools), 추가로 선택한것(Lombok, MySQL Driver, Oracle Driver)
8. finish
<br>

## Spring Boot 구조
DemoApplication - spring 시작점
src/main/java - java파일을 저장하는 공간(Controller, Service, DTO, VO, Entity등)
src/main/resources - java파일 제외한 파일들 공간() - templates : HTML문서 / static : css,JS,image등
application.properties - 환경설정(환경변수, Connection pool, DB, MyBatis등)
src/test/java - test용 코드
build.gradle - 라이브러리, plig-in, 프로젝트 등
dependencies 라이브러리에 기본으로 제공하는 ojdbc, jackson, tomcat등 지정된 버전으로 내장되어있음
<br>

## Spring과 Spring Boot의 차이점
pom.xml에서 하던 작업을 .properties파일에서 작업
boot에서는 jsp,html모두 사용가능 주로 html사용
좀 더 찾아보기

## plug-in 설치
Help - Eclipse Marketplace -> web developer검색(java,web developer 둘다 지원되는걸로 선택) -> 동의하고 finish
만약 없으면 Help - install new software에서 찾아보기
Trust Artifacts 창이 뜨면 모두 선택하고 Trust Selected 클릭 -> Restart Now 클릭

## 

> dependencies 추가시 반영하는 방법<br>
> 프로젝트 우클릭 -> gradle -> refresh gradle
