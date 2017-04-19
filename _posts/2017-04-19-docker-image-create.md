---
layout: post
title: "Docker의 image 만드는 과정"
author: "Pilsner"
date: 2017–04–19 11:26:00 +0900
categories: docker make image commit
---

## Docker의 image 만드는 과정
Docker에서 image를 만드는 과정은 크게 2가지로 분류됩니다. 첫 번째는 Dockerfile을 작성하여 image를 생성하는 방법이 있습니다. 두 번째는 container로부터 docker commit 하여 image를 만드는 작업입니다. 사실 내부적으로 살펴보면 사실 둘 다 같은 방법입니다.
## Dockerfile에서부터 image를 생성
예시로 아주 간단한 Dockerfile 하나를 만들어보겠습니다.
```
FROM ubuntu
LABEL maintainer "darkrasid@gmail.com"
RUN touch /tmp/test_file
CMD ["/bin/bash"]
```
위의 Dockerfile은 어떤식으로 이미지를 만들까요?
우선 layer가 몇개 생길지 한번 생각해볼까요? 정답은 4개입니다. FROM 절에서 하나 LABEL에서 둘, RUN이 3번째 마지막으로 CMD를 등록하면서 4번째 layer가 생성됩니다.
그럼 이번 Dockerfile은 몇개가 생성될까요?
```
FROM ubuntu
LABEL maintainer "darkrasid@gmail.com"
RUN touch /tmp/test_file && touch /tmp/test_file2 && touch /tmp/test_file3
CMD ["/bin/bash"]
```
이번 Dockerfile도 똑같이 layer는 4개입니다. 즉 layer는 명령어의 복잡도나 수에 관계없이
Dockerfile instructor 의 수에 따라 결정납니다. 왜 그럴까요?
첫번째 Dockerfile을 이용하여 docker image를 만들어보겠습니다.
```
docker build -t how_to_make_image_1 .
```
```
Sending build context to Docker daemon 2.048 kB
Step 1/4 : FROM ubuntu
 — -> 6a2f32de169d
Step 2/4 : LABEL maintainer "darkrasid@gmail.com"
 — -> Running in 17f3d260c524
 — -> ea7651d29bce
Removing intermediate container 17f3d260c524
Step 3/4 : RUN touch /tmp/test_file
 — -> Running in 737b346f5692
 — -> 0ff937a46f57
Removing intermediate container 737b346f5692
Step 4/4 : CMD /bin/bash
 — -> Running in 3b730dcef4d2
 — -> a0e940c0f001
Removing intermediate container 3b730dcef4d2
Successfully built a0e940c0f001
```
생성 로그를 보면 `FROM ubuntu ` 부분을 제외하곤 `Running in {hash_id}` 부분과 
`Removing intermediate container {hash_id}` 부분이 보입니다. 좀 이상하지 않나요?
아직 image도 생성을 안했는데 무슨 container를 만들어서 running을 하고 뭘 지우는 걸까요?

이미지를 생성하는 과정은 먼저 container를 만드는 것에서부터 시작합니다. step 2는 FROM 에서 들어온 base 이미지로부터 container를 하나 실행시키고, 그 container내에서 label을 생성합니다. 그 후에 그 container로 부터 image를 commit해서 image를 생성합니다. 그리곤 쓸모없어진 container를 삭제합니다.
정확한 명령어가 이런식으로 이뤄지진 않겠지만 이해를 위해 굳이 명령어로 만들어보면 (위의 log에서 step1과 step2의 hash_id를 토대로 만들어졌으니 같이 보세요.)
```
docker run 6a2f32de169d ubuntu /bin/sh -c #(nop) LABEL maintainer "darkrasid@gmail.com" 
=> 17f3d260c524 container 생성
docker commit 17f3d260c524 ea7651d29bce # 위에서 만든 container를 토대로 image 생성
docker rm -fv 17f3d260c524 # 필요없어진 container 삭제
```
이런 과정을 거치면 하나의 container(17f3d260c524)가 생성됐다 삭제가 됐고, ea7651d29bce image가 생성이 됩니다.
Dockerfile에서 image가 생성되는 과정은 위의 과정을 반복하는 것입니다. 다시 위의 docker build log를 살펴보면
```
Sending build context to Docker daemon 2.048 kB
Step 1/4 : FROM ubuntu
 — -> 6a2f32de169d => base image를 토대로 만들어진 image
Step 2/4 : LABEL maintainer "darkrasid@gmail.com"
 — -> Running in 17f3d260c524 => 위의 base image(6a2f32de169d)를 토대로 명령을 날려 만들어진 container
 — -> ea7651d29bce => 그 container를 토대로 만들어진 image
Removing intermediate container 17f3d260c524 => 쓸모없어진 container의 삭제
Step 3/4 : RUN touch /tmp/test_file 
 — -> Running in 737b346f5692 => 위에서 생성된 image(ea7651d29bce)를 토대로 명령을 날려 만들어진 container
 — -> 0ff937a46f57 => 그 container를 토대로 만들어진 image 
Removing intermediate container 737b346f5692 => 쓸모없어진 container의 삭제
Step 4/4 : CMD /bin/bash => 위의 과정 반복
 — -> Running in 3b730dcef4d2
 — -> a0e940c0f001
Removing intermediate container 3b730dcef4d2
Successfully built a0e940c0f001 => 최종적으로 생성된 image, 그리고 쓸모없어진 전 image들 삭제
```
이렇게 생성됩니다. 
진짜 그런지 한번 확인해볼까요?
## 확인!
```
docker history how_to_make_image_1
```
```
IMAGE          CREATED              CREATED BY                                       SIZE      COMMENT
a0e940c0f001   About a minute ago   /bin/sh -c #(nop) CMD ["/bin/bash"]              0 B
0ff937a46f57   About a minute ago   /bin/sh -c touch /tmp/test_file                  0 B
ea7651d29bce   About a minute ago   /bin/sh -c #(nop) LABEL maintainer=darkra…       0 B
6a2f32de169d   6 days ago /bin/sh   -c #(nop) CMD ["/bin/bash"]                      0 B
<missing>      6 days ago /bin/sh   -c mkdir -p /run/systemd && echo ‘…              7 B
<missing>      6 days ago /bin/sh   -c sed -i ‘s/^#\s*\(deb.*universe\…              2.76 kB
<missing>      6 days ago /bin/sh   -c rm -rf /var/lib/apt/lists/*                   0 B
<missing>      6 days ago /bin/sh   -c set -xe && echo ‘#!/bin/sh’ >…                745 B
<missing>      6 days ago /bin/sh   -c #(nop) ADD file:b8a2c16d65e533a…              117 MB
```
history는 밑에서부터 위로 최신입니다. 우선 missing 부분은 base image의 history입니다. 우리가 주목해야 할 것은 image에 hash 값이 있는 4개의 자료입니다.
image hash 값을 보면 우리가 Dockerfile로부터 만들어진 image hash 값과 같음을 알 수 있죠.
더 확실하게 확인하는 방법은 Dockerfile에서 build할 때 일부러 에러를 내보면 됩니다. 
위의 Dockerfile을 아래와 같이 변경한 후 build 해보면 확실히 할 수 있습니다.
```
FROM ubuntu
LABEL maintainer "darkrasid@gmail.com"
RUN tasdouch /tmp/test_file
CMD ["/bin/bash"]
```
아래와 같은 error log가 나올 겁니다. 
```
Sending build context to Docker daemon 2.048 kB
Step 1/4 : FROM ubuntu
 — -> 6a2f32de169d
Step 2/4 : LABEL maintainer "darkrasid@gmail.com"
 — -> Running in fc5cfbea326c
 — -> 19928d912115
Removing intermediate container fc5cfbea326c
Step 3/4 : RUN tasdouch /tmp/test_file
 — -> Running in a547a9e9c97c
/bin/sh: 1: tasdouch: not found
The command ‘/bin/sh -c tasdouch /tmp/test_file’ returned a non-zero code: 127
```
log을 잘 보면 3번째 step에서 container가 런은 됐는데 삭제했다는 말이 없네요. 
`docker container ls -a` 나 `docker ps -a`를 한번 해보면
```
CONTAINER ID IMAGE        COMMAND              CREATED       STATUS                       PORTS   NAMES
a547a9e9c97c 19928d912115 "/bin/sh -c ‘tasdo…" 4 minutes ago Exited (127) 4 minutes ago           vigilant_goldberg
```
image를 만들기 위해 만들었던 container가 삭제되지 않고 남아있네요. 이번 log를 다시 한번 보면 6a2f32de169d, 19928d912115 두개의 이미지가 만들어진 것을 확인할 수 있네요.
`docker image ls`나 `docker images` 로 image를 확인해보면
```
REPOSITORY          TAG        IMAGE ID         CREATED             SIZE
<none>              <none>     19928d912115     8 minutes ago       117 MB
<none>              <none>     efea57aae7a5     8 minutes ago       117 MB
how_to_make_image_1 latest     a0e940c0f001     About an hour ago   117 MB
ubuntu              latest     6a2f32de169d     6 days ago          117 MB
```
아까 만든 how_to_make_image_1이 있고 이상한 none image 2개가 있네요. hash 값을 보니 image를 만들다만 찌꺼기네요. 즉 중간에 생성되던 image는 image가 최종적으로 생성될 때 삭제됩니다.
위에는 일부러 에러를 냈지만 사실 Dockerfile을 만드는 과정은 신성한 노가다와 빡침의 현장입니다. 수많은 none image가 생성이 되죠. 생성하다보면.. 그러다 드라마틱하게 뙇 이미지를 생성하면 none image가 삭제가 됩니다. 
하지만 그러지 못했다면? 하다 ` — no-cache` 옵션이라도 썼다면? 저거 다 찌꺼기로 내 로컬에 남습니다. 이번엔 삭제하는 방법에 대해 알려드리겠습니다.
우선 찾아내는 방법 먼저
```
docker image ls — filter=dangling=true # docker images — filter=dangling=true
```
```
REPOSITORY TAG    IMAGE ID     CREATED        SIZE
<none>     <none> 19928d912115 38 minutes ago 117 MB
<none>     <none> efea57aae7a5 39 minutes ago 117 MB
```
이런 결과가 나옵니다. 이걸 삭제하려면
```
docker image rm $(docker image ls — filter=dangling=true -q) # -q 옵션은 hash_id만 빼오는 옵션입니다.
```
이러면 쓸모 없어진 image만 삭제가 됩니다.
최근에 나온 prune을 쓰셔도 되는데 이건 실수하면 container로 run되지 않은 모든 image를 지우기 때문에 좀 주의하셔야 합니다. 그것만 주의하시면 훨씬 간단하죠. 
사실 크게 주의할 것도 없습니다. ` — all, -a` 옵션만 ‘안’붙이시면 되요.
```
docker image prune
```
prune은 버전별로 낮으면 안될 수도 있습니다.
## 결론
서두에서 말씀드렸듯 docker에서 image를 만드는 방법은 사실 commit 하나라는 것을 확인할 수 있습니다. 사실 그 자체가 큰 의미가 있는 것은 아니고 이 과정을 알고 있으면 왜 volume 같은 instruction을 다룰 때 조심해야 하는지 이해하기가 쉽습니다. 여기까지 보시고 이런 생각을 하실 수도 있을 것 같습니다. 아 그럼 그냥 docker commit으로 image다 생성해야겠다. Dockerfile 따위… 절대 아닙니다. 이 포스트는 dockerfile이 image를 생성하는 과정을 설명하는 것이지 Dockerfile이 쓸모없음을 주장하는 포스트는 아닙니다. 반대로 개인적으로 Dockerfile은 꼭 필요하다고 생각합니다. 그래야 협업할 때 서로 어떤 이미지 위에서 작업하는지 혹은 뭐가 잘못됐는지 추적이 가능합니다. 왜 Dockerfile이 꼭 필요한지는 별도의 포스팅으로 다루겠습니다.
