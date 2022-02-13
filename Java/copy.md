## Shallow Copy v.s Deep Copy 
객체지향 프로그래밍에서는 object copying이 기존에 있던 객체의 복사본을 만드는 것이다. 이렇게 객체의 복사본을 만드는데 세가지 방법이 있다. 

이 방법들을 코드를 통해 알아보자.

```java
public class Student {
    String name;
    int grade;

    public Student(String name, int grade) {
        this.name = name;
        this.grade = grade;
    }

    public void addExtracredit(int extracredit) {
        this.grade += extracredit;
    }
}
```

<br>

### Shallow Copy
- 객체의 주소값을 복사한다.

```java
void shallowCopy() {
    Student jihye = new Student("jihye", 70);
    Student jihyeCopy = jihye;

    jihye.addExtracredit(10);
}
```
name은 jihye고 grade는 70인 Student 객체의 인스턴스 jihye을 생성한다. 그리고 jihyeCopy에 이 인스턴스를 복사한다. 이후 jihye인스턴스에 엑스트라크렛딧 10점을 더해준다. 

이렇게 한 후 두 인스턴스 모두 name: jihye, grade:80 을 가지고 있다. 

<img src="/images/shallowCopy.png" width="550" height="350"/><br>   
  
이렇게 jihye와 jihyeCopy모두 같은 주소값을 가지고 있어도 한 인스턴스의 값만 바뀌어도 다른 인스턴스의 값까지 바뀌게 된다.
<br>

### Deep Copy
- 실제 값을 새로운 메모리 공간에 복사 하는 것
- deep copy를 구현하는 방법
  - 복사 생성자 이용
  - 직접 객체 생성해 복사
  - cloneable구현해 clone 재정의

1. 복사 생성자 또는 복사 팩토리 이용해서 deep copy
```java
public class Student {
    // 복사 생성자
    public Student(Student student) {
        this.name = student.name;
        this.grade = student.grade;
    }
    
     // 복사 팩터리
    public static Stuent newInstance(Student student) {
        Student s = new Student();
        s.name = student.name;
        s.grade = student.grade;
        return s;
    }
}
```
2. 직접 객체 생성해서 복사하는 방법
```java
void deepCopy() {
    Student jihye = new Student("jihye", 90);//새로운 객체 생성
    Student jihyeCopy = new Student();
    jihyeCopy.setName(jihye.getName());
    jihyeCopy.setGrade(jihye.getGrade());
}
```

3. clone을 이용해서 복사하는 방법
```java
public class Student implements Cloneable {
  
    String name;
    int grade;

    ... 

    @Override
    protected Studetent clone() throws CloneNotSupportedException {
      return (Student) super.clone();
    }
}
```

```java
void deepCopy() throws CloneNotSupportedException{
    Student jihye = new Student("jihye", 90);
    Student jihyeCopy = jihye.clone();
}
```

clone() 재정의는 조심해서 해야한다. 그리고, cloneable을 재정의해 clone()을 구현하는것 보다 복사 생성자나 복사 팩토리를 이용해 Deep Copy를 하는것이 좋다고 한다. [자세한 내용](https://jackjeong.tistory.com/30)

<img src="/images/deepCopy.png" width="550" height="350"/><br>   

이렇게 각자 다른 주소값을 가지면서 값들만 복사해온다. 여기서 jihyeCopy의 name을 jiseok으로 바꿔도 jihye의 name은 jihye로 그대로 남는다.  

<img src="/images/deepCopy2.png" width="550" height="350"/><br> 

<br><br>


<!-- ## Lazy Copy
Shallow Copy 와 Deep Copy둘다 섞어서 쓰는 방법이다. 처음에는 shallow Copy를 사용해서 객체를 복사한다. 이후에 기존에 있던 객체가 바뀔때 해당 객체가 공유되고 있는지 확인 후에 필요시에는 Deep Copy를 한다.  -->

## equals메서드
두 객체가 동일한지 검사해 그 결과를 boolean값으로 알려주는 역할을 한다. 모든 클래스의 부모인 object class의 메서드이다. 

```java
public static void main(String[] args){
    Student s1 = new Student("jihye", 90);
    Student s2 = new Student("jihye", 90);

    if(s1.equals(s2)){
        System.out.println("s1과 s1은 같은 학생이다.")
    }
    //여기서 s1.equals(s2)를 하면 false가 나온다. 서로의 주소값이 다르기 때문이다.


    s1 = s2;
    if(s1.equals(s2)){
        System.out.println("s1과 s1은 같은 학생이다.")
    }
    //반면, 여기서는 같다고 나온다. s1이 이제 s2의 주소값을 가지고 있기 때문이다.

}
```
이렇게 두 객체의 주소값이 같은지 확인한다. 여기서 equals 메서드를 이용해서 두 사람의 이름이 같으면 true를 결과로 나오게 하기 위해서는 Student class에서 equals메서드를 오버라이딩 하면 된다.

String클래스 또한 equals메서드를 오버라이딩해서 equals로 String 값이 같은지 비교할 수 있다.

> 이처럼 equals메서드를 필요에 따라 오버라이딩해서 사용할 수 있다.

<br>


> __Object 클래스에 정의되어 있는 equals__  
```java
public boolean equals(Object obj){
    return(this == obj);
}
```
이렇게 Object클래스 내부에 있는 equals메서드를 보면 객체의 참조값을 서로 비교한 후 결과를 return 한다.


## hashCode메서드
heap에 저장된 객체의 메모리 주소값을 해싱기법을 사용해서 변환해 생성한 객체의 고유값이다. 일반적으로는 다른 두 객체가 같은 해쉬코드를 가질 수 있지만, Object클래스에 정의된 hashCode메서드는 객체의 주소값을 이용해서 해시코드를 만들어 반환하기 때문에 한번의 실행에서 서로 다른 두 객체는 같은 해시 코드를 가질 수 없다.

<br>

### equals메서드와 hashcode메서드
같은 객체는 같은 메모리 주소를 가지는 것을 의미 하므로 같은 객체는 같은 해시코드를 가저야 한다. 그래서 equals메서드를 오버라이딩하면서 hashcode메서드도 같이 오버라이딩 하는 것이 일반적이다. equals메서드를 오버라이딩한 String클래스 역시 hashCode메서드도 String의 값이 같으면 같은 hashCode를 반환하도록 오버라이딩 되어있다.


<br><br><br>

>참고자료
- https://zzang9ha.tistory.com/372
- https://jackjeong.tistory.com/100
- https://mangkyu.tistory.com/101
- https://nesoy.github.io/articles/2018-06/Java-equals-hashcode
- http://www.yes24.com/Product/Goods/24259565
