
## JPA로 회원 정보 INSERT하고 조회하기
- Controller
 ```
@Controller
@RequestMapping("/jpa")
public class UserJpaContorller {

	//입력폼 화면
	//요청주소 /jpa/user/addForm
	@GetMapping("user/addForm")
	public String userAssForm() {
		return "/view/jpa/addForm";
	}
	
	@Autowired
	private JPAUserRepository jPAUserRepository;
	
	//입력처리
	//요청주소 /jpa/user/add
	//view페이지에서 form 태그에 입력 처리 요청 주소 작성 <form action="/jpa/user/add"
	@PostMapping("/user/add")
	public String userAdd(JPAUserDTO jPAUserDTO) {
		//1. 파라미터 받기 -> 매개변수 @RequestParam 이용, JPAUserDTO이용
		System.out.println("입력처리 userAdd(JPAUserDTO)" + jPAUserDTO);
		
		JPAUser jpaUser = new JPAUser();
		//jpaUser.setId(null); //0제외 모두 입력 가능
		
		//파라미터로 받은 DTO에서 필요한 데이터를 꺼냄
		String name = jPAUserDTO.getName(); //1번 방법
		jpaUser.setName(name);
		jpaUser.setEmail(jPAUserDTO.getEmail()); //2번 방법
		
		//2. 비즈니스로직 -> DAO -> DB(이전방법)
						//Repository -> Entity
		//jPAUserRepository.save(엔티티 객체);
		
		//생성한 Entity 객체를 제시
		jPAUserRepository.save(jpaUser);
		
		//3. Model
		//4. view -> 임시 리다이렉트 처리
		return "redirect:/jpa/user/list"; 
	}
	
	//목록조회
	//요청주소 /jap/user/list
	//리턴유형 @ResponseBody Iterable<JPAUser> : JPAUser목록정보를 JSON 형태로 리턴 
	@GetMapping("/user/list")
	public @ResponseBody Iterable<JPAUser> userList() {
		//1.파라미터 받기
		//2.비즈니스로직수행->Service->Repository->Entity
		Iterable<JPAUser> jPAUsers = jPAUserRepository.findAll();
		System.out.println("jPAUsers" + jPAUsers.toString());
		//3.Model
		//4.view
		return jPAUserRepository.findAll();
	}
  ```
- Entity class
 ```
package com.ourCom.app.jpa;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class JPAUser {
	@Id //PK역할
	@GeneratedValue(strategy = GenerationType.AUTO) //자동증가
	private Integer id; //JPAUser 테이블의 PK 역할을하는 컬럼으로 매핑
	
	@Column(length = 100) //컬럼크기지정
	private String name; //이름
	@Column(columnDefinition = "TEXT")
	private String email; //이메일

	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
	
}
  ```

- CrudRepository 인터페이스를 상속받는 인터페이스
 ```
package com.ourCom.app.jpa;

import org.springframework.data.repository.CrudRepository;

//Repository(저장소) 생성 문법
//interface JPAUserRepository extends CrudRepository<엔티티타입, 해당엔티티의PK역할필드타입>{}
//Repository에는 CRUD작업을 할 수 있는 메서드가 내장되어 있다


//CRUD작업
//JPAUser 엔티티로 Repository를 생성한다는 뜻.
public interface JPAUserRepository extends CrudRepository<JPAUser, Integer> {

}
  ```

- 회원가입폼
 ```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h3>addForm.html문서</h3>
	<form action="/jpa/user/add" method="post">
		*이름 <input type="text" name="name"/><br/>
		*이메일 <input type="email" name="email"/><br/>
		<input type="submit" value="확인"/>
	</form>
</body>
</html>
  ```

- List 출력

 ```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h3>usreList.html 문서</h3>
	jPAUsers = ${jPAUsers}
</body>
</html>
  ```
