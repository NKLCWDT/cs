# Portable Service Abstraction

- PSA 는 환경과 세부 기술의 변화와 관계 없이 일관된 방식으로 기술에 접근할 수 있게 해준다.
- POJO 로 개발된 코드는 특정 기술에 직접 노출되어 만들어지면 안되는데, 이를 위해 스프링에서 제공하는 대표적인 기술이 `PSA` 이다.
- 서비스 추상화를 위해 필요한 기술은 `DI` 뿐이다.
  - 즉, PSA 는 DI 를 응용 하는 방법이다.
- 추상화 계층을 사용해서 `어떤 기술을 내부에 숨기고` 개발자에게 `편의성을 제공`해주는 것을 서비스 추상화(Service Abstraction) 이라고 한다.
- 서비스 추상화로 제공되는 기술을 `다른 기술 스택으로 간편하게 바꿀 수 있는 확장성`을 갖고 있는 것이 PSA 이다.

## PSA 핵심 

- __특정 세부적인 기술에 종속되지 않게 해준다.__
- __테스트가 어렵게 만들어진 API 나 설정을 통해 주요 기능을 외부에서 제어하게 만들고 싶은 경우 사용할 수 있다.__

## PSA 적용하기

### 기존 코드

```java
public class RepositoryRank {

  public int getPoint(String repositoryName) throws IOException {
    GitHub github = GitHub.connect();
    GHRepository repository = github.getRepository(repositoryName);
    
    int points = 0;
    if (repository.hasIssues()) {
      points += 1;
    }
    
    if (repository.getReadme() != null) {
      points += 1;
    }
  
    // 생략
  }
}
```

현재 getPoint() 메서드는 테스트하기 어려운 상태이다. 

바로 getPoint() 내부에서 GitHub API 를 호출하기 때문이다. 실제로 테스트할 때마다 API 를 매번 호출하게 할 것인가? 

API 를 매번호출하게 되면 GitHub API 자체에 제한이 있을 수도 있고, API 를 호출하는 과정에서 실패가 일어날 수도 있으며, 속도도 느릴 것이다.

이러한 문제를 PSA 를 사용하여 해결할 수 있다.

### 리팩토링

테스트하기 어려운 부분(`GitHub github = GitHub.connect()`)을 추상화하는 것이다.

```java
interface GitHubService {
  GitHub connect();
}
```
```java
public class DefaultGitHubService implements GitHubService {
  @Override
  public GitHub connect() {
    return GitHub.connect();
  }
}
```
```java
public class RepositoryRank {

  private final GitHubService gitHubService;
  
  public RepositoryRank(final GitHubService gitHubService) {
    this.gitHubService = gitHubService;
  }

  public int getPoint(String repositoryName) throws IOException {
    GitHub github = gitHubService.connect();
    GHRepository repository = github.getRepository(repositoryName);
    
    // 생략
  }
}
```

PSA 를 적용하여 리팩토링 하였더니 `테스트하기 쉬운 코드`가 되었다.

## References

- [토비의 스프링 3](#)
- [스프링 제대로 공부했는가?](https://www.youtube.com/watch?v=bJfbPWEMj_c&t=12s)
