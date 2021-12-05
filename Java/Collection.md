## Java의 Collection
자바에서 여러가지 반복적인 귀찮은 일들을 쉽게 하기 위해서 다양한 class들을 제공한다. 그 중 하나가 Collection Framework이다.

### Collection Framework
데이터를 다루는 표준화된 class를 제공해주는 프레임워크다. 주로 List, Set,Map으로 분류한다. 이렇게 세가지 중에 List와 Set은 서로 공통된 부분이 많아 따로 collection interface가 정의되어 있다.

>Interface란 다른 class작성시 틀을 제공해 조금 더 동일한 목적에 동일한 기능을 수행할 수 있도록해준다. 또 다형성을 높여 개발에 유지보수를 쉽게 하게 해준다.

![images](/images/collection.png)

__List__   
- 중복을 허용하면서 저장순서가 유지되는 컬레션을 구현하는데 사용.
- 리스트 인터페이스를 구현하는 클래스는 ArrayList, LinkedList, Vector, Stack등이 있다.

__Set__
- 중복을 허용하지 않고 저장 순서가 유지되지 않는 컬렉션을 구현하는데 사용
- Set인터페이스를 구현한 클래스로는 HashSet, TreeSet

__Map__
- key와 value 쌍으로 이루어져 묶어서 저장하는 컬렉션 클래스를 구현하는데 사용
- key는 중복될 수 없지만 값은 중복 허용
- Map인터페이스를 구현한 클래스로는 hashtable, hashmap, sortedmap, tree map등이 있다.




