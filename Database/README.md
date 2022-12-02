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