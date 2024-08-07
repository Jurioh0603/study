# Type 이해하기
## 자료형
### 원시 형태(Primitive)
- 숫자(`Number`) : 숫자형으로 정수나 실수를 포함
- 문자열(`String`) : 문자들의 나열
- 불리언(`Boolean`) : 참/거짓
- 정의되지 않음(`undefined`) : 값이 정해지지 않음을 표현
- 없음 : `null`

### 객체 형태(Object)
- 시간(`Data`)
- 배열(`Array`) : 인덱스 기반으로 값을 순서대로 삽입할 수 있는 객체 형태(0부터 시작)
	-> 값의 삽입 / 제거
	- `length` : 배열의 크기 리턴
 	- `push()` : 배열에 값 추가
  	- `unshift()` : 배열에 새로운 값 0번 인덱스로 추가
  	- `pop()` : 배열의 마지막 값 리턴하며 제거
  	- `shift()` : 배열의 처음 값 리턴하며 제거
    <br>
- 객체(`Object`) : 객체 기반의 언어, 여러 속성이나 값 로직을 여러 프로퍼티로 묶어 놓은 것
	- `property` : 객체의 구성 멤버, `key-value` 형태의 한 쌍으로 구성
 	- `method` : 객체 프로퍼티의 `Value`로 함수가 정의된 형태
    

### 리터럴 선언
= 코드상에서 값을 직접 명시해 선언하거나 할당하는 것으로 자료형에 따라 리터럴 선언 문법이 다름

### 객체 자료형 예시
`Object Type - property`
```
var objects = {
   studentName: '철수',
   id: '202010000'
 //propertyName: value
}
```
객체 프로퍼티에 접근하거나 값을 저장하고 싶으면 `.(dot)` 기호를 사용한다.

```
var objects = {userName: "인프런", age: 10};
objects.userName // "인프런", userName 값에 접근
objects.age = 12 // age 값을 12로 변경
objects.online = true // online 값을 true로 지정
```

### 배열형 
```
var array = [0,10,20];
array[0]; // 0
array[2]; // 20
```

<br>
<br>

# 변수와 연산
## 변수
변수의 선언 :`variable`의 약어인 `var` 키워드와 변수의 이름을 선언
변수에 값을 할당 : 대입 연산자 `(=)`을 사용하여 값을 대입
### 변수명 규칙
- 하이픈(-) 사용 불가
- 띄어쓰기 사용 불가
- 첫 글자로 숫자 사용 불가
- 예약어 사용 불가

### 산술 연산자
- 덧셈 : +, ++
- 뺄셈 : -, --
- 곱셈 : *
- 나눗셈 : /
- 나머지 : %
- 복합 연산 : +=, -=, *=, /=

### 비교 연산자
#### 비교 연산자
두 값을 비교하기 위해 사용, 참 거짓 결과가 나옴
- 동등 : ==
- 부등 : !=
- 일치 : ===
- 좌변이 큼 : >
- 좌변이 크거나 같음 : >=
- 우변이 큼 : <
- 우변이 크거나 같음 : >=

#### 논리 연산자
참 또는 거짓 연산시 사용, 비교문 조합해 복잡한 조건문 만듬
- 그리고(AND) : &&
- 또는(OR) : ||
- 부정(NOT) : !

#### 기타 연산자
- 삼항 연산자
- 단항 연산자(delete, typeof)
- 비트 연산자(&, |, ^, ~, <<, >>, >>>)

<br>

### 기본적인 문구 출력 및 입력 방법
```
<script>
console.log('개발자 도구 콘솔 창에 보여짐');
alert('alert창에 띄워짐');
prompt('입력하는 용도!')
</script>
```
