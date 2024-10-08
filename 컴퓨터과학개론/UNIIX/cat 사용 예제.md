# cat 명령이란
파일을 순서대로 읽고 그 내용을 순서대로 출력할 때 쓰이는 명령이다.
<br><br>

# 사용 예제
- 파일 내용 출력
```
cat FILE
```
<br><br>
- 파일 생성
```
cat > FILE
```
<br><br>
- 라인마다 번호 출력
```
cat -n FILE
```
<br><br>
- 라인 끝에 번호 출력
```
cat -E FILE
```
<br><br>
- 탭(TAP)을 ^I로 바꿔서 출력
```
cat -T FILE
```
<br><br>
- 반복된 공백 라인 무시
```
cat -s FILE
```
<br><br>
- 파일 복사, 합치기, 추가
```
cat FILE > OUT
```
<br><br>
- 파일 사이에 내용 추가
```
cat FILE1 - FILE2 > OUT
```
<br><br>
- 파일 내용을 페이지 단위로 출력
```
cat FILE | more
```
<br><br>
- 파일 내용 필터링
```
cat FILE | grep "STR"
```
<br><br>
- 모든 파일 내용 출력
```
cat *
```
<br><br>
- 특정 확장자를 가진 파일 내용 출력
```
cat *.txt
```
