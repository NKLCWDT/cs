## 시퀀스의 개념

데이터의 '순서'를 뜻하며, 시퀀스를 객체로 사용하여 자동으로 증가하는 숫자로 사용한다.

#### 시퀀스 생성 예시

```
CREATE SEQUENCE [이름]
INCREMENT BY [증감숫자]
START WITH [시작숫자] 
MINVALUE [최솟값] 
MAXVALUE [최대값]
CYCLE OR NOCYCLE 
CACHE [숫자, 생략가능] OR NOCACHE 
ORDER OR NOORDER
```

위 옵션들을 하나씩 알아보자.

> 1) INCREMENT BY \[증감숫자\]

시퀀스에서 생성할 번호의 증가 값이다. 설정하지 않으면 default 1을 가진다.

> 2) START WITH \[시작 숫자\]

시퀀스의 시작값을 지정한다.

> 3) MINVALUE \[최솟값\]

시퀀스에서 생성할 번호의 최솟값을 지정한다.

> 4) MAXVALUE \[최댓값\]

시퀀스에서 생성할 번호의 최댓값을 지정한다.

> 5) CYCLE OR NOCYCLE 

시퀀스에서 생성한 번호가 최댓값에 도달했을 경우 설정에 따라 수행된다.

- CYCLE : 시작값으로 돌아가서 처음부터 다시 시작한다.

- NOCYCLE : 최댓값에 도달하여, 중지된다.

> 6) CACHE \[숫자, 생략가능\] OR NOCACHE

- CACHE : 메모리에 시퀀스 값을 미리 할당한다.

> CACHE 10이 설정되어있을 경우, sequence 번호를 한번에 10개씩 메모리에 올려놓고 작업을 한다. 만일 메모리에 21~30번까지 시퀀스 번호를 올려놓았다고 가정할때, DB를 재시작하게 되면 메모리에 있던 21~30번은 삭제되고 31~40번까지의 시퀀스 번호가 새로 올라가기 때문에 21번~30번의 시퀀스 번호가 존재하지않을 수 있다.

- NOCACHE : 메모리에 시퀀스 값을 미리 할당하지 않는다.

> 7) ORDER OR NOORDER

거의 사용할 일이 없다. 위 CACHE, NOCACHE 옵션으로 대체할 수 있다.

- ORDER : 시퀀스를 순차적으로 모두 채운다.

- NOORDER : 시퀀스의 값을 건너뛸 수 있다.

## 시퀀스 사용
#### CURRVAL

```
-- 마지막에 생성된 시퀀스를 가져온다.
SELECT sequence명.CURRVAL FROM DUAL;
```

#### NEXTVAL

```
-- 다음으로 생성될 시퀀스를 가져온다.
SELECT sequence명.NEXTVAL FROM DUAL;
```

## 시퀀스 사용의 장단점

#### 장점

1) unique한 값을 자동 생성할 수 있다.

2) 쿼리에 sequence명.nextval 을 사용하여 쉽게 사용할 수 있다.

3) 테이블과 독립적으로 저장되고 생성된다.

4) 메모리에 Cache되었을 때 시퀀스값의 액세스 효율이 증가 한다.

5) PK 관리가 수월해진다.

#### 단점

1) 옵션값에 따라 또는 DB의 비정상적인 종료로 인해 순차적으로 번호 관리가 안될 수 있다.

2) 테이블과 같이 시퀀스도 Object다. 생성 및 사용에 번거로움이 있다.


## Q&A
### Mysql 의 시퀀스
Oracle 의 시퀀스라는 기능은 없다. 대신, 시퀀스와 동일한 기능을 하는 별도의 테이블 생성은 할 수 있다.
> 예시 참고 : https://proudin.tistory.com/28

Mysql InnoDB auto-increment 관련 포스팅
> https://gywn.net/2013/02/mysql-innodb-auto-increment/ 

### Oracle 12C 버전부터 IDENTITY 사용 가능
Mysql 의 auto_increment 와 비슷한 기능을 하는 IDENTITY 도입
```oracle
create table test(
id int generated always as IDENTITY ,
name varchar(50)
);
```

#### Reference.

[https://subbak2.tistory.com/16](https://subbak2.tistory.com/16)

[https://hong-study.tistory.com/99#recentEntries](https://hong-study.tistory.com/99#recentEntries)

[https://denodo1.tistory.com/273](https://denodo1.tistory.com/273)