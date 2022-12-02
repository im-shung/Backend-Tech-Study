# Database

## Statement와 PreparedStatement이란

### JDBC API Interface
Statement와 PreparedStatement 둘다 SQL 쿼리를 실행하는데 쓰이는 인터페이스다. 

```
SQL 문 실행 절차
1. 구문 오류 체크 (parse)
2. 공유 영역에서 해당 구문 검색 (parse)
3. 권한 체크 (parse)
4. 실행 계획 수립 (parse)
5. 실행 계획 공유 영역에 저장 (parse)
6. 쿼리 실행 (Excute)
7. 데이터 인출 (patch)
```
```
1~7 모두 실행 : Hard Parsing
```
```
1, 2, 6, 7 만 실행 : Soft Parsing
```
Statement를 사용하여 SELECT 쿼리를 입력했을 때에는 매번 parse부터 fetch까지 모든 과정을 수행한다. (HARD Parsing)

Prepared Statement를 사용하는 경우 parse과정을 최초 1번만 수행하고 이후는 생략할 수 있다. (Soft Parsing)
<hr>

### Statement
Statement는 문자열 기반 SQL 쿼리를 실행할 때 쓰인다.

1. 연쇄적인 문자열로 코드를 읽기 어렵다.
    ```
    public void insert(PersonEntity personEntity) {
        String query = "INSERT INTO persons(id, name) VALUES(" + personEntity.getId() + ", '"
        + personEntity.getName() + "')";

        Statement statement = connection.createStatement();
        statement.executeUpdate(query);
    }
    ```
2. SQL Injection에 취약하다. 
    ```
    --는 SQL문에서 주석으로 해서되므로 update문이 무시된다.
    dao.update(new PersonEntity(1, "hacker' --")); 

    insert문은 실패된다.
    dao.insert(new PersonEntity(1, "O'Brien"))
    ```

3. 캐시를 활용하지 않는다.
   
    SQL 실행절차 1번 처리구간을 매 요청마다 수행한다. 

4. DDL 쿼리에 적합하다.
    ```
    public void createTables() {
        String query = "create table if not exists PERSONS (ID INT, NAME VARCHAR(45))";
        connection.createStatement().executeUpdate(query);
    }
    ```

5. 파일이나 배열을 저장/조회 할 수 없다.
 
<hr>

### PreparedStatement
PreparedStatement는 Statement를 상속받은 인터페이스다. 매개 변수화된 SQL 쿼리를 실행할 때 쓰인다. 

1. 바인딩 메서드 (setXxx)

    다양한 객체 유형을 바인딩하는 메서드가 있습니다. (파일과 배열을 포함해서)
    ```
    public void insert(PersonEntity personEntity) {
        String query = "INSERT INTO persons(id, name) VALUES( ?, ?)";

        PreparedStatement preparedStatement = connection.prepareStatement(query);
        preparedStatement.setInt(1, personEntity.getId());
        preparedStatement.setString(2, personEntity.getName());
        preparedStatement.executeUpdate();
    }
    ```

2. SQL Injection을 예방한다. 

    Prepared Statement에서 바인딩 변수를 사용하였을 때 쿼리의 문법 처리과정이 미리 선 수행되기 때문에 
    바인딩 데이터는 문법적인 의미를 가질 수 없다. 

3. 미리 컴파일
    Soft Parsing 사용 

4. batch 실행을 제공한다.
    ```
    public void insert(List<PersonEntity> personEntities) throws SQLException {
        String query = "INSERT INTO persons(id, name) VALUES( ?, ?)";
        PreparedStatement preparedStatement = connection.prepareStatement(query);
        for (PersonEntity personEntity: personEntities) {
            preparedStatement.setInt(1, personEntity.getId());
            preparedStatement.setString(2, personEntity.getName());
            preparedStatement.addBatch();
        }
        preparedStatement.executeBatch();
    }
    ```

5. 파일, 배열 저장/조회

   파일: BLOB과 CLOB 데이터 타입 사용

   배열: java.sql.Array <-> SQL Array

6. getMetadata() 메서드
   결과값에 대한 정보를 포함하고 있는 메서드

### 결론
Statement는 DDL(CREATE, ALTER, DROP) 구문을 처리할 때 적합하다.
매 실행시 Query를 다시 파싱하기 때문에 속도가 느리며, SQL Injection공격에 취약하다.
-> 필터나 방어 로직을 추가하는 것이 좋다.

Prepared Statement는 DML(SELECT, INSERT, UPDATE, DELETE)구문 처리에 적합하다.
그리고 캐시에 저장된 Query를 활용하기 때문에 실행이 빠르며 SQL Injection을 막기 위한 방법으로 활용된다.

```
출처
https://www.baeldung.com/java-statement-preparedstatement
https://velog.io/@seaworld0125/SQL-injection-%EB%8C%80%EC%9D%91%EB%B0%A9%EB%B2%95-Prepared-Statement
```
<hr>

## Database에서 slow query가 발생했을 경우 어떻게 대처하는지?
### slow query란
DBMS에서 Client로 요청받은 query를 실행할 때 일정 시간 이상 수행되지 못한 query

slow query를 잡아내는 3가지 방법
- slow query 로그 남기기
- 쿼리 실행계획 로그에 남기기
- 쿼리 실행 통계 보기

PostgreSQL의 경우로 설명
### 1. slow query 로그 남기기
```
slow query 로그는 에러가 발생한 쿼리는 기록하지 않는다. 
general 로그에서는 에러가 발생한 쿼리를 포함해서 모든 쿼리를 기록하므로 두 개의 로그를 모두 활성화하면 프로파일링에 도움이 된다.
```
postgresql.conf에 `log_min_duration_statement = 1000` 를 설정한다. 
- 밀리세컨드 단위로 설정
- 설정값보다 오래 걸리는 쿼리를 로그 파일에 기록
- 0으로 설정하면 모든 로그 기록
- 수정 후 reload 하여 적용 (restart 필요 없음)

부하를 유발하는 단일 쿼리를 파악하기 쉽다. 그러나 처리 시간은 빠르지만, 여러번 호출되서 부하를 발생시키는 쿼리 실행을 인지하기는 어렵다는 단점이 있다. 


### 2. 쿼리 실행계획 로그에 남기기
PostgreSQL의 경우 postgresql.conf에 auto_explain 라이브러리를 추가한다. 
`session_preload_libraries = 'auto_explain';`

실행계획을 로그에 남기면, 당시의 쿼리 실행 계획을 볼 수 있단 장점이 있다.

롱쿼리가 발생한 이후 데이터가 더 쌓이거나 삭제되면, 문제가 된 순간의 실행계획을 알 수 없다. 때문에 이를 확인할 수 있단 장점이 있다. 그런데 EXPLAIN ANALYZE 명령문을 기반으로 로그를 남기기 때문에, 롱쿼리를 다시 실행시킨단 리스크가 있다. 

그리고 단일 롱쿼리만 파악할 수 있기 때문에, 짧지만 여러번 호출되서 문제를 일으키는 쿼리를 알 수 없는 단점이 있다.


### 3. 쿼리 실행 통계 보기
쿼리 실행 통계 라이브러리는 shared memory를 사용하기 때문에, 해당 모듈을 추가 / 삭제할 때는 항상 서버를 restart해줘야한다. `shared_preload_libraries = 'pg_stat_statements'`

이 방식을 사용하면, 빨리 실행되지만 부하를 일으키는 쿼리를 파악하기 좋단 장점이 있다. 


```
출처: https://americanopeople.tistory.com/288
https://velog.io/@breadkingdom/MySQL-%EC%84%B1%EB%8A%A5-%EA%B0%9C%EC%84%A0%EC%9D%84-%EC%9C%84%ED%95%9C-%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81-1
https://brufen97.tistory.com/5
```
