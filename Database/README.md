# Database

# Statement와 PreparedStatement이란

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

## Statement
Statement는 문자열 기반 SQL 쿼리를 실행할 때 쓰인다.

### 1. 연쇄적인 문자열로 코드를 읽기 어렵다.
```
public void insert(PersonEntity personEntity) {
    String query = "INSERT INTO persons(id, name) VALUES(" + personEntity.getId() + ", '"
    + personEntity.getName() + "')";

    Statement statement = connection.createStatement();
    statement.executeUpdate(query);
}
```
### 2. SQL Injection에 취약하다. 
```
--는 SQL문에서 주석으로 해서되므로 update문이 무시된다.
dao.update(new PersonEntity(1, "hacker' --")); 

insert문은 실패된다.
dao.insert(new PersonEntity(1, "O'Brien"))
```

### 3. 캐시를 활용하지 않는다.
SQL 실행절차 1번 처리구간을 매 요청마다 수행한다. 

### 4. DDL 쿼리에 적합하다.
```
public void createTables() {
    String query = "create table if not exists PERSONS (ID INT, NAME VARCHAR(45))";
    connection.createStatement().executeUpdate(query);
}
```

### 5. 파일이나 배열을 저장/조회 할 수 없다.
 
<hr>

## PreparedStatement
PreparedStatement는 Statement를 상속받은 인터페이스다. 매개 변수화된 SQL 쿼리를 실행할 때 쓰인다. 

### 1. 바인딩 메서드 (setXxx)
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

### 2. SQL Injection을 예방한다. 
Prepared Statement에서 바인딩 변수를 사용하였을 때 쿼리의 문법 처리과정이 미리 선 수행되기 때문에 
바인딩 데이터는 문법적인 의미를 가질 수 없다. 

### 3. 미리 컴파일
Soft Parsing 사용 

### 4. batch 실행을 제공한다.
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
### 5. 파일, 배열 저장/조회

파일: BLOB과 CLOB 데이터 타입 사용

배열: java.sql.Array <-> SQL Array

### 6. getMetadata() 메서드
결과값에 대한 정보를 포함하고 있는 메서드

## 결론
Statement는 DDL(CREATE, ALTER, DROP) 구문을 처리할 때 적합하다.
매 실행시 Query를 다시 파싱하기 때문에 속도가 느리며, SQL Injection공격에 취약하다.
-> 필터나 방어 로직을 추가하는 것이 좋다.

Prepared Statement는 DML(SELECT, INSERT, UPDATE, DELETE)구문 처리에 적합하다.
그리고 캐시에 저장된 Query를 활용하기 때문에 실행이 빠르며 SQL Injection을 막기 위한 방법으로 활용된다.

<hr>

출처
- [Difference Between Statement and PreparedStatement](https://www.baeldung.com/java-statement-preparedstatement)
- [SQL injection 대응 : Prepared Statement](https://velog.io/@seaworld0125/SQL-injection-%EB%8C%80%EC%9D%91%EB%B0%A9%EB%B2%95-Prepared-Statement)
  
<hr>

# Database에서 slow query가 발생했을 경우 어떻게 대처하는지?
## slow query란
DBMS에서 Client로 요청받은 query를 실행할 때 일정 시간 이상 수행되지 못한 query

### slow query를 잡아내는 3가지 방법
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

<hr>

출처
- [PostgreSQL 슬로우쿼리를 잡아내는 3가지 방법](https://americanopeople.tistory.com/288)
- [MySQL 성능 개선을 위한 프로파일링 1편: 슬로우 쿼리 로그](https://velogio@breadkingdomMySQL-%EC%84%B1%EB%8A%A5-%EA%B0%9C%EC%84%A0%EC%9D%84-%EC%9C%84%ED%95%9C-%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81-1)
- [PostgreSQL Slow Query 검출](https://brufen97.tistory.com/5)

<hr>

# Redo와 Undo
트랜잭션이 수행되는 동안 시스템에 알 수 없는 오류 또는 물리적으로 문제가 발생했을 때, 트랜잭션의 수행을 되돌려야 합니다. <strong>rollback</strong> 이란 트랜잭션 내의 질의를 수행하면서 문제가 발생했을 경우 수행되는 것이지만, 시스템의 오류 또는 물리적인 문제의 경우 <strong>시스템 상의 문제이므로 트랜잭션이 다시 수행되어야 합니다. </strong>

이를 <strong>시스템 회복(recovery)</strong>이라 합니다.
회복은 데이터의 신뢰성을 보장하며, 트랜잭션의 영속성과 원자성을 보장합니다. 

Checkpoint 이후 Log 기록을 보면서 완료되지 않은 트랜잭션에 대해 수행
- Redo: 이전 상태로 되돌아간 후, 실패가 발생하기 전까지의 과정을 그대로 따라가는 것을 의미합니다.
- Undo: 트래잭션을 이전 상태로 되돌리는 것을 의미

## Redo
DB 장애 시 Buffer Pool에 저장되어 있던 데이터의 유실을 방지(데이터 복구)하기 위해 사용된다.
   
### InnoDB Buffer Pool?
Buffer Pool은 InnoDB 엔진이 Table Caching 및 Index Data Caching을 위해 이용하는 메모리 공간이다. 
다시 말해, Buffer Pool의 크기가 클수록 상대적으로 캐싱되는 데이터의 양이 늘어나기 때문에 Disk에 접근하는 횟수가 줄어들고, 이것은 DB의 성능 향상으로 이어진다.
하지만, Buffer Pool은 메모리 공간이기 때문에 <strong>데이터베이스 장애 시 Buffer Pool에 있는 내용은 휘발된다. </strong> 이것은 ACID를 보장할수 없게 되고, 다시 해석하면 장애를 복구하더라도 데이터는 복구될 수 없다는 것을 의미한다. 

실제 DB에서 Commit이 발생하면 바로 디스크 영역(Table Space)으로 들어가는 것이 아닌 메모리 영역(Buffer Poll & Log Buffer)에 들어가는 것을 확인할 수 있다. (DISK I/O 절약)

### Redo Log
Redo를 하기 위해서는 정상적으로 실행 되기까지의 과정을 기록해야 하는데, 이를 <strong>Redo Log</strong> 라고 한다. DML(SELECT 제외), DDL, TCL 등 데이터 변경이 일어나는 모든 것을 Redo Log 에 기록한다.

- DML (Data Manipulation Language)
  - SELECT, INSERT, UPDATE, DELETE

- DDL (Data Definition Language)
  - CREATE, ALTER, DROP, TRUNCATE

- TCL (Transactioin Control Language)
  - COMMIT, ROLLBACK

### Redo Log File
Log Buffer는 메모리 영역이기에 용량이 제한적이다. 용량이 제한적이기 때문에 Checkpoint 이벤트 발생시점에 Redo Log Buffer에 있던 데이터들을 Disk에 File로 저장하게 된다. 이 파일을 <strong>Redo Log File</strong> 라고 한다.
```
Checkpoint란 메모리상의 수정된 데이터 블럭과 디스크의 데이터 파일을 동기화시키는 데이터베이스 이벤트

다음을 목적으로 한다.
1. 데이터의 일관성 보장
2. 데이터베이스의 빠른 복구
```
Redo Log File은 두 개의 파일로 구성된다. 하나의 파일이 가득차면 Log switch가 발생하며 다른 파일에 쓰게 된다. Log switch가 발생할 때 마다 Checkpoint 이벤트도 발생되는데, 이때 InnoDB Buffer Pool Cache에 있던 데이터들이 백그라운드 스레드에 의해 Disk에 기록된다.

```
Checkpoint 이벤트가 발생하기 전에 장애가 발생한다면 Buffer Pool에 있던 데이터들은 휘발되지만, 
마지막 Checkpoint가 수행된 시점까지의 데이터가 Redo Log File로 남아있기 때문에 이 파일을 사용하여 데이터를 복구할 수 있다. 
```

### 과정
1. 실제 데이터 변경 전에 Redo Log Buffer에 데이터 변경에 대한 내용을 먼저 저장한다. (선 로그 기법: Log Ahead)
2. DB Cache에도 데이터 변경에 대한 내용을 기록한다.
3. LGWR 백그라운드 프로세스에 의해 Redo Log Buffer에 있는 내용을 Redo Log File에 저장한다.
4. Commit이 발생하게 되면 Control File에 트랜잭션의 고유한 번호(SCN)를 기록한다.
5. Log Switch가 발생하게 되면 Checkpoint 발생. 이 때 Checkpoint 프로세스가 DBWR 프로세스에게 Checkpoint 신호를 전달하면서 DBWR은 DB Cache 블록을 Data File로 저장한다.
6. Checkpoint가 DBWR에게 Checkpoint 신호를 전달하면서 Checkpoint SCN, 꽉 찬 로그파일 안의 제일 마지막 번호를 알려준다. 그 번호는 Data File과 Control File에 저장된다.

LGWR 자세히 
: [4장. 로그 기록자 백그라운드 프로세스(Log Writer, LGWR)](https://1duffy.tistory.com/24)

<hr>

## Undo
실행 취소 로그 레코드의 집합으로 트랜잭션 실행 후 <strong>Rollback</strong> 시 <strong>Undo Log를 참조해 이전 데이터로 복구</strong>할 수 있도록 로깅 해놓은 영역이다.

### Undo Log가 필요한 이유 
작업 수행 중에 수정된 페이지들이 버퍼 관리자의 버퍼 교체 알고리즘에 따라서 디스크에 출력될 수 있다.

버퍼 교체는 전적으로 버퍼의 상태에 따라 결정되며, 일관성 관점에서 봤을 때는 임의의 방식으로 일어나게 된다. 즉 아직 완료되지 않은 트랜잭션이 수정한 페이지들도 디스크에 출력될 수 있으므로, 만약 해당 트랜잭션이 어떤 이유든 정상적으로 종료될 수 없으면 트랜잭션이 변경한 페이지들은 원상 복구되어야 한다. 이러한 복구를 UNDO라고 한다.

### 어떻게 파일로 저장되는가?
Undo Log도 Redo Log와 마찬가지로 Log Buffer에 기록된다. Undo Records영역에 기록되는 것이다.
저장되는 데이터는 PK 값과 변경되기 전의 데이터 값이다. 

Redo Log가 트랜잭션 Commit과 CheckPoint 시 디스크에 기록되지만, Undo Log는 CheckPoint 시 디스크에 기록된다.

<strong>데이터를 수정함과 동시에 Rollback을 대비하기 위해, 업데이트 전의 데이터를 Undo Records로 기록하는 것이다. </strong>

<hr>

출처
- [Mysql Redo / Undo Log](https://velog.io/@pk3669/Mysql-Redo-Undo-Log)
- [🙈[DB이론] 신뢰성과 회복(Recovery)🐵](https://victorydntmd.tistory.com/130)

<hr>

# DELETE, TRUNCATE, DROP 차이
## DELETE
```
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias]
    [PARTITION (partition_name [, partition_name] ...)]
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]
```
- WHERE 절을 사용하여 데이터를 하나하나 선택해서 제거하는 방식
  - `DELETE FROM dbtable WHERE {조건};`
- WHERE 절을 사용하지 않고 테이블의 모든 데이터를 삭제하더라도, 내부적으로는 한 줄씩 제거된다.
  - `DELETE FROM dbtable;` 
- 원하는 데이터만 골라서 삭제할 때는 DELETE 사용 / 전체 데이터를 삭제할 때는 TRUNCATE 사용
- 데이터를 삭제하더라도 데이터가 담겨있던 Storage는 Release 되지 않는다.
- DELETE된 데이터는 COMMIT 명령어를 사용하기 전이라면 ROLLBACK 명령어를 통해 되돌릴 수 있다.
- 모든 작업을 로그에 남긴다.

## TRUNCATE
```
TRUNCATE [TABLE] tbl_name
```
- 테이블을 완전히 비운다. `DROP` 권한이 필요하다.
- 논리적으로 모든 행을 삭제하는 `DELETE` 문 또는 연속의 `DROP TABLE` 과 `CRATE TABLE` 문과 유사하다.
- 고성능을 위해 데이터 삭제의 DML을 bypass(우회)한다. 따라서 `ON DELETE` 트리거가 발생하지 않으며 부모-자녀 외부 키 관겨가 있는 InnoDB 테이블에 대해 수행할 수 없으며 DML 처럼 롤백할 수 없다. 그러나 automic DDL을 지원하는 스토리지 엔진을 사용하는 테이블의 `TRUNCATE TABLE` 작업은 서버가 작업 중에 중지되면 완전히 커밋되거나 롤백된다.
- `DELETE` 문과 유사하지만 DML문이 아닌 DDL문으로 분류된다. 다음과 같은 점에서 DELETE와 다르다.

### TRUNCATE가 DELETE와 다른 점 
- 행을 하나씩 삭제하는 `DELETE`보다 테이블을 drop 후 re-create하는 `TRUNCATE`가 속도가 더 빠르다.
- `TRUNCATE`는 암시적 커밋을 실행하므로 롤백할 수 없다.
- 이진 로깅과 replication(복제)를 위해 DDL로 처리되며 항상 로깅된다.


## DROP
```
DROP [TEMPORARY] TABLE [IF EXISTS]
    tbl_name [, tbl_name] ...
    [RESTRICT | CASCADE]
```
- 하나 이상의 테이블을 제거한다.
- 테이블이 파티셔닝 된 경우, 테이블의 정의, 테이블의 모든 파티션, 해당 파티션에 저장된 모든 데이터 및 삭제된 테이블과 관련된 모든 파티션 정의가 제거된다.
- 테이블을 삭제하면 테이블에 대한 트리거도 삭제된다.
- `TEMPORY` 키워드와 함께 사용되는 경우를 제외하고 암시적 커밋을 실행한다.
- 테이블이 삭제되어도 테이블에 대해 특별히 부여된 권한은 자동으로 삭제되지 않는다. 수동으로 삭제해야 한다. 

<hr>

출처
- [Delete vs truncate vs drop 차이점](https://itblacksmith.tistory.com/72)
- [13.1.37 TRUNCATE TABLE Statement](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)
- [13.1.32 DROP TABLE Statement](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html)

<hr>

# ORM 
## Object-Relational-Mapping
- 객체-관계 매핑
- 프로그래밍 언어의 객체 지향 패러다임을 사용하여 SQL 쿼리를 작성할 수 있는 아이디어
- 즉, SQL 대신 우리가 선택한 언어를 사용하여 데이터베이스와 상호작용 하려고 한다.
- 여기서 Object-relational-mapper가 등장한다. 대부분의 사람들이 "ORM"이라고 말하면 이 기술을 구현하는 라이브러리를 나타낸다.
- 대부분의 경우 객체 지향 프로그램과 관계형 데이터베이스 사이의 다리를 만드는 데 사용되는 기술이다. 즉, ORM은 객체 지향 프로그래밍(OOP)을 관계형 데이터베이스에 연결하는 계층으로 볼 수 있다. 
- 웹 애플리케이션과 데이터베이스 사이에 위치하는  미들웨어 어플리케이션 또는 도구이다.
- Model 클래스가 데이터베이스의 테이블이 되고 각 인스턴스가 테이블의 행이 되는 방식으로 매핑을 수행한다.

## 장점과 단점
장점
- 객체 지향적인 코드로 인해 더 직관적이다. 비즈니스 로직에 집중할 수 있게 도와준다.
  - SQL의 절차적이고 순차적인 접근이 아닌 객체 지향적인 접근으로 인해 생산성이 증가한다.
- 복잡한 과정을 숨김으로써 고수준의 구현만 신경쓰면 된다. 
- 데이터베이스 시스템을 추상화하여 데이터베이스를 변경해도 코드를 변경할 필요가 없다.

단점
- ORM이 복잡한 세부 정보를 추상화하는 데 도움이 되더라도 각 명령의 결과를 알고 있어야 한다.
- ORM을 사용하여 복잡한 쿼리를 코딩하는 데 성능 문제가 발생할 수 있다.

<hr>

출처
- [What is an ORM – The Meaning of Object Relational Mapping Database Tools](https://www.freecodecamp.org/news/what-is-an-orm-the-meaning-of-object-relational-mapping-database-tools/)
- [ORM Tools in Java](https://www.javatpoint.com/orm-tools-in-java)
- [ORM이란](https://gmlwjd9405.github.io/2019/02/01/orm.html)
- [What is an ORM and Why You Should Use it](https://blog.bitsrc.io/what-is-an-orm-and-why-you-should-use-it-b2b6f75f5e2a)

<hr>

# 데이터베이스 튜닝 (Database tuning)
## 개념
- 데이터베이스 어플리케이션, 데이터베이스 시스템, 운영체제의 이해를 바탕으로, 불합리한 요소를 찾아 제거/수정하여 성능을 개선하기 위한 일련의 작업
- 특히 데이터베이스 어플리케이션이 높은 작업 처리량과 짧은 응답시간을 갖도록 하는 것이 중요하다.
## 목적 
- 데이터베이스 설계 및 활용에 존재하는 문제점을 파악하여 분석하고, 이렇게 분석된 문제점들을 해결함으로써 데이터베이스의 활용 성능을 최적화시킨다.
- 데이터베이스 튜닝을 수행함으로써 데이터베이스를 활용하는 시스템을 안정시키고, 또한 사용자의 만족과 관리자의 관리 능력을 향상시킬 수 있다. 

## 튜닝 3단계 
### 1. DB 설계 튜닝 (모델링 관점)
- <strong>데이터베이스 설계 단계</strong>에서 성능을 고려하여 설계
- 데이터 모델링, 인덱스 설계
- 데이터파일, 테이블 스페이스 설계
- 데이터베이스 용량 산정
- Ex. 반정규화, 분산파일, 배치

### 2. DBMS 튜닝 (환경 관점)
- 성능을 고려하여 메모리나 블록 크기 지정
- CPU, 메모리 I/O에 대한 관점
- Ex. Buffer 크기, Cache 크기

### 3. SQL 튜닝 (APP 관점)
- SQL 작성 시 성능 고려
- Join, Indexing, SQL Execution Plan
- Ex. Hash, Join


<hr>

출처
- [데이터베이스 튜닝 (DB Tuning)](http://blog.skby.net/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%ED%8A%9C%EB%8B%9D-db-tuning/)
- [데이터베이스 튜닝](https://dataonair.or.kr/db-tech-reference/d-lounge/report/?mod=document&uid=239679)
- [데이터베이스 튜닝이란?](http://wiki.gurubee.net/pages/viewpage.action?pageId=14024765)

<hr>