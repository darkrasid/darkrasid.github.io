---
layout: post
title:  "도커와 vm"
author: "Pilsner"
date:  2017-04-12 10:18:00 +0900
categories: docker container vm
---

Docker를 공부하면 가장 처음 등장하는 내용이 바로 `docker vs. vm`입니다. 사실 둘의 비교는 정확히는 `container vs. vm `이긴 한데, 이 포스트에서는 docker container와 vm은 서로 어떻게 다른지 장점 단점에 대해 아주 ‘자세하지 않게’ 다룹니다.

vm과 container의 격리 레벨에 대해 조금 생각해봅시다. vm이 container보다 훨씬 더 강력하게 격리됩니다. vm은 가상화된 하드웨어 위에 os가 올라가는 형태로 거의 완벽하게 host와 분리된다고 봐도 무방하죠. 반면 container는 os 가상화입니다. os 부분을 가상화해서 올리고 커널을 host와 공유합니다. vm보다 얕게 격리되죠. 이 차이에서 발생하는 장단점 이슈들을 몇 개 살펴보겠습니다.

### 작다.
우선 vm 보다 작습니다. 아래 그림을 보면
![container-vm](https://github.com/darkrasid/darkrasid.github.io/blob/master/_image/dockervsvm1.png?raw=true)          

일단 그림의 볼륨만으로도 container가 작죠. 왜 작은가 하니 vm은 hypervisor가 hardware 가상화를 합니다. 그리고 그 위에 Guest OS가 올라가게 되죠. 위의 그림만 해도 3개의 GuestOS가 올라갑니다. 반면 container는 Docker-engine 위에 application에 실행에 필요한 바이너리만 올라갑니다. 그 외 커널 부분은 호스트의 커널을 공유하죠. 격리는 linux namespace를 통해 개념적으로 수행합니다. 만약 호스트의 커널과 container 의 커널이 다르다면? 다른 부분만큼만 패키징 됩니다. 그만큼 또 공간을 절약할 수 있죠. 근데 작은 것이 왜 장점일까요? 요즘 서버 용량 어마어마하잖아요?

만들어진 이미지가 작다는 것 자체가 장점이기도 하겠지만 network를 덜 사용할 수 있다는 장점도 있습니다. docker 의 라이프 사이클은 우선 hub(혹은 private Registry)로부터 image를 pull을 받아오면서 시작하죠. 여기서부터 용량이 작다는 것은 큰 장점이 됩니다. 그만큼 네트워크를 덜 쓸 수 있고 (pull 받는데 시간이 적게 걸리는 건 덤이죠) cloud는 사용량만큼 과금하죠. 돈과도 직접적으로 연결이 됩니다. 게다가 host가 하나가 아니라 clustering 환경이었다면? 용량이 큰 vm으로 배포한다고 생각하면 답답해지죠.
### 빠르다.
우선 vm이나 container나 (당연히) host 머신보다는 느립니다. 하지만 container와 vm을 비교하자면 container가 더 빠릅니다. 몇까지 원인이 있겠지만 단순하게 비교를 해보자면 vm은 io가 발생하는 통로가 container보다 많습니다. vm은 host os에서 거의 완전히 분리된 형태로 운영이 되지만 io가 오갈 때는 이야기가 좀 다릅니다. vm이 처리한 io를 결국 host os의 커널이 다시 받아 자신의 드라이버에 맞게 처리해줘야 하죠. 이 부분에서 병목이 발생하게 됩니다. 반면 container는 커널을 공유하기 때문에 들어온 io가 쉽게 처리돼서 나갈 수 있게 되죠. 즉 쉽게 표현하면 vm이 container보다 더 많은 커널 처리가 들어가야 되니 당연히 성능이 좀 더 느립니다.        

근데 요즘 vm벤더들은 이 부분을 의식해서 계속해서 성능경쟁을 하고 있습니다. 몇몇 자료에서는 vm이 container보다 빠르다는 증거를 내기도 하더군요. 하지만 기본적인 개념으론 container가 더 빠릅니다.
### 라이프 사이클?
개인적으로 container 기술이 vm보다 우위에 서는 가장 큰 이유라고 생각하는 것이 바로 이 부분입니다. 사실 이 파트는 docker의 특징이기도 합니다. docker의 뛰어난 라이프 사이클을 거의 바로 적용할 수 있는 것이죠. 사실 container 기술은 계속해서 꽤 오랜 시간 동안 존재해왔습니다. docker는 사실 많은 container 기술 중 하나일 뿐이죠. 근데 그동안 container가 그리 뜨지 못하고 최근 들어서 이렇게 온 사방에서 docker, docker 하고 있는 것일까요?      

버전업을 생각해볼까요? 원래대로라면 어떤 application의 소스를 고쳐 배포를 하게 되면? vm에 접근해서 repository를 pull 받겠죠. 혹은 사이트마다 어쩌면 원래 소스를 지우고 clone을 새로 받을 수도 있을 겁니다. 거기에 변경된 config를 변경하고.. 오 손이 많이 가네요. 근데 서버가 하나가 아니면요? 30대면 이걸 어떻게 하죠? 뭔가 해결책을 마련하겠죠. capistrano? 아니면 jenkins를 구성할 수도 있을 겁니다. 근데 그 스크립트에서 에러 나면? 문제가 복잡해지죠. debugging도 힘듭니다.        

하지만 docker 환경이라면 내 개발 환경에서 운영 환경 image를 만들어 registry에 배포하고 원래 돌던 거 내리고 새 거 올리고 끝. 같은 docker engine 위에서 이미 테스트했기 때문에 운영환경이라도 에러가 날 가능성이 매우 낮죠. 똑같은 jenkins를 구성해도 이미 스크립트의 양에서 어마어마하게 차이가 날 것입니다. 그만큼 배포의 위험성도 줄어들게 됩니다.       

거기에 한술 더 떠 요즘은 오케스트레이션까지 제공됩니다. 여러 대의 host에 여러 대의 container를 동시에 배포하면서 container를 관리할 수도 있죠. scale out도 쉽고 또한 병목 현상이 생긴 container를 찾아낸다면 전체 infra의 scale up이 아니라, 그 현상이 이뤄나는 container의 scale out을 통해 전체 서비스의 성능을 올릴 수 있는 유연성도 생깁니다. 위의 배포 예를 생각해보면 쉽게 버전업을 하는 것뿐만 아니라 롤링 업데이트를 위해 HA까지 쉽게 구성할 수 있을 것입니다.      

네, 맞습니다. docker를 사용함으로써 vm에 비해 가지는 가장 큰 장점은 바로 microservices로 발전할 수 있는 토대를 마련하고 적용해나갈 수 있다는 것입니다. 위의 작업을 vm으로 한다면요? 그 수많은 bandwidth와 scale out 등을 어떻게 처리하죠? 뭔가 방법은 있겠지만 아마 비교도 할 수 없이 복잡하고 많은 지식을 동원해야 할 것입니다.
## vm의 장점
그럼 과연 모든 환경에서 container가 완벽한 대안일까요?
### 보안
위의 내용을 보면 container가 host의 커널을 공유한다는 개념이 여러 장점을 만들고 있는 것을 알 수 있습니다. 근데 이거 장점만 있는 것일까요? container 가 host를 공유한다는 것은 container 하나가 뚫리면 바로 host 커널이 위험해진다는 것을 뜻하기도 합니다. 동시에 호스트 커널까지 오지 않더라도 커널이 공격당하면, 커널을 공유한다는 개념 때문에 모든 container가 위험해지는 현상이 발생할 수 있습니다. 이건 완벽하지 않은 가상화에 따른 양날의 검이라고 볼 수 있습니다. 보안에 취약해진다는 것이죠. 반면 vm은 하나의 vm이 공격당해도 거의 완벽하게 다른 vm이나 host가 보호됩니다. 공유되는 것이 없으니까요. 아키텍처적으로 보안엔 vm이 더 안전합니다.
### 멀티 os
커널을 공유한다는 내용 때문에 container가 못하는 것이 또 있습니다. 호스트 os와 전혀 다른 os를 container로 올릴 수는 없습니다. 무슨 말인고 하니 linux 머신에서 window 서버를 container로 올릴 수는 없다는 뜻입니다. 반면 vm은 하드웨어 가상화를 통해 완벽히 가상화된 os를 올릴 수 있기 때문에 os 선택의 자유로움이 있을 수 있습니다.
## vm 위의 container?
![moni-os](https://github.com/darkrasid/darkrasid.github.io/blob/master/_image/dockervsvm2.png?raw=true)          

마지막 이야기는 위의 장점과 단점을 버무려서 좀 더 좋은 환경에서 container를 사용하기 위한 방법입니다. 보안적으로 조금 더 안전한 vm 위에 container로 서비스하는 방법이죠. 아마 이 글을 읽자마자 몇 분은 ‘아니 이게 무슨? 이건 그냥 단점을 하나로 합치는 방법 아냐?’라고 생각하실 수도 있을 것입니다. 꼭 그렇지는 않습니다.        

기본 흐름은 이렇습니다. 베어메탈 위에 host os를 올리고 그 위엔 여러 개의 vm을 올린 후에 그 위에 docker-engine을 올리는 거죠. 원래 2중첩의 그림에서 기본 3중첩의 그림이 되는 것입니다. 아니 그럼 단점으로 얘기했던 느린 건??? 어쩌고? vm에 올라가는 guest os를 경량화 시킴으로서 어느 정도 해결하는 방법입니다. vm의 guest os를 ubuntu나 redhat 등을 사용하는 대신 nano(ms)나 photon(vmware), atomic(fedora) 등의 초경량 os를 사용해서 vm의 단점을 대폭 줄이는 방법입니다. 유명한 coreos도 docker를 지원하니 좋은 대안이 될 수 있겠죠. 어차피 모든 application은 container로 구동할 것이니 불필요한 모든 것을 버린 초경량 os를 채용할 수 있는 것입니다.        

여러 대의 vm이 각각 여러 개의 container를 굴리는 식으로 운영이 되니 하나의 container가 공격받아도 그 container가 속한 vm만 위험해지고 나머지는 안전하죠. 동시에 요즘은 오케스트레이션 레벨에서 스케줄링과 로그밸런싱까지 지원하니 서비스엔 더더욱 문제가 없고요.
## 결론
(docker) container와 vm 과의 차이에 대해 조금 알아봤습니다. 제 개인적인 생각엔 어지간한 상황에선 container를 적용하는 것이 꽤 좋은 선택이 될 수 있다고 생각합니다. 물론 배포가 거의 없고 보안이 엄청 중요한 프로젝트 같은 특이한 환경이 아니라면요. 도움이 되셨길 바랍니다.
