## call by value, call by reference

### Call by value

메소드 호출 시에 사용되는 인자의 메모리에 저장되어 있는 값(value)을 복사하여 보낸다.  
`int a = 3`이 있으면 메소드에서 인자값을 받을 때 a라는 자체에 주소를 받는게 아니라 a의 값인 3을 받아 처리하는 방식이다.

아래에 예시는 call by value에 대한 설명이다.

```java

public class Diffrence {
    
    static void swapS(String one, String two) {
        String temp = one;
        one = two;
        two = temp;
    }
    
    public static void main(String args[]) {
        String a = new String("안녕");
        String b = new String("잘가");

        System.out.println("Before");
        System.out.println(a+b); // 안녕잘가
        swapS(a,b);
        System.out.println("After");
        System.out.println(a+b); // 안녕잘가
    }
}
```

스왑 메소드를 이용해서 서로를 바꿔준 것 같지만 바뀌지 않는다.

왜냐하면 String one에는 a에 주소가 아닌 a값에 주소가 복사되고 마찬가지로 two에도 b의 주소가 아닌 b값에 주소가 복사되어진다.

직접적인 참조를 넘긴 게 아닌, 주소 값을 복사해서 넘기기 때문에 자바에서는 기본형, 참조형 모두 call by value 방식을 사용한다.

아래 예시에서는 참조형을 예로 들겠다.

![image](https://user-images.githubusercontent.com/70622731/160238975-8c506ac9-c937-4d85-b469-48459f6b8d2a.png)

이렇게 메인 메소드의 a,b와 swapS 메소드의 one, two는 서로 같은 값을 참조할 뿐이다.

![image](https://user-images.githubusercontent.com/70622731/160238983-dd3c8448-fe36-4221-8d0d-37cf854b1da5.png)

그러므로 swap메소드를 실행해도 main값에 영향을 주지 못한다.

만약 call by reference 방식 이였으면 메소드내에서 one,two가 a,b의 주소값을 받아 서로 값이 바뀌니 메소드 수행을 하고나서도 a,b가 바뀌어 있을 것이다.

이러한 효과를 내는 방법은 있다.

```java

public class Diffrence {
    int value;
    
    Diffrence(int value) {
        this.value = value;
    }
    
    public static void swap(Diffrence one, Diffrence two) {
        int temp = one.value;
        one.value = two.value;
        two.value = temp;
    }
    
    public static void main(String[] args) {
        Diffrence a = new Diffrence(10);
        Diffrence b = new Diffrence(20);

        System.out.println("swap() 호출 전 : a = " + a.value + ", b = " + b.value);
        swap(a,b);
        System.out.println("swap() 호출 후 : a = " + a.value + ", b = " + b.value);
    }
}
```
결과값은  
swap() 호출 전 : a = 10, b = 20  
swap() 호출 후 : a = 20, b = 10

이렇게 값이 바뀌는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/70622731/160238995-862787d3-4a79-4a23-8305-cf91e775e7d4.png)

마찬가지로 one과 two는 값의 주소를(call by value) 복사 받아 같은 인스턴스를 참조하는 것 이지만

![image](https://user-images.githubusercontent.com/70622731/160239001-96bc5061-0c63-43e2-ad51-a1ae45a45b99.png)

이렇게 참조 되어지는 값을  마치 call by reference가 이루어진 것 처럼 보인다.

<br>

---

### Reference

https://sleepyeyes.tistory.com/11

https://deveric.tistory.com/92
