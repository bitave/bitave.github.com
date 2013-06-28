---
layout: post
title: "키위마켓 테크 스택"
description: ""
category: 
tags: []
---
{% include JB/setup %}

![초기 종이 목업](/assets/images/paper_mockup.jpg "초기 종이 목업")


키위마켓의 초기 종이 목업
<br><br>

[키위마켓](http://kiwimarket.co.kr)은 처음에 종이로 만들어진 목업으로 시작했습니다. 종이에 그림을 그려서 가지고 다니면서 고등학생에서 주부분들까지 각계각층의 분들과 인터뷰를 했고 괜찮을 것 같다는 반응을 얻어 개발에 착수하기 시작했습니다. 

상시 개발자가 2명에 불과했고 팀에 안드로이드 개발 경험도 거의 없는 상황이었지만 최대한 빨리 제품을 만들어 사용자로부터 가설검증과 제품 개선을 해야한다는 목표가 있었습니다. 1월부터 개발을 시작해 두달안에 개발을 완료한다는 목표아래 최대한 Quick & Dirty로 빠르게 만드는 것에 중점을 두었습니다. 

이에 서버는 기존에 최대한 익숙한 구조인 LAMP환경에서 Cake PHP로 일주일 이내에 구현을 하기로 하였고 별도로 인증 및 보안등의 구현은 건너뛴 째 클라이언트 개발에 중점을 맞추고 개발을 하였습니다. 디자인이나 기능도 최소한의 기능 테스트를 위한 정도로만 설계 했고요.
<br><br>
![초기 클베 버전](/assets/images/kiwi_old.png "초기 클베 버전")
![현재의 키위마켓](/assets/images/current.png "현재의 키위 마켓")


키위마켓 클베 초기 버전(좌), 현재의 키위마켓(우)
<br><br>
초기버전에서 어느정도 가능성을 확인한 뒤 유저들의 피드백을 바탕으로 계속해서 업그레이드 된 버전을 개발하였습니다. 유저들의 피드백에 따른 디자인의 개선이 있었고 사진의 품질 개선이나 카테고리 기능 추가등을 하게 되었습니다. 그리고 어느정도 제품의 컨셉에 대한 확신을 가진 뒤 대대적인 코드 리팩토링 및 Stack 정리 작업을 하게 되었습니다. 

## 서버 Stack

- Django
- Tastypie
- PostgreSQL
- Redis
- Celery + RabbitMQ
- Nginx + Uwsgi
- Fabric + Boto.

### Django

처음에 Cake PHP로 클베 서버를 구현한 뒤 본격적으로 production 환경에 구현될 언어와 framework을 선택하는 고민을 하였고 결론적으로 Python Django를 선택하였습니다. 아직은 중고거래라고 하지만 결국은 커머스라는 서비스의 특성상 admin 페이지가 중요하다고  생각했고 이에 대한 기능을 가장 풍부하게 제공하는 framework으로 Django를 선택하게 되었습니다. admin 페이지 외에도 문서화가 잘 되어있다는 점이나(Django에 관한 문제는 결국 찾다보면 모든 답이 django 매뉴얼에 있더라는...) 다양한 오픈소스 라이브러리가 제공된다는 점이 좋았습니다. 예를 들어 문자인증을 위해 otp를 구현해야 된다고 하더라도 이에 대해 googling 해보면 바로 관련 라이브러리가 검색되어서 나온다든지요. 디버깅이나 프로파일링이나 배포에 있어서도 툴도 잘 갖추어져 있어 수월하게 진행할 수 있었습니다. 

### Tastypie

안드로이드 클라이언트와의 통신은 RESTful API를 만들어서 하기로 했고 이에 대한 Django framework로 Tastypie를 선택하였습니다. 문서가 깔끔하고, 실제 tutorial을 따라서 만들어 봤을 때  10분만에 스키마까지 정리되어 깔끔하게 구현되는 REST API가 인상적이었습니다. 하지만 실제 개발에 들어가서는 Tastypie에서 제공하는 거의 모든 method들을 override 해서 우리 서비스에 맞게 바꾸어 주어야 했습니다. 결과적으로 대부분의 코드를 다시 작성한 것 처럼 되어 버렸는데 그래도 전체적인 REST구조를 잡고 인증 부분의 처리를 간편하게 처리할 수 있었던 장점이 있던 것 같습니다. 부가적으로 후속 개발자에게 레거시 코드를 넘길 때 따로 문서 작업을 덜고 Tastypie의 문서와 함께 넘기면 좀 더 쉽게 레거시 코드를 이해할 수 있을 것 같다는 장점이 있는 것 같습니다. 

### PostgreSQL

초반에는 NoSQL도 고민하였으나 서비스의 특성에 맞지 않는다는 판단하에 RDBMS 중에서 선택을 하기로 하였고 MySQL로 개발을 시작하였었습니다. 하지만 개발이 진행될 수록 MySQL의 Trigger나 Stored Procedure등의 기능이 부족하여 Postgre SQL로 갈아탔고, 덕분에 DB에서 많은 로직을 처리해주어 어플리케이션 로직이 단순해지는 효과를 보았습니다. 속도 측면에 있어서도 벤치마크 결과에 따라 들쑥날쑥한 부분이 있지만 버전 9 이후에 Postgre SQL의 속도에 대대적인 성능향상이 있었고 실제 서비스에 적용하는 사람들의 체감속도 면에서 MySQL보다 Postgre SQL이 더 빠르다는 평이 나오고 있습니다. 과거에 서비스를 시작한 스타트업들은 MySQL을 많이 사용하는 반면 최근에 만들기 시작한 서비스들은 PostgreSQL을 많이 사용하고 있고 북미권에서는 특히 많이 사용되고 있고요. 기능도 많고 인덱스도 다양하게 지원하고(예를 들어 R-Tree같은 spatial index라던지), 아직 한글 지원은 부족하지만 full text search를 지원한다는 점도 장점일 것 같네요. Connection Pool로 PgBouncer와 같이 사용하고 있고, 속도나 기능 측면에서 모두 MySQL을 사용할 때 보다 만족하고 쓰고 있습니다. 

### Redis

유저 관련 세션이나 자주 사용되는 내용의 경우 Redis로 메모리에 캐시해서 사용하고 있습니다. Memcached에 비해 성능이 떨어지는 부분이 없고 기능이 다양하게 지원되어 Redis를 캐시로 사용하고 있습니다.

### Celery + RabbitMQ

주기적으로 푸시를 대량으로 보내야 한다던가 웹서버에서 처리하기에는 시간이 많이 걸리는 작업들은 Celery를 이용해서 비동기로 분산하여 작업하고 있습니다. Celery의 task queue는 RabbitMQ를 사용하였고 Django Celery 모듈을 사용하여 쉽게 적용할 수 있습니다. 개발을 하다가 조금 오래 걸릴 것 같다 하는 작업들은 task로 만들어 쉽게 worker로 넘겨서 처리할 수 있기 때문에 향후에도 편리하게 이용할 것 같습니다. 

### Nginx + Uwsgi

리버스 프록시는 Nginx, 컨테이너는 Uwsgi를 사용하고 있습니다. 셋팅도 쉽고 가볍고 안정적인 것 같습니다.

### Fabric + Boto

각 서버에 대한 어플리케이션 배포는 Fabric을 이용하여 하고 있습니다. 여러대의 서버를 사용하고 있고 AWS에서 제공하는 Auto-Scaling을 사용하고 있는데 서버가 계속 유동적으로 바뀌므로 Boto를 이용해서 서버들을 관리하여 Fabric으로 배포하는 방식을 사용하고 있습니다. 


## 클라이언트 Stack

- Actionbar Sherlock
- Universal Image Loader
- Android Asynchronous Http Client
- Jackson
- Cruton
- Pull to Refresh
- Flurry
- Postman

키위마켓의 안드로이드 클라이언트 역시 수많은 오픈소스를 사용해서 만들어졌습니다. 

### Actionbar Sherlock
액션바를 이용한 UX구성은 안드로이드앱의 표준이 되가고 있습니다. 하지만 아쉽게도 안드로이드3.0(API Level11)부터 액션바를 사용할 수 있어 2.x를 지원하는 안드로이드앱에서는 사용할 수 없습니다. 
이 문제점을 해결하기 위해서 안드로이드 2.x에서 액션바를 사용할 수 있도록 Jake Wharton이 [Actionbar Sherlock](https://github.com/JakeWharton/ActionBarSherlock)를 개발하여 오픈소스로 배포하고 있습니다. 이 외에도 [Jake Wharton](https://github.com/JakeWharton/)의 리파지터리에 가 보면 안드로이드 프로젝트에 도움이 되는 다양한 오픈소스 프로젝트들이 있으니 참고하시면 좋을 것 같습니다.

### Universal Image Loader
[Universal Image Loader](https://github.com/nostra13/Android-Universal-Image-Loader)는 비동기적으로 이미지를 불러오고 이미지를 캐싱하는 등의 작업을 해주는 라이브러리입니다. 사용하기가 편리하다는 장점이 있지만 1.8.x 버전임에도 버그 이슈가 많이 리포트 되고 있어
차기 버전에서는 Square의 [Picasso](https://github.com/square/picasso) 사용을 고려하고 있습니다. 

### Jackson
여러 json 라이브러리 중에서 가장 빠르다고 하는 [jackson](http://jackson.codehaus.org/) 라이브러리를 사용하였습니다.

### Android Asynchronous Http Client
loopJ의 안드로이드 비동기 [HttpClient](http://loopj.com/android-async-http/)는 비동기로 웹서비스에 요청을 보낼 수 있는 오픈소스입니다. UI와 사용자 인터렉션을 담당하는 메인 스레드가 응답이 올 때까지 기다리지 않게 하기 위해서 비동기로 요청을 보내고 응답을 받는 일은 GUI 프로그램에서 중요한 요소입니다.
요청 메소드를 쉽게 지정할 수 있는 점과 헤더 필드의 설정 그리고 파일 업로드와 다운로드 등이 매우 쉽고 요청을 보내기 위한 프로세스가 start-success or failed-finish 로 단일화 되어 있어 로직을 구성하기가 매무 편했습니다. 또한 오랜 기간동안 유지되어온 프로젝트라서 마음에 들었고 인스타그램과 핀터레스트에도 사용되고 있습니다.

### Cruton
[Crouton](https://github.com/keyboardsurfer/Crouton)은 카테고리 검색 결과에 쓰인 라이브러리로 원하는 뷰그룹에 슬라이드 인/아웃 알림뷰를 생성해 주는 오픈소스입니다. 커스텀뷰를 지원하여 물건올리기를 할 때 진행상태를 보여줄 때도 사용할 수 있었습니다.

### Pull to Refresh
Pull to Refresh UX는 모바일앱의 more와 refresh를 처리하는 표준 UX로 받아들여지고 있습니다. 안드로이드에서 리스트뷰, 그리드뷰 그리고 스크롤뷰 등의 Pull to Refresh를 지원하는 오픈소스는 Chrisbane의 [Pull to Refresh](https://github.com/chrisbanes/Android-PullToRefresh) 라이브러리가 있습니다. 트위터와 같은 Pull to Refresh를 구현할 수 있으며 subclassing해서 쉽게 customize도 할 수 있습니다.  현재 프로젝트가 멈춘 상태이지만 적용하는데 문제가 없고 사용하기도 편했습니다. 
반가운 소식은 액션바에 잘 어울리는 새로운 Pull to Refresh를 Chrisbane이 [ActionBar-PullToRefresh](https://github.com/chrisbanes/ActionBar-PullToRefresh) 프로젝트로 진행하고 있습니다. 참고하시면 좋을 것 같습니다.

### EventBus
안드로이드의 컴포넌트간의 의존성을 줄이기 위해서 현재 버전에서는 Local Broadcast 를 사용하고 있습니다. 하지만 코드가 지저분해지는 것은 별 차이가 없어 고민하던 중에 [EventBus](https://github.com/greenrobot/EventBus)를 알게 되었습니다. EventBus를 사용하면 컴포넌트간의 의존성을
줄이면서도 코드를 아주 깔끔하게 작성할 수 있어 차후 업데이트 때 적용할 예정입니다. 

### Flurry
키위마켓의 데이터 분석을 위한 로그는 Flurry를 통하여 남기고 있습니다. 

### Postman
부가적으로 Chrome 웹 플러그인으로 [Postman](https://www.google.co.kr/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&ved=0CD8QFjAC&url=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fpostman-rest-client%2Ffdmmgilgnpjigdojojpjoooidkmcomcm&ei=XPPMUa6qNYaAiQeYkICQBw&usg=AFQjCNFL71vN61QG0LKlw7VDJvIZDprjHA&bvm=bv.48572450,d.aGc)이라는 녀석이 있는데 REST Client 테스트 하는데 유용하게 쓰였습니다. 

## 프로젝트 관리

 - Git
 - Trello

### Git

말이 필요없죠. 키위마켓의 모든 소스코드는 Github에서 Git으로 관리하고 있습니다. 이 블로그 역시 협업을 통해 블로깅을 하기 위해 Git으로 관리하고 있고요. 브랜치를 쉽게 만들 수 있다는 점이 여타 버전 관리 시스템 대비 좋은 것 같습니다. Github가 있어서 따로 저장소 서버를 만들지 않아도 되고요. 

### Trello

키위 마켓의 전체 프로젝트는 Trello를 통하여 이루어지고 있습니다. 개발 뿐만아니라 기획, 디자인, 마케팅 협업 툴로 트렐로를 사용하고 있고요. 개발 변경 사항이 있으면 Trello에 모아서 올린 다음 난이도와 필요성에 따라 우선순위를 선정하여 개발을 하고 있습니다. 각자 어느정도의 일을 얼마만큼 하고 있는지 파악할 수 있어 협업할 때 좋은 것 같고, 개인적으로도 다음에 무슨 일을 해야할지 관리할 수 있어서 좋은 것 같습니다. 사무실에 오자마자 무슨일을 해야할지 망설여야 하는 시간이 없는 것 같아요. 
<br><br>

### 키위마켓을 함께 만드실 분을 찾습니다.
<br>

아직 작은팀이지만 그만큼 기회도 많고 이제 시작단계라 할 수 있는 일이 많다라고 생각합니다. 저희와 함께 고민하며 키위마켓을 만들어 나가실 분을 찾습니다. 언제든지 관심 있으신 분은 주저말고 메일 부탁드립니다. ([developer@bitave.com](mailto:developer@bitave.com)) 사무실로 오심 직접 얘기나누면서 커피와 빵도 사드려요~ [구인공고 바로가기](/2013/06/27/job/)


