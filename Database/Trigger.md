## 트리거 (Trigger)

특정 테이블에서 INSERT, DELETE, UPDATE 같은 DML 문이 수행되었을때 데이터베이스에서 자동으로 동작하도록 작성된 프로그램이다. 트리거는 사용자가 직접 호출하는 것이 아닌 데이터베이스에서 자동적으로 호출한다. DM 문의 수행시, 묵시적으로 함께 수행되는 프로시저(PROCEDURE) 이다.

<br/>

> 특징

1) 자동으로 실행되며 수동으로 실행할 수 없다.

2) 어떤 동작에서 실행되는지 지정해야한다.

3) 트리거를 실행할 때 트리거 내의 SQL 명령문은 잠정적으로 다른 트리거를 실행할 수 있다.

4) 트리거의 동작 영역은 PL/SQL로 작성한다.
- 참고 : https://kslee7746.tistory.com/entry/PLSQL-%EA%B8%B0%EC%B4%88%EA%B5%AC%EC%A1%B0-%EB%B3%80%EC%88%98-%EB%AC%B8%EB%B2%95

5) 하나의 트리거가 종료되지 않은 상태에서 다른 트리거 호출 불가하다. 
   
<br/>   
<br/> 

## 트리거 종류

#### 행 트리거

- 조건을 만족하는 여러개의 행에 대해 트리거를 반복적으로 여러번 수행하는 방법

- 트리거 생성시, \[FOR EACH ROW WHEN 조건\] 을 명시

- 변경 후의 행은 OLD, NEW 를 통해 가져올 수 있다. (키워드는 바로 아래 참고)

<br/>

```
DELIMITER $$
	CREATE TRIGGER test
	BEFORE  ... 
    ON test
	FOR EACH ROW 
	BEGIN
		...
		SET idTemp = OLD.id;
		SET answerTemp = OLD.answer;
		...
	END $$
DELIMITER ;
```

여기서 OLD, NEW 키워드를 알아보자.

<br/>

> 1) OLD

예전의 데이터를 말한다.

DELETE로 삭제된 데이터 혹은 UPDATE로 바뀌기전 데이터

<br/>

> 2) NEW

새로운 데이터를 말한다.

INSERT로 삽입된 데이터 혹은 UPDATE로 바뀐 후 데이터


<br/>
<br/>

#### 문장 트리거

- DML 문에 대해서 한번만 실행된다.

- 트리거가 생성된 테이블에 트리거 이벤트가 발생하면 많은 행에 대해 변경 작업이 발생하더라도, 오직 한번만 트리거를 발생시키는 방법이다.

- 컬럼값이 변화가 생길때마다 스스로 알아서 실행된다.

<br/>

```
UPDATE test SET grade = 50
-- 여러 row를 변경될 경우 트리거는 한번만 실행
```

<br/>

- 트리거 생성시, \[FOR EACH ROW WHEN 조건\] 을 명시하지 않음

<br/>
<br/>

## 트리거 장점/단점

> 장점

1) 데이터 참조 무결성의 강화

- 참조 무결성이란? 참고 관계에 있는 두 테이블의 데이터가 항상 일관된 값을 갖도록 유지되는것
- 트리거로 인해 데이터 변경이 발생했을 경우, 참조 무결성에 위반되면 에러가 발생하면서 롤백된다. 

<br/>

2) 업무 처리 자동화 가능

- 사용자가 직접 개입하지 않고 구현된 규칙대로 알아서 실행된다.

<br/>

3) 여러 작업을 하나의 트랜잭션으로 처리할 수 있다.


<br/>

> 단점

1) 트리거를 과도하게 사용하면 복잡한 상호의존성 야기
2) 트리거의 연쇄 수행 가능성 존재

> ** 트리거 연쇄  
> 하나의 트리거가 활성화되어 이 트리거 내의 SQL문이 수행되고, 그 결과로 인해 또 다른 트리거가 활성화되는 경우


<br/>
<br/>

## 트리거 문법

작동 시점

1) After : 이벤트 발생 이후 트리거 실행

2) Before : 이벤트 발생 이전 트리거 실행

<br/>

```
CREATE [ OR REPLACE ] TRIGGER trigger_name
[AFTER/BEFORE]  [이벤트1] [OR 이벤트2] ……
   ON table_name
   [ FOR EACH ROW ]  -- 행트리거 여부를 결정, DML의 영향을 받는 레코드마다 트리거동작
   [WHEN 조건]
DECLARE
  -- 선언부
BEGIN
  -- 트리거 실행 코드
EXCEPTION
END;
```

<br/>

#### 생성 예시

```
--주말 오전10시에서 오후 6시 사이 EMP 테이블에 DML 사용못하게 하는 트리거
--아래는 명령문 트리거, 여러건이 DML의 영향을 받더라도 한번만 실행
--[FOR EACH ROW] 없으므로 문장 트리거라고 볼 수 있다.

SQL> create or replace trigger chk_emp_dml
       before insert or update or delete on emp
       begin
        if (
            (to_char(sysdate,'DY') in ('토','일')) AND
            (to_number(to_char(sysdate,'HH24'))) >= 10 AND
            (to_number(to_char(sysdate,'HH24'))) <= 18) THEN
             raise_application_error(-20001,'주말변경불가합니다.');
        end if;
       end;
```

<br/>
<br/>

## 트리거 트랜잭션

트리거는 데이터베이스에 의해 자동 호출되지만 결국 INSERT, UPDATE, DELETE 구문과 하나의 트랜잭션 안에서 일어나는 일련의 작업들이다. 여러가지 작업을 하나의 트랜잭션으로 묶어서 실행해야할때 트리거를 많이 사용한다.

트리거 수행중 에러 발생시, 수행된 이전의 DML문까지 모두 롤백된다.

<br/>
<br/>


## 트리거 수행 예제

> 테이블 생성

```
CREATE TABLE chat(
	id VARCHAR(32),
	answer VARCHAR(32) NOT NULL
);

CREATE TABLE chatBackup(
	idBackup VARCHAR(32),
	answerBackup VARCHAR(32) NOT NULL
);


INSERT INTO chat VALUE ('pigg', '안녕');
INSERT INTO chat VALUE ('lemon', '반가워');
INSERT INTO chat VALUE ('pigg', '오늘 날씨 어때?');
INSERT INTO chat VALUE ('lemon', '날씨 좋아');

-- [출처] [MySQL] 트리거(Trigger)의 활용|작성자 피끄
```

<br/>

> 위 데이터 결과

- chat 테이블

| id | answer |
| --- | --- |
| pigg | 안녕 |
| lemon | 반가워 |
| pigg | 오늘 날씨 어때? |
| lemon | 날시 좋아- |

<br/>

> 트리거 생성

- 트리거 발동 조건 : chat 테이블 데이터 DELETE (BEFORE DELETE ON chat)


```
DELIMITER $$
	CREATE TRIGGER autoBackup
	BEFORE  DELETE ON chat
	FOR EACH ROW 
	BEGIN
		DECLARE idTemp VARCHAR(32);
		DECLARE answerTemp VARCHAR(32);
		
		SET idTemp = OLD.id;
		SET answerTemp = OLD.answer;
		
		INSERT INTO chatBackup VALUE (idTemp, answerTemp);
		
	END $$
DELIMITER ;

-- [출처] [MySQL] 트리거(Trigger)의 활용|작성자 피끄
```

<br/>

> 트리거 생성 확인

```
SHOW TRIGGERS;
```

<br/>

> 트리거를 발동시켜보자.

```
DELETE FROM chat WHERE id='pigg';
```

<br/>

> 트리거가 수행된 이후 결과를 조회해보자.

```
SELECT * FROM chatbackup;
```

<br/>

> 결과

- chatbackup 테이블

| idBackup | answerBackup |
| --- | --- |
| pigg | 안녕 |
| pigg | 오늘 날시 어때? |


<br/>
<br/>


## 정리
트리거의 특징, 장점/단점 위주로 준비하면 될 것 같다. 트리거의 사용 문법 등 보다는 트리거의 장점/단점/특징을 파악하여 어떤 상황에서 트리거를 사용해야할지를 준비하는걸 권장한다.



<br/>
<br/>

### Reference.

[https://blog.naver.com/PostView.nhn?blogId=alcmskfl17&logNo=221859839012](https://blog.naver.com/PostView.nhn?blogId=alcmskfl17&logNo=221859839012)

[https://limkydev.tistory.com/154](https://limkydev.tistory.com/154)

[https://m.blog.naver.com/leejongcheol2018/222035608132](https://m.blog.naver.com/leejongcheol2018/222035608132)

[https://runcoding.tistory.com/32](https://runcoding.tistory.com/32)