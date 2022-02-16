# git merge vs git rebase

- git merge 를 사용하게되면 merge 커밋이라는 것이 생깁니다.
- 반면에, git rebase 를 사용하게되면 merge 커밋이 없기 때문에 히스토리 관리를 깔끔하게 할 수 있습니다.
- git rebase 는 `base` 라는 개념(`커밋 해시값들의 묶음`이라고 생각합니다.)을 활용하여 원격 저장소와 Local Repository 의 해시값을 비교해서 Merge 가 가능한지 판단합니다.
