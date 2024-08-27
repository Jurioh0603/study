## JDBC를 이용하여 데이터베이스 연결할 경우
<br>

PreparedStatement를 활용하여 SQL문을 작성하고, ResultSet에 담아서 값을 처리합니다.
<br>
### DAO 예시

 ```
List<BoardVO> list  = jdbcTemplate.query(sql, new Object[]{}, new RowMapper<BoardVO>() {
			public BoardVO mapRow(ResultSet rs, int rowNum) throws SQLException {
				BoardVO boardVO = new BoardVO();
				boardVO.setNo(rs.getInt("no"));
				boardVO.setWriterName(rs.getString("writerName"));
				boardVO.setTitle(rs.getString("title"));
				boardVO.setContent(rs.getString("content"));
				return boardVO;
			}
		});
```

## mybatis를 이용한 경우의 mvc
Mapper 인터페이스와 Mapper를 구현하는 xml을 활용
<br>
<br>
### MyBatis의 DAO
<br>

**MyBatis 연결과 쿼리문 처리의 차이점**

- **SQL문 작성**:
   - MyBatis: Mapper 인터페이스와 연결된 XML 파일에서 쿼리문을 작성합니다.
   - JDBC: 직접 PreparedStatement를 사용하여 SQL문을 작성합니다.

- **파라미터 처리**:
   - MyBatis: `#{} 속성`을 사용하여 SQL문에 파라미터를 치환합니다. {}안에 값을 VO객체 변수명과 반드시 동일해야함
   - JDBC: 직접 PreparedStatement의 매개변수 위치에 값을 설정하여 치환합니다.
     <br>
   - id="구현하고 있는 인터페이스의 메서드명과 동일하게"
   - parameterType="변수의 타입 작성"
   - 만약 파라미터가 여러개일 경우 parameterMap 사용
  <br>
  ### Mapper 인터페이스 예시
  ```
  //목록보기
  public List<BoardVO> getList();
  //글보기
  public BoardVO getBoard(int no);
  ```

  ### xml 문서 예시
   ```
   <!-- 상세보기 -->
	<select id="getList" resultType="com.mycom.board.vo.BoardVO">
		select no, writerName, title, content 
		from board 
		order by no desc
	</select>
	<select id="getBoard" resultType="com.mycom.board.vo.BoardVO" parameterType="int">
		select no, writerName, title, content 
		from board 
		where no=#{no}
	</select>
   ```

- **DAO 역할**:
   - MyBatis: XML 파일이 DAO의 역할을 대신합니다. XML 파일에서 쿼리문과 매핑되는 메서드를 정의합니다.
   - JDBC: DAO 클래스에서 SQL문을 실행하고, 결과를 처리합니다.


<br>
<br>


### MyBatis의 Service

**root-context.xml 설정**:
- 하단 Namespaces의 `mybatis-spring`을 선택합니다.
- `mybatis-spring:scan base-package`을 작성하여 MyBatis의 Mapper 인터페이스가 위치한 패키지를 스캔합니다.

### root-context.xml 예시
 ```
 <mybatis-spring:scan base-package="com.mycom.board.mapper"/>
 <mybatis-spring:scan base-package="com.mycom.*"/>   
 ```

### BoardService 구현클래스 예시
```
//목록보기 요청
	@Override
	public List<BoardVO> getList() {
		//필요시 추가작업
		//기존의 db연동
		List<BoardVO> list = boardMapper.getList();
		return list;
		
		//mybatis의 경우
	}
	//상세보기
	@Override
	public BoardVO getboard(int no) {
		/* 2줄을 아래 한줄로 표현
		 * BoardVO boardVO = boardMapper.getBoard(no); 
		 * return boardVO;
		 */
		return boardMapper.getBoard(no);
	}
```

### BoardController 구현클래스 예시
```
//목록보기 요청
	@GetMapping(value = "/list")
	public String boardList(Model model) {
		List<BoardVO> list = boardService.getList();
		model.addAttribute("list", list);
		return "board/list";
	}
//상세보기 요청
	@GetMapping(value = "detail")
	public String boardDetail(@RequestParam("no") int no, Model model) {
		//목록보기에서 추가작업하고 -> 파라미터 받고 -> 모델작업 -> view 만들어서 출력
		//1.파라미터 받기(@RequestParam("no") int no)
		
		//2.비즈니스 로직
		//파라미터 int no : 상세조회하고 싶은 글번호 
		BoardVO boardVO = boardService.getBoard(no);
		
		//3.Model
		model.addAttribute("boardVO",boardVO);
		
		//4.View
		return "board/detail";
	}
```
