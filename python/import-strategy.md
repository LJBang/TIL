# Import Strategy

파이썬 프로젝트를 진행할 때, 최상위 디렉토리에서 모든 파일을 실행할 수도 있지만,  
테스트용으로 내부 디렉토리에서 실행해야 하는 경우도 많다.  
이 때, 임포트 구조 때문에 애로사항이 생긴다.  

## 패키지 구조
```shell
project
    ├── utils
    │   ├── __init__.py
    │   └── constants.py
    ├── preprocess
    │   ├── __init__.py
    │   └── preprocess.py
    └── main.py
```  
- `main.py`는 최상위 실행파일, `preprocess/preprocess.py`를 호출한다.
- `preprocess.py`는 전처리를 수행하는 함수가 있는 파일, `utils/constants.py`를 호출한다.
    - 함수 테스트를 위해서 직접 실행하면서 테스트 필요


## 전략 1
`sys.path.append`를 이용한다.
```python
# preprocess.py
import sys
sys.append('../)
from utils import constants
```  

단점: 경로가 직관적으로 안보이고, 각종 린트에 잡힌다.. (import문이 최상단에 와야함)

## 전략 2
`project`디렉토리에서 `python -m preprocess.preprocess`로 실행.
이 때 `preprocess.py`파일은 다음과 같다.
```python
from utils import constants
```  