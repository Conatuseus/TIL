
# .gitignore란?
 **깃에서 특정 파일 혹은 디렉토리를 관리 대상에서 제외할 때 사용하는 파일.**
 이 파일 안에 기입된 내용들은 모두 깃에서 관리하지 않겠다는 것을 의미합니다. 예를 들어 자동으로 생성되는 로그파일, 프로젝트 설정 파일 등을 관리 대상에서 제외할 수 있습니다.

# 인텔리제이에 플러그인 설치하기
인텔리제이에서는 .gitignore 파일에 대한 기본적인 지원이 없는 대신 플러그인에서 .gitignore 지원을 하고 있습니다.

.ignore 플러그인에서 지원하는 기능
- 파일 위치 자동완성
- 이그노어 처리 여부 확인
- 다양한 이그노어 파일 지원(.gitignore, .npmignore, .dockerignore 등등)


![image.png](https://images.velog.io/post-images/conatuseus/12694ff0-1f39-11ea-b8fb-8b8d39340afe/image.png)

Plugins의 Marketplace탭에서 .ignore 플러그인을 설치해줍니다.

# 사용하기

![image.png](https://images.velog.io/post-images/conatuseus/68572540-1f39-11ea-af4c-3f9f13d48807/image.png)

위와 같이 .gitignore 파일을 최상단 디렉토리에 생성합니다.
그리고 아래와 같이 작성합니다.
.gitignore 파일의 `.gradle`, `.idea`는  .gradle 파일과, .idea파일을 깃 체크 대상에서 제외한다는 뜻입니다.

![image.png](https://images.velog.io/post-images/conatuseus/c7d7ff80-1f39-11ea-af4c-3f9f13d48807/image.png)

> 문법은 [여기](https://nesoy.github.io/articles/2017-01/Git-Ignore)에 잘 나와있습니다.


# .gitignore가 정상적으로 작동하지 않는다면?

~~~ java
// 현재 Repository의 cache를 모두 삭제
git rm -r --cached .
// .gitignore에 넣은 파일 목록들을 제외하고 다른 모든 파일을 다시 track하도록 설정
git add .
// commit
git commit -m "fixed untracked files"
~~~


> 참고
>
> https://jojoldu.tistory.com/307
> https://nesoy.github.io/articles/2017-01/Git-Ignore
