500에러 String 타입에서 Object타입으로 변환 안됨 -> 어디서 안되는건지 syso로 찍어봄 items에서 안되는것 확인 -> instanceof로 해결<br>
500에러([dispatcherServlet] in context with path [] threw exception [Request processing failed: java.lang.NullPointerException: Cannot invoke "java.lang.CharSequence.toString()" because "s" is null] with root cause
java.lang.NullPointerException: Cannot invoke "java.lang.CharSequence.toString()" because "s" is null	at java.base/java.lang.String.contains(String.java:2858)) <br>
데이터가 화면에 안뜸 -> 예외 처리해주었더니 해결<br>
데이터 디비 저장하다가 만난 에러<br>
There is no getter for property named 'overview' in 'class com.triplan.planner.tour.dto.Api' dto에 있는 컬럼명이 하나 수정이 안되어있어서 수정함<br>
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near이번에는 문법 오류가 났다고 함... 보니까 Mapper sql문에 } 하나 더 들어감 -> }삭제<br>
Column 'PLACE_NAME' cannot be null -> 아마 api데이터 중에서 해당 컬럼에 대해 null값이 있나보다 -> notnull해제
