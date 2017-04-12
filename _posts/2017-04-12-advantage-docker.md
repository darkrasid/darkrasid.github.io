---
layout: post
title:  "docker container"
author: "Pilsner"
date:  2017-04-12 10:18:00 +0900
categories: docker container 
---


이번에 [Docker captain](https://www.docker.com/community/docker-captains)중 한명인 John Zaccone이 Docker를 쓸 때의 5가지 이점에 대한 포스팅을 올렸더군요.. 그래서 못쓰는글 창작하느니 잘쓰여진 글 한번 번역해봤습니다.. 솔직히 크게 새로운 내용이 있는 post는 아니긴 한데 재미로 보기에 좋은 것 같습니다. 

겁나 많은 오역, 의역이 있습니다. 영어 울렁증이 없으신 분은 그냥 원문으로 보시는걸 추천합니다 ㅋㅋ 
[원문보러가기](http://www.johnzaccone.io/top-5-benefits-of-docker/) 
원 링크에는 세미나 영상도 올라와 있습니다. 

# 이점 1 : "제 로컬에선 잘 되는데요" 신드롬을 풀어라.
운영에선 재현되면서 로컬에선 재현안되서 곤란했던 이슈가 있었나요? 그건 보통 당신의 로컬환경과 운영환경의 차이 때문에 발생합니다. 운영환경에서의 추가적인 변경은 (예를 들어 security patch를 적용한다던가 하는) 직접적인 환경의 차이를 만들어 냅니다. 해결책? immutable infrastructure를 사용하세요. 그러면 환경을 직접적으로 변경하는 것을 피할 수 있습니다. Docker image는 이런 환경을 제공해줍니다. 

# 이점 2 : 보안
이런 질문이 한동안 나온적이 있습니다. "도커 컨테이너가 안전한가요?" [여기 답이 있습니다.](https://blog.docker.com/2016/08/software-security-docker-containers/) Gartner 나 NCC Group 의 전문가들은 컨테이너가 보안적으로 안전할 뿐만 아니라 컨테이너를 사용하지 않는 환경과 비교해서도 이점을 얻을 수 있다고 말합니다. 컨테이너와 호스트 사이의 고립적인 환경을 이용해 또 다른 defense layer를 쌓을 수 있다는 것이죠. 뿐만 아니라 application에서 install된 라이브러리와 리눅스의 필수적인 접근 제어를 whitelist로 관리할 수도 있습니다. 

추가적으로 [Docker Security Scanning](https://docs.docker.com/docker-cloud/builds/image-scan/) 서비스를 당신의 CI/CD pipeline에 적용함으로써, 보안 취약점을 알아내기 위해 docker images를 scan할 수 있는 옵션도 선택할 수 있습니다. 이 서비스를 이용하면 [Common Vulnerabilities and Exposures](https://cve.mitre.org/)(CVE®) database 룰에 따라서 보안취약점을 listing해서 유저에서 알려줍니다. database가 새로 업데이트가 되면 이 서비스는 image를 소급적으로 다시 스캔하고 취약점이 발견되면 운영자에게 알려줍니다. Security scanning service와 docker [content trust](https://docs.docker.com/engine/security/trust/content_trust/)를 결합하여 이미지의 게시자를 확인할 수도 있습니다. 

# 이점 3 : Microservices 시장으로의 빠른 진출
우린 모두 monolith를 싫어합니다. : 꼬여 있는 스파게티 코드는 모든 것을 멈추지 않고선 아무것도 바꿀 수 없는 환경을 만들어 우리가 비지니스적인 가치를 창출하는것을 방해합니다. Microservices는 독립적으로 배포될 수 있는 loosely-coupled services를 개발함으로써 이런 사태를 방지할 수 있도록 해줍니다. 물론 모든 architecture는 trade-offs가 있습니다. 그리고 microservices의 trade-offs는 바로 도전이죠. 예를 들자면 이런 것들이요..  service orchestration, centralized monitoring and logging, 주문형 환경...

다행스럽게도, Docker는 이런 이슈들을 도와줄 수 있습니다. Docker container는 표준적인 인터페이스를 제공하여 operations-type problems를 훨씬 쉽게 해결되도록 하면서 ecosystem tools를 그 위에 구축가능하게 합니다. 도커 컨테이너의 빠른 구동 시간은(원문에서 spin-up time 인데 맞는 해석인지 모르겠네요..) multiple environments를 빠르게, 또 격리된 상태(ci/cd pipeline에서 필요한 환경을 포함하여, 예를들면 build나 test 환경같은..)로 구축할 수 있게 합니다. 

# 이점 4 : Unlock the Ecosystem
Docker에 대해 제일 좋다고 생각하는 것 중 하나는 Docker technology를 둘러싸고 있는 커뮤니티와  ecosystem 입니다. 컨테이너 management 툴이 필요한가요? [ECS on Amazon](https://aws.amazon.com/ko/ecs/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=ecs_b&sc_content=ecs_general_bmm&sc_detail=%2Bamazon%20%2Becs&sc_category=ecs&sc_segment=145512007429&sc_matchtype=b&sc_country=US&s_kwcid=AL!4422!3!145512007429!b!!g!!%2Bamazon%20%2Becs&ef_id=VpUrmAAABaDM4QNM:20161206184559:s)를 한번 사용해보세요. CI/CD 플러그인? 젠킨스가 Docker와 함께 일할 수 있게 몇몇 플러그인을 제공합니다. 서비스 orchestration? [Kubernates](http://kubernetes.io/)가 하나의 답이 될 수 있습니다. (사실 전 1.12에서부터 built in된 docker swarm mode를 추천합니다). 이런 tool들을 활용하여 쓸떼없는 문제를 피하고 당신의 고객에게 더 많은 가치를 전달하기 위해 투자하세요. 

# 이점 5 : Open된 공간에서의 개발
기술은 떠오르기도 하고 사라지기도 합니다. 회사는 오랜된 기술이 쓸모 없어지거나 새로운 기술을 적용해야 할 때 엄청난 migration을 수행해야 합니다. 이런 주기 때문에 어떤 기술을 적용할 지 고려하는 것 뿐만 아니라 그 기술이 향하고 있는 방향을 파악하는 것이 매우 중요합니다.

Docker는 Docker 프로젝트의 방향을 결정하는 데 도움이되는 커뮤니티를 완전히 수용해서 이런 싸이클에서 보호합니다. 게다가 Docker engine자체로 오픈소스이면서 infrakit, datakit, hyperkit같은 것들을 추출하여 분리된 오픈소스 project로 만들었습니다. 이런 프로젝트들은 docker에 의존적이지 않습니다. 동시에 커뮤니티는 이러한 구성 요소에 영향을 미치고 빌드 할 수 있습니다. Docker는 또한 커뮤니티에 의해 정의된 공개 표준 위에서 만들어졌습니다. [Open Container Initiative](https://www.opencontainers.org/)(OCI)는 컨테이너 포맷과 실행환경을 둘러싼 공개 표준(Open Standard)을 정의하기 위해 만들어졌으며 Google이나 Redhat 같은 수많은 리더들에 의해 뒷받침 되고 있기도 합니다. Docker는 OCI 사양([runc](https://runc.io/))의 첫 번째 구현에 기여했고 Docker engine은 1.11버전부터 OCI를 준수하고 있습니다.

