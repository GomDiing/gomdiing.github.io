---
title:  "Synology NAS에서 Joplin Server 설치하는 법"
excerpt: "Joplin Server 설치법"
categories:
  - installation
tags:
  - joplin
  - nas
  - synology
last_modified_at: 2025-04-27T17:00:00+09:00
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
---

# 개요

지금까지 동기화 메모앱은 애플의 메모, Synology에서 자체 제공하는 Synology Note Station를 써왔습니다.

다만 윈도우도 점점 써야할 일이 늘면서 애플 메모 앱은 그 한계성을 느꼈습니다. 

Synology의 Note Station은 자체서식도구를 제공하는 WYSIWYG(What You See Is What You Get) 에디터를 제공하고 큰 신경 쓸 필요없이 모바일을 포함한 다양한 플랫폼 앱을 제공해서 좋습니다.

그렇지만 생각보다 앱 실행이 무거우면서 마크다운 형식을 지원하지 않아 대체 메모 앱을 찾고 있었습니다. 

그 중 Joplin이 Docker로 Joplin Server를 제공하면서 마크다운 형식도 지원하고 다양한 모바일 앱도 지원하여 Joplin을 사용하기로 결정했습니다.

<br>

# Joplin Server 설치

당연하지만 NAS에 Docker가 이미 설치되어 있는 환경이어야 합니다.

먼저 Synology NAS에 접속해 Docker 이미지를 다운받아야 합니다. 

방법은 크게 2가지가 있습니다.

1. Container Manager를 통해 GUI로 설치

- Container Manager 앱 실행
- 레지스트리 탭 > joplin 검색 > joplin/server (공식 Joplin 이미지) 다운로드

2. 터미널 CLI의 명령어로 설치
SSH로 NAS에 접속해서 다음 명령어를 입력합니다:

```bash
docker pull joplin/server
```

joplin/server의 이미지에 대한 자세한 정보는 [Docker URL 주소](https://hub.docker.com/r/joplin/server)에서 확인할 수 있습니다.

<br>

# 컨테이너 생성 및 실행

컨테이너 생성 시 포트번호와 환경변수를 설정해야 합니다.

## GUI에서 설정하기

Container Manager에서 설정할 때 `자동 재시작 활성화`를 체크하면 NAS 재기동 시 알아서 재실행됩니다.

## 포트 설정
- 외부 포트 : 22300
- 내부 포트 : 22300

## 환경변수설정

```
- APP_PORT = 22300
- APP_BASE_URL = http://your-nas-ip:22300
- DB_CLIENT = sqlite3
- TZ = Asia/Seoul
```

## 유의사항

사용하려는 포트번호가 NAS 자체 방화벽에 막히지 않는지 확인하고, 
`APP_BASE_URL`과 실제 접속 URL이 일치하지 않으면 다음과 같은 오류가 발생합니다:

`Invalid origin: http://your-ip-nas:22300`

반드시 `APP_BASE_URL`을 실제 접속 URL과 동일하게 해야한다

<br>

# 공유기 포트포워딩

마지막으로 사용 포트번호(여기선 22300)를 공유기에서 포트포워딩하면 완료됩니다. 
(설정 방법은 공유기마다 다를 수 있어 생략)

<br>

# 아이폰과 연결하려면 HTTPS 필수

아이폰과 연결해서 사용하려면 HTTPS 포트번호를 사용해야 합니다. 

Synology NAS에 SSL/TLS 인증서를 발급받아 이미 사용하고 있다는 전제하에, 
처음부터 HTTPS로 설정하거나 역방향 프록시 기능을 사용해서 적용할 수 있습니다.

## 역방향 프록시 설정 방법

1. 제어판 > 로그인 포털 > 고급 탭 > 역방향 프록시
2. `생성` 버튼을 클릭하여 다음 규칙 입력:

- 설명 : Joplin Server

### 소스
  - 프로토콜 : HTTPS
  - 호스트이름 : your-nas-ip
  - 포트 : 사용할 HTTPS 포트 (예: 5001)

### 대상
  - 프로토콜 : HTTP
  - 호스트이름 : localhost
  - 포트 : 22300 (또는 사용 중인 포트번호)

<br>

# 관리자 계정으로 접속

이제 컨테이너를 실행한 다음 설정한 URL을 브라우저에 입력해 접속합니다.

`http://your-nas-ip:22300`

또는 HTTPS를 설정한 경우:

`https://your-nas-ip:5001`

로그인 창이 나타나면 초기 관리자 계정으로 로그인합니다:

- 계정명: admin@localhost
- 비밀번호: admin

그 후 업데이트하라는 알림을 클릭하면 새로운 계정과 비밀번호를 입력하는 창이 나타납니다. 

새 계정 정보를 입력하면 메일을 보냈다는 메시지가 표시되지만, 실제로는 메일이 발송되지 않으므로 직접 DB에서 확인해야 합니다.

<br>

# 인증 메일 확인

## 1. 컨테이너 접속

먼저 터미널로 joplin-server의 root 계정으로 접속합니다:

```bash
docker exec -it -u root joplin-server /bin/bash
```

## 2. SQLite 설치
필요한 도구를 설치합니다:
```bash
apt-get update
apt-get install -y sqlite3
```

## 3. DB 파일 검색
데이터베이스 파일을 찾습니다:
```bash
find / -name "*.sqlite" 2>/dev/null
```

## 4. DB 접속 및 이메일 확인
db-prod.sqlite 파일을 찾으면 다음 명령어로 접속합니다:
```bash
sqlite3 /path/to/db-prod.sqlite
```

SQLite가 실행되면 이메일 테이블을 조회합니다:
```sql
select * from emails;
```
인증 링크가 포함된 이메일 내용이 표시됩니다. 

해당 링크를 복사하여 브라우저에 붙여넣으면 인증이 완료되고 설정한 관리자 계정으로 접속할 수 있습니다.