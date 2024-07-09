---
title:  "미니 프로젝트 - 백엔드 구조"
excerpt: "미니 프로젝트"

categories:
  - miniProject
tags:
  - Architecture
last_modified_at: 2022-11-14T08:06:00+09:00
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
---

![image](/images/2022/miniproject/beautiful_anime.gif){: .align-center}

이번 포스트에선 미니 프로젝트 시간에 사용했던 백엔드 환경에 대해 설명하고자합니다.

미니 프로젝트는 **영화 웹사이트 구축**이였고

필자는 프로젝트의 **백엔드**를 담당했습니다.

더 자세히는 

* 백엔드 서버를 어떻게 설계했는지,

* 해당 기술 혹은 어플리케이션을 왜 선택했는지,

* 또한 문제가 발생하면 해결하기 위해 어떤 고민을 했는지 설명하겠습니다.

> 백엔드 서버를 `NGINX`로 배포를 하였으나 어디까지나 편법임을 염두하고 읽어주셨으면 합니다.

<br/>

## 초기 구조 (Base Architecture)

![image](/images/2022/miniproject/miniproject-base_architecture.png){: .align-center}

### 첫번째 고민: 환경 구축
---
초기 프로젝트를 시작했을때 강사님이 설정해주신 초기 구조는 위와 같습니다.

프론트는 <span style="color:blue">ReactJS</span>, 백엔드는 <span style="color:orange">Java Servlet</span>, DB는 <span style="color:red">Oracle</span>로 구성되어있습니다.

이런 구조로도 코드는 Git으로 공유하며 개발할 수 있지만

1. DB는 각각 개발자 PC에서 설치해야하고

2. DB 프로그램 설정과 테이블 구조가 <u>실시간으로 동기화되지 않아</u> 충돌이나거나 달라질 수 있는 등

설정으로 인한 스트레스가 발생할 가능성이 매우 높은 구조입니다.



백엔드 담당으로 먼저 해결해야할 첫 우선 과제는 프론트엔드 분들이 개발하실 때 

* **백엔드 환경 관련 요소로 불편한 점을 겪게 하지 않고** 

* 최대한 **동일한 환경에서 개발**할 수 있도록 지원하는 것

을 중점적으로 생각해 설계했습니다.

즉, **<u>서버에 백엔드 관련 서비스를 배포해 언제든지 접속해서 개발할 수 있는 환경구축</u>** 이 당면 과제라 생각했습니다.

<br/>

### 두번째 고민: 어떤 서버를 사용할까? <span style="color:red">AWS?</span> <span style="color:blue">NAS?</span>
---
당연히 AWS가 범용성도 높고 관련 자료도 많으며 현 트렌드에도 맞지만 <span style="font-size:10pt">~~무엇보다 NAS는 별도로 구매해야한다...~~</span>

AWS를 지금 당장 배운 적 없고 우선 배포해서 기능이 잘 작동하는 것이 목적입니다.
<br/>
그래서 개인적으로 구매했던 NAS를 이용하기로 결정했습니다. 

NAS(정확히는 Synology NAS)는 리눅스 환경, SSH 등을 지원하고 애플리케이션을 구축, 배포할 수 있는 소프트웨어 플랫폼인 Docker도 지원합니다.

NAS 환경에서의 배포 경험이 AWS에 배포했을때도 도움이 될 것 같아 추후 프로젝트에는 AWS의 사용법을 공부하고 사용하기로 계획했습니다.

<br/>

### 세번째 고민: 어떤 DB를 사용할까? <span style="color:red">오라클?</span> <span style="color:blue">MariaDB?</span> <span style="color:skyblue">MySQL?</span>
---
학원 과정에서는 오라클 DB를 배웠지만 오픈 소스이고 관련 자료양과 Docker의 배포도 유리하고 유무료 버전에따른 기능 차이도 없는 MySQL 혹은 MariaDB를 사용하기로 선택했습니다.


<br/>

### 네번째 고민: 서버를 배포할 플랫폼으로는 어느 것을 사용할까? <span style="color:blue">Docker?</span> <span style="color:skyblue">쿠버네티스?</span>
---
도커와 쿠버네티스 두 플랫폼 모두 좋지만 
<br/>
아래와 같은 이유 때문에 도커로 선택했습니다.
<details>
<summary>Docker로 선택한 이유</summary>
<div markdown="1">       

* Docker를 사용해본 경험도 있고 
* Spring 설치 지원 가이드도 있으며 
* 검색했을때 관련 자료도 많고
* 이번 프로젝트에선 도커만으로도 충분하다 생각하기 때문에
</div>
</details>


<br/>

### 다섯번째 고민: 영화 자료를 가져 올 방식으로 <span style="color:red">API?</span> <span style="color:blue">크롤링?</span>, <span style="color:orange">사이트</span>는 어디?
---

이번 미니 프로젝트에서 정말 영화 DB에 가깝게 구축하기위해 실 규모 크기의 영화 DB 데이터를 모으려고 계획했습니다.

먼저 크롤링보다 사용하기 편한 API방식을 선택했습니다.<br/>
그 다음 오픈API로 데이터를 제공하는 사이트를 검색했습니다.

한국 영화 관련 오픈API를 제공하는 사이트는 네이버, KMDB 등이 있으나<br/>
KMDB는 일일 API사용량이 1백회인 것에 비해 1천회 정도 되는 네이버 API를 선택했습니다.

크롤링은 API에서 제공해주지 않는 정보를 가져올 때 사용하려 계획했었으나 실제로는 사용하지 않았습니다.


<br/>

### 여섯번째 고민: 백엔드 서버로 <span style="color:red">서블릿</span>을 그대로 사용할 것인가?
---
개인적으로 <span style="color:green">스프링</span>에 관심있어 따로 공부했었고, 스프링의 여러 기능을 사용하고 싶어서 <span style="color:red">서블릿</span> 대신 <span style="color:green">스프링</span>을 사용하기로 결정했습니다.

<span style="color:green">스프링 프레임워크</span>에 <span style="color:green">스프링 부트</span>, <span style="color:green">스프링 MVC</span> 등을 사용할 생각입니다.


<br/>

이렇게 **첫 번째 구조**가 완성되었습니다.


<br/>
<br/>

## 첫번째 구조 (First Architecture)

![image](/images/2022/miniproject/miniproject-first_architecture.png){: .align-center}

**하지만:** 위 사진과 같은 환경으로 새롭게 구성되었지만 여러가지 문제점이 있습니다.
{: .notice--warning}

<br/>

### 첫번째 고민: ReactJS는 아직 배포를 못해 개발자 컴퓨터에서 localhost로만 접근 가능하다
---
해당 배포 기능은 이 구조에서는 염두해두지 않아 우선순위를 후순위로 두었습니다.


<br/>

### 두번째 고민: Spring 서버가 두 가지 역할을 담당하고 있다
---
1. 네이버 사이트에서 OpenAPI로 자료를 요청하면서 설계한 DB에 맞게 조작하고 Commit하는 역할
2. 프론트엔드에 API요청을 처리하는 역할

이렇게 Spring 서버가 두 가지 역할을 하고있어 네이버의 데이터를 처리하는 별도의 API 서버를 두기로 계획했습니다.



<br/>

### 세번째 고민: 네이버 OpenAPI 일일할당량이 낮고 제공되는 정보량이 부족하다
---
[위에서 언급했다싶이](#다섯번째-고민-영화-자료를-가져-올-방식으로-api-크롤링-사이트는-어디) 실 규모 크기의 영화 DB 데이터를 모으려고 계획했습니다.

일단 네이버에서 제공하는 OpenAPI를 사용했으나, 수십만에 해당하는 영화 데이터에 비해서 네이버가 제공하는 API의 일일할당량인 1천건은 많이 낮았습니다.

또한 제공되는 정보도 만족스럽지 않았습니다.


<br/>

그래서 **두 번째 구조**로 변경했습니다.


<br/>
<br/>

## 두번째 구조 (Second Architecture)

![image](/images/2022/miniproject/miniproject-second_architecture.png){: .align-center}

### 첫번째 해결: DB 웹사이트 변경
---
DB사이트 중 제공하는 영화 데이터량이 충분하고 API할당량도 무제한이며 어느정도 한글 데이터도 존재하는 사이트를 검색했습니다.

[TMDB(The Movie DataBase)](https://www.themoviedb.org/)란 사이트를 찾게되었고 다양한 OpenAPI 방식을 지원하며 별도의 할당량도 없이 데이터도 풍부해서 <span style="color:green">네이버</span>에서 <span style="color:blue">TMDB</span>로 변경했습니다.

<br/>

### 두번째 해결: API서버 구축
---
별도의 API서버를 설계할때, 먼저 어떤 언어를 사용할 서버를 띄울지 결정했습니다.

<span style="color:green">Spring 서버</span>를 띄워도 됐으나 다음과 같은 점 때문에 <span style="color:orange">Python</span> 웹 서버를 띄우기로 결정했습니다.

Python과 관련된 라이브러리로는

* Python을 서버에 띄울 <u>**FlaskAPI**</u>
* Python의 ORM인 <u>**SqlAlchemy**</u>

위 두 가지를 선택했습니다

Python으로 API서버 코드를 작성한 후 Docker로 배포했습니다.

이렇게 <span style="color:green">Spring 서버</span>와 분리하여 <span style="color:green">Spring</span>은 React와의 데이터 처리에 집중하게 했습니다.

<br/>

### 번외: Spring Data JPA 도입
---
번외로 Spring과 DB 사이의 쿼리문 처리를 JDBC로 하려다가 아래와 같은 이유 때문에 포기하고<br/>
ORM인 Spring Data JPA를 찾아서 사용법을 알아가며 사용하고 Spring 내부 구조도 Entity 관련 클래스를 추가하여 변경했습니다.

<details>
<summary>JDBC 포기한 이유</summary>
<div markdown="1">

* 반복되는 쿼리문
* 코드도 길어짐
* 유지보수 힘듦
</div>
</details>

<br/>
이제 개발하는 것과 관련하여 현 프로젝트 단계에서 크게 문제 될 수준은 없습니다.

<br/>

그래도 조금 더 욕심이 났습니다. 그것이 <u>**도메인 주소를 연결한 배포**</u>입니다.

즉, 외부 사용자가 <span style="color:red">**호스팅받은 도메인 주소로 접속**</span>하게 만들고 싶습니다.


<br/>
<br/>

## 마지막 구조 (Final Architecture)

![image](/images/2022/miniproject/miniproject-final_architecture.png){: .align-center}

### 첫번째 문제: ReactJS는 아직 배포를 못해 개발자 컴퓨터에서 localhost로만 접근 가능하다
---
[여기서 후순위로 미뤘던 문제](#첫번째-고민-reactjs는-아직-배포를-못해-개발자-컴퓨터에서-localhost로만-접근-가능하다)입니다.

아직 ReactJS는 개발자 컴퓨터에서만 `localhost`로만 접속할 수 있습니다.

먼저 React 관련 Docker 배포 설정을 검색해 NAS에 배포했습니다.

이 문제는 빠르게 해결했습니다.


<br/>

### 두번째 문제: 도메인 주소 연결
---
도메인 주소를 연결하기위해 

* [freenom](https://my.freenom.com/)이란 도메인 호스팅 사이트에서 moviescript 도메인을 등록

* 해당 도메인에 NAS IP 주소에 등록해서 연결

위와 같은 절차를 거쳤습니다.

그러나 아래와 같은 문제가 발생했습니다...

<br/>

### 세번째 문제: 포트 문제 해결... 그리고 NGINX 사용
---
바로 <u>**포트 에러 문제**</u>입니다.

보통 도메인 사이트에 접속할 때 포트번호를 적지 않습니다. 그러면 자동으로 **80번 포트**를 사용하게됩니다.

그러나 80포트를 NAS에서 이미 점유하고있습니다. (정확히는 Synology의 Web Station에서 점유)<br/> 

이를 해결하기위해 웹 서버를 사용하기로 결정했습니다.

검색해보니 대표적인 웹서버로는 <span style="color:orange">Apache</span> 혹은 <span style="color:green">NGINX</span>를 현재 많이 사용하고 있습니다.

급하게 검색해보니 <span style="color:green">NGINX</span>가 점점 많이 사용하고 역방향 프록시란 기능을 이용해 <span style="color:blue">React</span>와 편하게 연결할 수 있을 것 같아 사용하도록 계획했습니다.

하지만 또 문제가 발생했으니, <span style="color:green">NGINX</span>를 <span style="color:blue">Docker</span>로 배포하면 호스트 포트, 즉 NAS의 포트와 연결되고 NAS에선 여전히 80포트를 사용 중이라 여전히 사용할 수 없는 상황입니다.

이 문제를 해결하기 위해 <span style="color:blue">Docker</span>의 <span style="color:green">NGINX</span>컨테이너 IP를 외부에 바로 노출시키기 위해 관련 기술들을 조사했고, macvlan, ipvlan 등을 조사했으나... 

* 복잡한 네트워크 구조
* 서브넷과 게이트웨이까지 설정
* macvlan같은 경우 NIC의 promiscuous mode 설정

등 복잡한 설정을 해야하고 실력도 미흡해 결국 포기했습니다.

계속 검색하다 NAS 내부에 이미 별도의 웹서버가 설치되어있는 상태라는 것을 찾게되었고<br/>
편법으로 해당 NGINX에서 설정값만 변경하여 바로 ReactJS의 컨테이너 포트와 연결하도록 설정했습니다.

> 다만 이 방법은 어디까지나 편법입니다.<br/>
> 미리 NAS에서 설치된 웹 서버의 설정값만 변경해서 사용하기 때문이죠...<br/>
> Apache나 NGINX 설정에대해 더 공부해야할 필요성을 느꼈습니다.


<br/>

이렇게 백엔드 마지막 구조가 완성되었습니다.

사실 이 구조도 썩 만족스럽진않지만 현 프로젝트 단계에서 배포까지 완료한 최종 구조로 끝냈습니다.

추후엔 이미지를 어떻게든 처리할 수 있도록 별도의 이미지 서버를 설치하거나<br/>
Docker의 mount 기능을 사용할 계획입니다...