H2 database 연결하기

1. https://www.h2database.com/html/download.html 에서 OS에 맞게 H2 데이터베이스를 다운받는다.

2. 설치를 한 다음 H2 console을 찾아 실행시킨다.
   2-1. 실행시킨다음 정상적으로 설치가 잘됐으면 바로 콘솔 브라우저가 뜬다. 다만 'java'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는
배치 파일이 아닙니다. 메세지가 cmd에서 보이면 자바가 설치되어있지 않다는 말이기에 원하는 버전의 java를 설치한 뒤 다시 실행시키면 된다.

3. ![image](https://github.com/sytp93/sytp93.github.io/assets/94149889/7eddb768-58a6-4b9a-a879-ee2e770025bb) 정상적으로 실행된 화면이다.
  http://localhost:8082/ h2 console의 기본 접속 주소이다.

4. 이 상태에서 아무것도 설정하지않고 연결 시험을 눌렀을때
   Database "C:/Users/..." not found, either pre-create it or allow remote database creation (not recommended in secure environments) [90149-224]
   이런 메세지가 발생할수도 있다.
   이 메세지가 발생했을때 그냥 연결 시험이 아니라 연결을 누르면 DB가 생성된다.

이러면 H2 DataBase 생성이 완료됐다. 이 상태로 자신이 원하는 소스에 가져다 사용하면 된다.
