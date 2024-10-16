## Test Mapper



1. **테스트 환경 설정:**
   - `test` 패키지 아래에 `mapper` 패키지를 생성합니다.
   - `mapper` 패키지 안에 `TestMapper` 인터페이스와 해당하는 구현 XML 파일을 만듭니다.
   - 구현 클래스를 실행할 테스트 클래스를 생성합니다.
   - `root-context.xml`에서 `mybatis-spring:scan`을 사용하여 MyBatis 스캐닝을 구성합니다.
   <br>

2. **인터페이스 메서드 생성:**
   - 인터페이스(`TestMapper`)에서 필요한 데이터베이스 작업을 위한 추상 메서드를 선언합니다.
   - 필요한 파라미터와 반환 타입을 지정합니다. 예를 들어, 게시글 정보를 가져오는 경우 `List`를 반환합니다.
<br>

3. **XML 구현 설정:**
   - XML 파일(`TestMapper.xml`)에서 MyBatis 매퍼 DTD를 지정하고 메서드에 대한 SQL 문을 정의합니다.
   - 인터페이스의 메서드 이름과 XML의 `id` 속성을 일치시킵니다.
   - `resultType`에서는 select 문에 대한 반환 타입을 지정하고, 파라미터가 있을 경우 `parameterType`에 파라미터 타입을 지정합니다.
<br>

4. **테스트 클래스 생성:**
   - `@Autowired`를 사용하여 `TestMapper` 빈을 테스트 클래스에 주입합니다.
   - `@Test` 어노테이션을 사용하여 매퍼 메서드를 실행하는 테스트 메서드를 작성합니다.

     
 ```
 @Test
     public void testGetBoard() {
	List<BoardVO> list = testMapper.getBoard();
	System.out.println(list.size());
	System.out.println(list.toString());
     }
```

   - 필요한 경우 임시 데이터를 제공합니다. 예를 들어, 파라미터가 있는 메서드를 테스트할 때는 임시 데이터를 생성합니다.

 ```
 @Test
    public void testUpdateBoard() {
	BoardVO boardVO = new BoardVO(0,"테스트작성자명","테스트용title","테스트용 content");
	testMapper.updateBoard(boardVO);
    }
```

<br>

5. **테스트 실행:**
   - 테스트 클래스를 마우스 오른쪽 클릭하여 "Run As -> JUnit Test"를 선택하여 테스트를 실행합니다.

  > 이전에 실행한 test는 주석처리 해주어야 데이터가 중복으로 생성 또는 삭제되지 않음

## List, Map Test
1. **HashMap 예시**
 - xml문서
	```
	<select id="getBoard2" resultType="java.util.HashMap" parameterType="int">
		select * 
		from board
		where no = #{no}
	</select>
	```
- 인터페이스
  ```
   public HashMap<String, Object> getBoard2(int no);//파라미터X, 리턴type:List<BoardVO>
  ```
- 실행클래스
  ```
   @Test
	public void testGetBoard2() {
		HashMap<String, Object> map = testMapper.getBoard2(4);
		System.out.println(map.toString());
		System.out.println(map.get("NO"));//키로 값을 출력할 때 대문자 사용
		System.out.println(map.get("WRITERNAME"));//키로 값을 출력할 때 대문자 사용
	}
  ```
  > 결과 : 데이터 각 속성과 값을 키와 값의 형태로 HashMap으로 가져올 수 있음
2. **List<HashMap<지네릭,지네릭>> 예시**

    - xml문서
	```
 	 <select id="selectBoard" resultType="map" parameterType="int">
	 	select * 
	 	from board 
	 	where no between 1 and #{no}
	 </select>
	```
- 인터페이스
  ```
   public ArrayList<HashMap<String, Object>> selectBoard(int no);//=> 키와 값의 쌍으로 이루어진 리스트
  ```
- 실행클래스
  ```
   @Test
	public void testSelectBoard() {
		ArrayList<HashMap<String,Object>> list = testMapper.selectBoard(5);
		System.out.println("글번호 1부터 no번까지"+list);
	}
  ```
  > 결과 : 데이터 각 속성과 값을 키와 값의 형태로 HashMap에 저장하고 그 내용들을 지정한 행의 갯수만큼 가져올 수 있음
3. **파라미터 여러개일 경우 예시**
   - xml문서
	```
 	 <select id="selectBoard2" resultType="java.util.HashMap" parameterType="int">
	 	select * 
	 	from board 
	 	where no between #{sNo} and #{eNo}
	 </select>
	```
- 인터페이스
  ```
   public ArrayList<HashMap<String, Object>> selectBoard2(@Param("sNo") int startNo, @Param("eNo") int endNo);//=> 키와 값의 쌍으로 이루어진 리스트
	
  ```
- 실행클래스
  ```
   @Test
	public void testSelectBoard2() {
		ArrayList<HashMap<String,Object>> list = testMapper.selectBoard2(1,5);
		System.out.println("글번호 " +(list.size()) +list);
	}
  ```
  > 2번과 3번 차이 : 3번에는 검색하고자하는 시작번호와 끝번호를 두개의 파라미터로 값을 받아와서 처리한다.
  <br>
## 파라미터
  > xml문서에서 파라미터의 타입을 작성할때 파라미터가 한개이면 생략이 가능하다<br>
  > 타입명은 MyBatis가 지정한 alias로 입력할 수 있다(사이트 참고)
## 내가 지정한 alias
- root-context에서 mybatis-config.xml를 빈으로 생성
  ```
   <property name="configLocation" value="classpath:/mybatis-config/mybatis-config.xml"/>
  ```
- 실제 mybatis-config.xml 문서에서 alias지정
  ```
   <configuration>
	<!-- alias 직접 지정할 때 -->
	<!-- 존재하지 않는 클래스를 입력하면 에러가 발생(미리 등록하지 않기) -->
	<typeAliases>
		<typeAlias alias="boardVO" type="com.mycom.board.vo.BoardVO"/>
	</typeAliases>
   </configuration>
  ```

- Test xml문서에 적용
  ```
   <update id="updateBoard2" parameterType="boardVO">
		update board
		set title=#{title}, content=#{content}
		where no=3
   </update>
  ```
