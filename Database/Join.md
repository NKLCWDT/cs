## 조인의 개념

- 두 개 이상의 테이블 들을 연결 또는 결합하여 데이터를 출력하는것

- JOIN은 관계형 데이터베이스의 가장 큰 장점이면서 대표적인 핵심 기능이라고 할 수 있다.

- 일반적인 경우 행들은 PRIMARY KEY(PK)나 FOREIGN KEY(FK) 값의 연관에 의해 JOIN이 성립된다.

- 하지만 어떤 경우에는 이러한 PK, FK의 관계가 없어도 논리적인 값들의 연관만으로 JOIN이 성립 가능하다.

## Inner Join (내부 조인)

#### 1) Equi Join (동등 조인)

- 두 개의 테이블 간에 컬럼 값들이 서로 정확하게 일치하는 경우에 사용된다.

- 위 세타조인 중 비교연산자 = 를 사용한 조인이다.

- 동등 조건에 해당하는 튜플을 반환한다.

> 예제 쿼리

```
SELECT 테이블1.칼럼명, 테이블2.칼럼명, ...
FROM 테이블1, 테이블2
WHERE 테이블1.칼럼명1 = 테이블2.칼럼명2;
-- WHERE 절에 JOIN 조건을 넣는다.
```

![IMAGES](../images/Join1.png)

#### 2) Non-equi Join

- 두 개의 테이블 간에 컬럼 값들이 서로 정확하게 일치하지 않는 경우에 사용된다.

- 위 동등조인의 '=' 연산자가 아닌 Between, >, >=, <, <= 등의 연산자들을 사용하여 JOIN을 수행한다.

> 예제 쿼리

```
SELECT 테이블1.칼럼명, 테이블2.칼럼명, ...
FROM 테이블1, 테이블2
WHERE 테이블1.칼럼명1 BETWEEN 테이블2.칼럼명1 AND 테이블2.칼럼명2;
```

![IMAGES](../images/Join2.png)

## Natural Join (자연조인)

- 위 동등 조인의 결과에서 중복되는 속성(=컬럼)을 제거한 결과를 반환한다.

- 두 테이블 간의 동일한 이름을 갖는 모든 컬럼들에 대해 Equi Join을 수행한다. 추가로 USING 조건절, ON 조건절, WHERE 조건절에서 JOIN 조건절을 정의할 수 없다.

```
SELECT DEPTNO, EMPNO, ENAME, DNAME
FROM   EMP NATURAL JOIN DEPT;
-- NATURAL JOIN은 식별자를 가질 수 없다. 또한 동일한 열에 대해서는 생략된다.

SELECT *
FROM   EMP NATURAL JOIN DEPT;
-- 두 테이블에 중복된 컬럼이 ENAME 일때, 결과에 ENAME 은 하나만 포함된다.
-- 이 말은 즉, JOIN에 이용된 속성의 값이 1개만 포함된다.
```

## Outer Join (외부 조인)

- 여러 테이블에서 한쪽에는 데이터가 존재하고, 한쪽에는 데이터가 없는 경우 외부 조인을 사용하여 원하는 데이터를 모두 출력한다.

- 조건에 맞지 않아도 외부 조인에 맞게 결과에 포함된다.

- 조인 조건은 WHERE절이 아닌 ON절에 작성한다.

#### 1) LEFT OUTER JOIN

```
SELECT *
FROM 테이블A LEFT OUTER JOIN 테이블B
ON 테이블A.컬럼명 = 테이블B.컬럼명;

-- 테이블 A의 결과를 모두 가져온후, 테이블 B의 데이터를 매칭한다.
-- 매칭되는 결과가 없을 경우 NULL로 표시된다.
-- 따라서 테이블 A는 모든 결과가 출력된다.
```

> 결과 예시

| 테이블A | 테이블B |
| --- | --- |
| 1 | null |
| 2 | null |
| 3 | 3 |

#### 2) RIGHT OUTER JOIN

```
SELECT *
FROM 테이블A RIGHT OUTER JOIN 테이블B
ON 테이블A.컬럼명 = 테이블B.컬럼명;

-- 테이블 B의 결과를 모두 가져온후, 테이블 A의 데이터를 매칭한다.
-- 매칭되는 결과가 없을 경우 NULL로 표시된다.
-- 따라서 테이블 B는 모든 결과가 출력된다.
```

> 결과 예시

| 테이블A | 테이블B |
| --- | --- |
| null | 1 |
| null | 2 |
| 3 | 3 |

#### 3) FULL OUTER JOIN

```
SELECT *
FROM 테이블A FULL OUTER JOIN 테이블B
ON 테이블A.컬럼명 = 테이블B.컬럼명;

-- 위 LEFT OUTER JOIN, RIGHT OUTER JOIN 의 결과를 모두 합친 결과와 동일하다.
-- 양쪽 모두 조건이 일치하지 않는 것까지 모두 결합하여 출력한다.
```

> 결과 예시

| 테이블A | 테이블B |
| --- | --- |
| 1 | null |
| 2 | null |
| null | 4 |
| null | 5 |
| 6 | 6 |

#### 오라클에서 (+)를 사용하여 Outer Join 수행

> 1) 하나의 컬럼을 사용하는 경우

_emp LEFT OUTER JOIN DEPT_ 와 동일하다고 이해하자.

![IMAGES](../images/Join3.png)

> 2) 두개 이상의 컬럼을 사용하는 경우

조인 조건인 2개에 모두 (+)를 사용해줘야한다.

b.useryn(+) = 'Y' 또한 (+)를 사용해줘야한다. 만약 붙이지 않으면, LEFT OUTER JOIN으로 조회되지 않는다.

![IMAGES](../images/Join4.png)

## Semi Join (세미조인)

- 서브쿼리를 사용하여 서브쿼리에 존재하는 데이터만 메인쿼리에서 추출하는 조인이다.

- IN, NOT IN, EXISTS, NOT EXISTS 등을 사용한다.

- 결과적으로 중복되는 결과값을 제거한다.

- EXISTS는 서브쿼리에서만 사용이 가능하다.

> IN 사용 예제 쿼리

```
SELECT * 
  FROM department d 
 WHERE d.dept_id IN ('1', '2', '3');
```

> EXISTS 사용 예제 쿼리

모든 row를 찾지 않는다는 장점이 있다.  EXISTST 서브 쿼리에 해당하는 첫번째 데이터 A를 찾았다고 가정하자. 그럼 A를 찾은 이후 다른 row는 찾지 않는다. 그리고, 다시 두번째 데이터 B를 찾는다.

```
SELECT * 
  FROM department d 
 WHERE EXISTS 
        (SELECT 1 
           FROM student s 
          WHERE d.dept_code = s.dept_code)
```

#### Reference.

[https://spidyweb.tistory.com/149](https://spidyweb.tistory.com/149)  
[https://rh-cp.tistory.com/44](https://rh-cp.tistory.com/44)  
[https://gent.tistory.com/289](https://gent.tistory.com/289)  
[https://velog.io/@godkimchichi/Oracle-%EC%84%B8%EB%AF%B8%EC%A1%B0%EC%9D%B8](https://velog.io/@godkimchichi/Oracle-%EC%84%B8%EB%AF%B8%EC%A1%B0%EC%9D%B8)