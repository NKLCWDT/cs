# JDBC(Java Database Connectivity)
Java프로그램에서 데이터베이스에 접근할 수 있도록 제공하는 API이다. 
- JDBC API는 JDBC Driver을 이용해서 데이터베이스에 연결한다. 
- JDBC는 데이터베이스에서 쿼리를 실행하거나 업데이트 하게 해준다. 

- 모든 Persistence Framework는 내부적으로 JDBC API를 이용한다. 
<img src="/images/jpa.png" width="550" height="350"/><br> 

>Persistence Framework는 JDBC프로그래밍의 복잡함이나 번거로움 없이 간단한 작업만으로 DB와 연동되는 시스템을 삐르게 개발할 수 있다. ex) JPA, Mybatis

> JDBC 이전에는 ODBC API가 데이터베이스에 연결 혹은 쿼리를 실행하기 위해 사용되었다. 하지만, ODBC API는 ODBC Driver를 사용하는데 해당 Driver는 C언어로 작성되었다. 그래서 자바가 JDBC Driver를 사용하는 자바만의 API를 만들었다.

<img src="/images/jdbc.png" width="350" height="350"/><br> 
이렇게 JDBC는 JDBC API, JDBC Driver Manager, 그리고 JDBC Driver로 이루어져있다.
<br>

### JDBC API
자바에서 데이터베이스를 연결하고 데이터베이스를 제어 할 수 있도록 데이터 베이스 연결 및 제어를 위한 인터페이스와 클래스를 제공한다. 

### JDBC Driver
Java 어플리케이션이 데이터베이스와 연결할 수 있게 해준다. 총 4가지 종류의 JDBC driver가 있다. 

### JDBC-ODBC bridge driver
<img src="/images/jdbcodbcDriver.png" width="550" height="350"/><br> 

- 이 드라이버는 ODBC드라이버를 사용해서 데이터베이스에 연결한다. 
- JDBC 메서드들을 ODBC의 함수 들로 바뀌는 과정을 거쳐야 하기 때문에 성능이 저하된다.
- 자바8에서는 없어졌다.

#### 단점
- Client측에서 ODBC드라이버를 다운받아야 한다.
<br>

### Native API driver
<img src="/images/nativeDriver.png" width="550" height="350"/><br> 
- JDBC-ODBC bridge driver에서 ODBC 부분이 Native API로 대처되었다. 
- Native API는 특정 데이터베이스를 대상으로 하며 데이터베이스의 client측 라이브러리를 사용한다. 
- JDBC에 들어온 호출 내용은 oracle과 같이 각 DBMS시스템에 맞춰 변환해서 전달한다. 
- DBMS와 연결되는 부분은 C/C++같은 native언어로 작성되어 있고 이것을 Java로 wrapping해서 구현한다. 

#### 장점
- JDBC- ODBC bridge driver보다는 성능이 향상되었다. 

#### 단점
- Client측에서 각 db에 맞는 library를 다운로드 받아야한다.
- Client측에서 native driver를 다운로드 받아야한다.  
<br>

### Network Protocol Driver
<img src="/images/networkProtocolDriver.png" width="550" height="350"/><br> 

- JDBC driver에 들어온 호출 내용을 middleware(중간 서버)에 보낸다. 
- 여기서 middle ware(중간 서버)가 각자 맞는 DB에 명령어를 변환해 전송한다. 
- 100% 자바로 작성되었다.

> 미들웨어는 양 쪽을 연결하여 데이터를 주고받을 수 있도록 중간에서 매개 역할을 하는 소프트웨어


#### 장점
- client측에서 library를 다운로드 받지 않아도 된다.
#### 단점
- middle ware 서버를 구현 해야한다.
<br>

### Thin Driver
<img src="/images/thinDriver.png" width="550" height="350"/><br> 
- 100% 자바로 작성되었다.
- JDBC driver에 들어온 호출 내용을 바로 각 데이터베이스로 전송하는 방식이다.
- 요즘 가장 많이 사용되는 방식

#### 장점
- 가장 효율적이고 빠르다.
#### 단점
- 각 데이터베이스마다 다른 JDBC driver를 사용해야한다.
<br>

### JDBC 데이터베이스 연결 방법
<img src="/images/jdbcConnectivity.png" width="350" height="350"/><br> 

1. Driver class를 등록해야한다. 
```java
Class.forName("oracle.jdbc.driver.OracleDriver");  
```

> DriverManager 클래스는 JDBC Driver들의 관리를 한다. 위에 위의 과정으로 사용할 driver을 drivermanager에 등록한다. 이 과정이 항상 connection을 맺기 전에 필요하다. 

2. 연결을 맺는다.  
connection 객체 생성됨
```java
Connection con=DriverManager.getConnection(  
"jdbc:oracle:thin:@localhost:1521:xe","system","password");  
``` 

3. statement 객체 생성  
query를 실행시킬 수 있게 statement 객체를 생성한다.

```java
Statement stmt=con.createStatement();  
```

4. query 실행
```java
ResultSet rs=stmt.executeQuery("select * from emp");  
```
>result set은 쿼리 실행후의 반환되는 record set을 저장한다. record set은 가상의 데이터베이스 형태

5. 연결 끊기

```java
con.close();  
```
<br>

### Statement 
- 쿼리를 실행하기 위한 interface이다.
- 파라미터값을 받지 못한다.
- 정적 SQL문을 사용할때 유리하다.
- 매번 컴파일한다.

```java
//Statement 객체 생성
Statement GFG = con.createStatement();
  
//Statement 실행
GFG.executeUpdate("CREATE TABLE STUDENT(ID NUMBER NOT NULL, NAME VARCHAR)");

```
<br>

### PreparedStatement
- 캐시를 사용해 SQL문을 여러번 사용하고 싶을때 유리하다.
- 한번만 컴파일 하고 이후에는 ?부분만 변화를 주기 때문에 statement에 비해 빠르다.


```java
//PreparedStatement 객체 생성
PreparedStatement GFG = con.prepareStatement("update STUDENT set NAME = ? where ID = ?");
  
//첫번째 ?에 대한 값을 넣는다.
GFG.setString(1, "RAM");   
          
//두번째 ?에 대한 값을 넣는다.
GFG.setInt(2, 512);     
 
//PreparedStatement를 실행한다.
GFG.executeUpdate(); 
```
<br>

#### Spring JDBC
기존에 제공하던 JDBC API에서는 error handling, database연결 및 연결 닫기 등을 직접해줘야 하는 번거러움이 있었다. 

Spring JDBC 프레임워크는 이런 요소들을 신경 쓸 필요 없게 해준다.<br>
<img src="/images/springJdbc2.png" width="550" height="350"/><br> 


__Spring JDBC가 하는일__  
- Connection 열기와 닫기
- Statement 준비와 닫기
- Statement실행
- ResultSet loop 처리
- Exception 처리 반환
- Transaction 처리

__그렇다면, 개발자는 어떤일을 해야할까?__  
<img src="/images/springJdbc.png" width="550" height="350"/><br> 

1. datasource설정  
데이터베이스와 연결하기 위해 DB server에 관한 property를 설정해준다. 
2. SQL문을 작성한다. 
3. 결과를 처리한다.
<br>

Spring Framework는 여러가지 Spring JDBC접근 방법을 제공한다.   
1. JDBC Template
2. NamedParameterJdbcTemplate
3. SimpleJdbcTemplate
4. SimpleJdbcInsert 및 SimpleJdbcCall

이 중에서 JDBC Template class가 전형적인 Spring JDBC 접근 방법이며 가장 인기가 좋다. 


<br><br><br>
> 참고자료
- https://m.blog.daum.net/bigdown/112
- https://ko.myservername.com/java-substring-method-tutorial-with-examples
- https://www.javatpoint.com/steps-to-connect-to-the-database-in-java
- https://codedragon.tistory.com/5975
- https://gmlwjd9405.github.io/2018/12/25/difference-jdbc-jpa-mybatis.html
