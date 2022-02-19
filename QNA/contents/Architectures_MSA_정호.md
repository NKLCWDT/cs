# MSA 와 모놀리식의 차이를 설명해 주세요.

- __Monolithic Architecture__
  - 배포 및 테스트가 한 애플리케이션에서 수행되기 때문에, 작은 서비스에 적합합니다.
  - 하지만, 시스템이 커지고 여러 컴포넌트들이 추가되면 `빌드/테스트 시간이 길어지고`, `선택적 확장이 불가능`하며, 하나의 서비스가 마비되면 `모든 서비스에 영향`을 준다는 단점이 있습니다.
  - ![mono](https://user-images.githubusercontent.com/47518272/154800774-d6546979-f5bb-4754-81d9-057624a82e53.png)
- __Microservice Architecture__
  - MSA 는 복잡한 웹 시스템에 맞춰 개발된 API기반의 서비스 지향적 아키텍처 스타일입니다. 
  - MSA 의 철학은 `한 가지만, 아주 잘 처리하자` 이며, 단일책임원칙(SRP)를 중시합니다.
  - 서비스별 DB 를 따로 가져갈 수 있습니다.
  -  다른 서비스 컴포넌트에 대한 의존성이 없이 서비스를 독립적으로 개발 및 배포/운영 할 수 있지만, 다른 컴포넌트의 데이터를 API통신을 통해 가져와야 하기 때문에 성능상의 문제가 발생 할 수 있습니다.
  -  ![msa](https://user-images.githubusercontent.com/47518272/154800927-d5dbef9d-8f69-45fc-a5ab-1d433da666c7.png)

## References

- http://clipsoft.co.kr/wp/blog/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98msa-%EA%B0%9C%EB%85%90/
