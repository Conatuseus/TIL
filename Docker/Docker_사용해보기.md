Docker의 명령은 `docker run`, `docker push`와 같이 **`Docker <명령>`** 형식이며, 항상 root 권한으로 실행해야 합니다.

# search 명령으로 이미지 검색하기

Docker는 [Docker Hub](https://registry.hub.docker.com)을 통해 이미지를 공유하는 생테계가 구축되어 있습니다.
유명 리눅스 배포판, 오픈 소스 프로젝트(Redis, Nginx 등)의 Docker 이미지는 모두 Docker Hub에서 구할 수 있습니다.

![image.png](https://images.velog.io/post-images/conatuseus/650350b0-1775-11ea-bca6-193faaf8cc7a/image.png)



# pull 명령으로 이미지 받기

Docker Hub에서 우분투 리눅스 이미지를 받아보겠습니다.

`sudo docker pull ubuntu:latest`

Docker pull <이미지 이름>:<태그> 형식입니다. latest를 설정하면 최신 버전을 받습니다.
ubuntu:14.04, ubuntu:12.10처럼 태그를 지정해 줄 수도 있습니다.


![image.png](https://images.velog.io/post-images/conatuseus/767df480-1775-11ea-af24-bff57c817829/image.png)


# images 명령으로 이미지 목록 출력하기

이제 받은 이미지의 목록을 출력해보겠습니다.

`sudo docker images`

![image.png](https://images.velog.io/post-images/conatuseus/7f48a060-1775-11ea-af24-bff57c817829/image.png)


**docker images** 명령은 모든 이미지 목록을 출력합니다.



# run 명령으로 컨테이너 생성하기

이미지를 컨테이너로 생성한 뒤 Bash 셸을 실행해보겠습니다.

`sudo docker run -i -t --name hello ubuntu /bin/bash`

**docker run <옵션> <이미지 이름> <실행할 파일> 형식입니다.

여기서는 ubuntu 이미지를 컨테이너로 생성한 뒤 ubuntu 이미지 안의 /bin/bash를 실행합니다. 이미지 이름 대신 이미지 ID를 사용해도 됩니다.

> **-i**(interactive), **-t**(Pseudo-try) 옵션을 사용하면 실행된 Bash 셸에 입력 및 출력을 할 수 있습니다.
>
> **--name** 옵션으로 컨테이너 이름을 지정할 수 있습니다. 이름을 지정하지 않으면 Docker가 자동으로 이름을 생성하여 지정합니다.

![image.png](https://images.velog.io/post-images/conatuseus/87b59be0-1775-11ea-bca6-193faaf8cc7a/image.png)


이제 호스트 OS와 완전히 격리된 공간이 생성되었습니다. `cd`,`ls` 명령으로 컨테이너 내부를 보면 호스트 OS와는 다르다는 것을 알 수 있습니다. `exit`를 입력하여 Bash 셸에서 빠져나옵니다. 우분투 이미지에서 /bin/bash 실행 파일을 직접 실행했기 때문에 여기서 빠져나오면 컨테이너가 정지됩니다.



# ps 명령으로 컨테이너 목록 확인하기

`sudo docker ps -a`

**docker ps** 형식입니다. **-a** 옵션을 사용하면 정지된 컨테이너까지 모두 출력하고, 옵션을 사용하지 않으면 실행되고 있는 컨테이너만 출력합니다.

![image.png](https://images.velog.io/post-images/conatuseus/904dc610-1775-11ea-b432-675960e51cd4/image.png)




# start 명령으로 컨테이너 시작하기

`sudo docker start hello`

**docker start <컨테이너 이름>** 형식입니다. 컨테이너 이름 대신 컨테이너 ID를 사용해도 됩니다.


![image.png](https://images.velog.io/post-images/conatuseus/98448020-1775-11ea-b432-675960e51cd4/image.png)


hello 컨테이너가 시작된 것을 확인할 수 있습니다.



# restart 명령으로 컨테이너 재시작하기

OS 재부팅처럼 컨테이너를 재시작해보겠습니다

`sudo docker restart hello`

**docker restart <컨테이너 이름>** 형식입니다. 컨테이너 이름 대신 컨테이너 ID를 사용할 수 있습니다.


![image.png](https://images.velog.io/post-images/conatuseus/9d34d800-1775-11ea-bca6-193faaf8cc7a/image.png)




# attach 명령으로 컨테이너에 접속하기

이제 시작한 컨테이너에 접속해보겠습니다.

`sudo docker attach hello`

**docker attach <컨테이너 이름>** 형식입니다. 컨테이너 이름 대신 컨테이너 ID를 사용할 수 있습니다.

우리는 /bin/bash를 실행했기 때문에 명령을 자유롭게 입력할 수 있지만, DB나 서버 애플리케이션을 실행하면 입력은 할 수 없고 출력만 보게 됩니다.

Bash 셸에서 `exit` 또는 Ctrl+D 를 입력하면 컨테이너가 정지됩니다.


![image.png](https://images.velog.io/post-images/conatuseus/a1538800-1775-11ea-af24-bff57c817829/image.png)




# exec 명령으로 외부에서 컨테이너 안의 명령 실행하기

현재 컨테이너가 /bin/bash로 실행된 상태입니다. 이번에는 /bin/bash를 통하지 않고 외부에서 컨테이너 안의 명령을 실행해보겠습니다.

`sudo docker exec hello echo "Hello World"`

**docker exec <컨테이너 이름> <명령> <매개변수>** 형식입니다. 컨테이너 이름 대신 컨테이너 ID를 사용할 수 있습니다. 컨테이너가 실행되고 있는 상태에서만 사용할 수 있으며 정지된 상태에서는 사용할 수 없습니다.

컨테이너 안의 `echo` 명령을 실행하고 매개 변수로 "Hello World"를 지정했기 때문에 Hello World가 출력됩니다. docker exec 명령은 이미 실행된 컨테이너에 apt-get, yum 명령으로 패키지를 설치하거나, 각종 데몬을 실행할 때 활용할 수 있습니다.



# stop 명령으로 컨테이너 정지하기

이번에는 컨테이너를 정지해보겠습니다. 


![image.png](https://images.velog.io/post-images/conatuseus/a84bbdd0-1775-11ea-af24-bff57c817829/image.png)


`sudo docker ps` 명령을 통해 hello 라는 이름을 가진 컨테이너가 실행 중인 것을 확인할 수 있습니다.

그리고 `sudo docker stop hello` 명령을 통해 컨테이너를 정지하고 다시 확인해보니 hello 컨테이너가 정지한 것을 알 수 있습니다.

**docker stop <컨테이너 이름>** 형식입니다.



# rm 명령으로 컨테이너 삭제하기

생성된 컨테이너를 삭제해보겠습니다.

`sudo docker rm hello`

**docker rm <컨테이너 이름>** 형식입니다.


![image.png](https://images.velog.io/post-images/conatuseus/ac3924a0-1775-11ea-af24-bff57c817829/image.png)


삭제 명령 후에 `sudo docker ps -a` 명령을 실행해보면 hello 컨테이너가 삭제된 것을 볼 수 있습니다.



# rmi 명령으로 이미지 삭제하기

이번에는 이미지를 삭제해보겠습니다.

`sudo docker rmi ubuntu:latest`

**docker rmi <이미지 이름>:<태그>** 형식입니다. 이미지 이름 대신 이미지 ID를 사용해도 됩니다.

**docker rmi ubuntu**처럼 이미지 이름만 지정하면 **태그**는 다르지만 ubuntu 이름을 가진 모든 이미지가 삭제됩니다.


![image.png](https://images.velog.io/post-images/conatuseus/b01c5240-1775-11ea-af24-bff57c817829/image.png)


