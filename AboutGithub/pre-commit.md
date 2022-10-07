# Git Hooks
깃에는 특정 상황에 특정 스크립트를 실행할 수 있도록 `hooks`이라고 하는 기능을 제공한다.  
이는 모든 깃헙 레포지토리에 기본적으로 들어가 있으며, 각 레포지토리의 `.git/hooks/` 폴더에서 샘플 파일들을 확인할 수 있다.  

훅에 대해서는 나중에 기회가 된다면 알아보기로 하고, 그 중 하나인 `pre-commit`을 정리한다.

# Pre-commit
`pre-commit`이란 말 그대로 `commit`하기 전에 스크립트를 실행하는 것이다.  
커밋 전에 실행하면 가장 좋은 것이 무엇일까? 바로 코드의 품질 검사이다.  
보통 코드를 작성하고 커밋을 올리기 전에 각 언어에 맞는 `lint`를 한번 돌려보고 코드가 올바른 규칙에 맞게 작성되었는지, 확인한 후에 커밋을 할 텐데, 매번 `bash` 명령어를 입력하는 것도 일이다...  
그래서 많은 사람들이 pre-commit을 코드 검사에 사용하고 있다.  
물론 용량이 큰 파일을 압축한다거나 하는 등의 다른 용도로도 사용이 가능하다.  

## 사용법
사실 git hook에서 제공하는 pre-commit을 사용하기위해서는 실행가능한 파일을 작성해야 하는데, 이것조차 좀 번거로운 감이 있다.  
pre-commit 패키지를 이용해 이조차도 간단하게 설정할 수 있다.  

### 설치
pre-commit 패키지는 여러 방법으로 설치가 가능하다.  

```bash
$ pip install pre-commit
```  

```bash
$ brew install pre-commit
```  

```bash
$ conda install -c conda-forge pre-commit
```  

### 시작
pre-commit 설정을 위해서는 `.pre-commit-config.yaml` 파일이 필요하다.  

```bash
$ pre-commit sample-config > .pre-commit-config.yaml
```  

그럼 다음과 같은 파일이 생성된다.  

```yaml
# .pre-commit-config.yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
    -   id: check-yaml
    -   id: end-of-file-fixer
    -   id: trailing-whitespace
```  

`repos`에는 여러가지 `repo`들이 들어갈 수 있다.  
`repo`에는 말 그대로 내가 사용하려는 `hook`을 제공하는 레포지토리의 주소를 넣으면 된다.  
기본적으로는 `pre-commit`의 레포지토리가 들어가 있다.  
`rev`를 통해 해당 레포지토리에서 릴리즈된 버전 중 내가 사용하고 싶은 버전을 사용하면 된다.  
`hooks`에는 적용할 `hook`을 적는다. 여기서는 세 가지 `hook`이 적용되고 있다.  
- `check-yaml`: 모든 yaml파일의 구문 검증
- `end-of-file-fixer`: 모든 파일에 new line이 없으면 추가
- `trailing-whitespace`: 모든 라인의 끝에 공백이 있으면 삭제

### 설정
위 상황에서 `pre-commit`이 제공하는 `hooks`를 더 추가할 수도 있겠지만, 새로운 레포지토리에서 `hook`을 가져와서 추가해보자.  
추가할 레포지토리는 파이썬 코드 에서 `PEP8`규칙을 확인해주는 `flake8`이다.

```yaml
# .pre-commit-config.yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
    -   id: check-yaml
    -   id: end-of-file-fixer
    -   id: trailing-whitespace
-   repo: https://github.com/PyCQA/flake8
    rev: 5.0.4
    hooks:
    -   id: flake8
```  

방법은 간단하다 추가할 `repo`를 붙이고, 버전과 `hook`을 명시해주면 된다.  
[여기](https://pre-commit.com/hooks.html)서 pre-commit을 적용할 수 있는 다양한 레포지토리들을 확인할 수 있고, 보통은 깃헙에 들어가면 최신 버전을 알 수 있다.  

설정한 훅들은 커밋시에 위에서부터 적용이 된다.  

또한 버전을 업데이트 하고싶을 경우 명령어를 통해 자동으로 업데이트 할 수도 있다.  
```bash
$ pre-commit autoupdate
```  

### 적용
이렇게 적용할 `hook`들을 설정했다면 적용해야한다.  

```bash
$ pre-commit install
```  

여기까지 했다면 모든 설정이 끝났다.  
파이썬 코드를 작성하고 commit을 작성하면 설정해둔 `flake8`이 오류를 발견할 것이다. 물론 고쳐지지는 않고 그대로 commit이 되긴한다.  
이후에는 직접 수정하거나 `black`과 같이 수정해주는 툴을 사용해서 수정 후 다시 commit을 작성하면 된다.  