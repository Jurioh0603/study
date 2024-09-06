## JPA 소개
컬렉션에 저장하듯이 객체를 데이터 베이스에 편리하게 저장할 수 있게 해주는 JPA<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/a0b2ea84-00b4-4171-9b88-54db8968232b)<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/57321226-1794-4595-8c46-3c74fcda9eed)<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/1817e10e-efce-43e3-9eef-bcd3522ab94a)<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/ada7e0cf-590e-439e-9203-99a724587bfe)<br><br>

#### JPA 사용 이유 <br>
- SQL 중심적인 개발에서 객체 중심으로 개발<br>
- 생산성<br>
- 유지보수<br>
- 패러다임의 불일치 해결<br>
- 성능<br>
- 데이터 접근 추상화와 벤더 독립성<br>
- 표준<br><br>

#### JPA CRUD<br>
• 저장: `jpa.persist(member)`<br>
• 조회: `Member member` = `jpa.find(memberId)`<br>
• 수정: `member.setName(“변경할 이름”)`<br>
• 삭제: `jpa.remove(member)`<br><br>

#### 유지보수 측면<br>
- 기존 : 필드 변셩 시 모든 SQL 수정<br>
- JPA : 필드만 추가하면 SQL은 JPA 가 처리<br><br>

#### JPA와 패러다임의 불일치 해결(중요)<br>
1.JPA와 상속<br>
2.JPA와 연관관계<br>
3.JPA와 객체 그래프 탐색<br>
4.JPA와 비교하기<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/da553849-48d4-4474-9079-2b89ad6b7d42)<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/da9e75b8-7c22-430a-8d1b-e6cbc4523b90)<br>
상속 관계에 있는 테이블까지 insert 해줌<br><br>

![image](https://github.com/Jurioh0603/study/assets/148063470/f0758721-d366-493c-a893-a76fd1e61b68)<br>
조인처리를 알아서 해줌<br><br>

![image](https://github.com/Jurioh0603/study/assets/148063470/f5ae7940-f52a-41ab-a021-1ea9072bed88)<br>

![image](https://github.com/Jurioh0603/study/assets/148063470/1a376e09-d768-4423-a550-90d0e2a52b91)<br>
신뢰할 수 있는 엔티티, 계층임<br><br>

#### JPA 비교하기<br>
```
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);
member1 == member2; //같다.
```
동일한 트랜젝션에서 조회한 엔티티는 같음을 보장함<br><br>

#### JPA 성능 최적화 기능<br>
1. 1차 캐시와 동일성(identity) 보장<br>
- 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상<br>
- DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장<br>
> SQL은 한번만 실행되고 같은 트랜젝션의 요청이 있다면 캐시에서 찾아 값을 줌<br>
> 즉 SQL은 1번만 실항하게 됨<br><br>
2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)<br>
- 트랜잭션을 커밋할 때까지 INSERT SQL을 모음<br>
- JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송<br>
```
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
- UPDATE, DELETE로 인한 로우(ROW)락 시간 최소화<br>
- 트랜잭션 커밋 시 UPDATE, DELETE SQL 실행하고, 바로 커밋<br>
- (좀 어려운 개념이라 이후 자세히 설명)<br>
```
transaction.begin(); // [트랜잭션] 시작
changeMember(memberA); 
deleteMember(memberB); 
비즈니스_로직_수행(); //비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다. 
//커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

<br>

3. 지연 로딩(Lazy Loading)<br>
- 지연 로딩: 객체가 실제 사용될 때 로딩<br>
- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회<br>
![image](https://github.com/Jurioh0603/study/assets/148063470/6f24e8db-00cd-4085-874a-387ba14bf33c)<br><br>
> 실무에서는 지연 로딩으로 우선 해놓고 한번에 처리되어야 성능이 좋아질것 같다 판단될 때 즉시 로딩 사용
