# config

**함께하개의 설정 저장소입니다.** 서비스 17개의 설정값이 전부 여기 있습니다.

```
서비스 기동  ──▶  설정 서버  ──▶  config 저장소
                      │                 │
                      │                 └──▶  GitHub 에서 yml 을 읽음
                      │                       main 브랜치
                      │
                      └──▶  4계층을 순서대로 겹쳐 하나로 만들어 돌려줌

        서비스는 자기 application.yml 에 세 줄만 두고 나머지를 여기서 받음

        ⚠ 이 저장소는 공개임 — 비밀값은 ${환경변수} 자리만 남김
```

<br><br>

---

## 0. 이 저장소가 하는 일

**설정을 저장소에 두면 이렇게 됩니다.**

| | 여기에 두면 | 각 서비스에 두면 |
|---|---|---|
| 값을 바꾸면 | **push 하면 끝** | 빌드 → 이미지 → 배포 |
| DB 주소를 옮기면 | 한 줄 고침 | **14개 서비스를 다시 배포** |
| 환경별 차이 | 파일로 갈림 | 코드에 분기 |
| 값이 어디 있는지 | 여기 하나 | 저장소 17곳 |

---

**숫자로 보면 이렇습니다.**

| | 값 | 어디에 |
|---|---|---|
| yml 파일 | **23개** | [1장](#1-파일-지도) |
| 계층 | 4개 | [2장](#2-4계층--숫자가-큰-쪽이-이김) |
| 서비스 파일 | 17개 | 도메인 14 + 플랫폼 2 + 템플릿 1 |
| 환경 | 3개 | `local` · `dev` · `prod` |
| 4계층 실사례 | 2개 | `eureka-server-{local,dev}.yml` |
| **저장소 공개 여부** | **공개** | [4장](#4-비밀값은-여기-두지-않습니다) |

---

**이 저장소만의 특징 셋입니다.**

```
1  코드가 없음                 yml 23개와 README 뿐
                              빌드도 CI 도 없음

2  main 에 직접 커밋            이슈·PR·브랜치를 만들지 않음
                              push 하면 설정 서버가 바로 읽음

3  공개 저장소                  ⚠ 비밀값을 절대 적지 않음
                              한 번 커밋하면 지워도 이력에 남음
```

<br><br>

---

### 먼저 알아 두면 좋은 것 4가지

---

**① 설정이란 — 코드에서 뺀 "값"**

```
코드에 박으면                            설정으로 빼면

  url = "jdbc:postgresql://localhost"     url = ${DB 주소}
        │                                       │
        └── DB 를 옮기면 코드를 고치고            └── 값만 바꾸면 됨
            빌드하고 다시 배포                     코드는 그대로
```

**포트 · DB 주소 · 만료 시간 · 라우트 같은 것들이 전부 설정입니다.**
이 저장소는 그 값들을 모아 둔 곳이고, **코드는 한 줄도 없습니다.**

---

**② 프로파일이란 — "지금 어디서 돌고 있나"**

같은 서비스라도 **어디서 도느냐에 따라 값이 다릅니다.** 그것을 이름 하나로 가릅니다.

| 프로파일 | 어디서 | DB 주소 | Kafka 주소 |
|---|---|---|---|
| `local` | IntelliJ (내 컴퓨터) | `localhost` | `localhost:29092` |
| `dev` | 로컬 도커 컨테이너 | `postgres` | `kafka:9092` |
| `prod` | AWS EC2 | (아직) | (아직) |

**서비스를 띄울 때 프로파일을 하나 고르면** 그 이름이 붙은 파일이 함께 적용됩니다.

---

**③ `${...}` — 값이 아니라 자리**

```yaml
password: ${SERVICE_DB_PASSWORD}
          ▲
          └── "이 값은 여기 안 적음. 환경변수 SERVICE_DB_PASSWORD 에서 가져와라"
```

**비밀번호·키를 저장소에 적지 않기 위한 장치입니다.** 서비스가 뜰 때 자기 환경변수를
보고 채웁니다. **환경변수가 없으면 `${SERVICE_DB_PASSWORD}` 라는 문자열이 그대로 들어가
접속에 실패합니다** — 그것이 "빠뜨렸다" 는 신호입니다.

---

**④ YAML — 들여쓰기가 곧 구조**

```yaml
server:                    #  server
  port: 8084               #    └── port = 8084

spring:                    #  spring
  datasource:              #    └── datasource
    url: jdbc:...          #          └── url = jdbc:...
```

**공백 두 칸이 한 단계입니다.** 한 칸이라도 어긋나면 **오류 없이 다른 위치로 읽히거나
무시됩니다.** 기존 항목을 복사해 값만 바꾸는 편이 안전합니다.

<br><br>

---

### 이 문서를 읽는 순서

| 지금 하려는 일 | 볼 곳 |
|---|---|
| 어떤 파일이 있는지 보고 싶다 | [1장](#1-파일-지도) |
| 계층이 뭔지 모르겠다 | [2장](#2-4계층--숫자가-큰-쪽이-이김) |
| 새 값을 넣어야 하는데 어디 둘지 모르겠다 | [3장](#3-값을-어디에-둘지-정하기) |
| 비밀번호·키를 넣어야 한다 | [4장](#4-비밀값은-여기-두지-않습니다) |
| 새 서비스를 등록해야 한다 | [5장](#5-새-서비스-등록하기) |
| 고쳤는데 반영이 안 된다 | [6장](#6-값이-언제-반영되나) |
| 뭔가 안 된다 | [7장](#7-막히기-쉬운-자리) |
| "왜 이렇게 만들었지" | [8장](#8-왜-이렇게-만들었나) |
| 아직 안 채운 것이 궁금하다 | [9장](#9-아직-안-한-것) |

> **서비스를 만드는 사람이 실제로 하는 일은 [5장](#5-새-서비스-등록하기) 하나입니다.**
> 파일 하나를 만들고 게이트웨이 라우트를 여는 것이 전부입니다.

<br><br>

---

## 1. 파일 지도

```
paw-trail/config
│
├── application.yml               1계층   모든 서비스 · 모든 환경        178줄
│
├── application-local.yml         3계층   IntelliJ 로 띄울 때            75줄
├── application-dev.yml           3계층   로컬 컨테이너                  62줄
├── application-prod.yml          3계층   AWS EC2                       81줄 (거의 TODO)
│
├── 플랫폼 (2개)
│   ├── gateway-server.yml        2계층   *라우트 19 · 공개키 · 인증예외  293줄
│   ├── eureka-server.yml         2계층                                  44줄
│   ├── eureka-server-local.yml   4계층   *실사례                        25줄
│   └── eureka-server-dev.yml     4계층   *실사례                        15줄
│
├── 도메인 서비스 (14개)
│   ├── auth-service.yml          2계층   *제일 큼 — JWT · 메일 · OAuth  238줄
│   ├── user-service.yml                  17줄
│   ├── pet-service.yml                   23줄
│   ├── place-service.yml                 23줄
│   ├── policy-service.yml                23줄
│   ├── search-service.yml                17줄
│   ├── report-service.yml                23줄
│   ├── review-service.yml                17줄
│   ├── notification-service.yml          17줄
│   ├── verdict-service.yml               12줄   DB 없음 — 포트만
│   ├── congestion-service.yml            12줄   DB 없음
│   ├── route-service.yml                 12줄   DB 없음
│   ├── ingest-service.yml                23줄   배치 — auditor 를 덮음
│   └── extract-service.yml               30줄   배치
│
└── template-service.yml          2계층   service-template 을 그대로 띄울 때
```

> **`config-server.yml` 이 없습니다.** 설정 서버는 **자기 설정을 이 저장소에서
> 받지 않습니다.** 저장소 주소를 알아야 저장소를 읽을 수 있기 때문입니다.
> 그 값들은 `config-server` 저장소 안에 있습니다.

<br><br>

---

### 1-1. 파일 크기가 갈리는 이유

```
293줄  gateway-server    라우트 19개 · 공개키 PEM · 인증 예외 9줄
238줄  auth-service      JWT · SMTP · OAuth · 쿠키 · permit-all 9줄
 23줄  place-service     포트 · DB · outbox 스위치
 12줄  verdict-service   포트만
```

**대부분의 서비스는 짧습니다.** 공통값이 1계층에, 주소가 3계층에 있어서
**2계층에는 그 서비스만의 것만 남습니다.**

---

**12줄짜리 파일의 전부입니다.**

```yaml
# =============================================================================
# 2계층 — verdict-service
# =============================================================================
# 판정 담당임
#
# DB 를 쓰지 않으므로 datasource 를 두지 않음
# outbox 도 없어 relay 스위치를 적지 않음
# =============================================================================

server:
  port: 8086
```

<br><br>

---

### 1-2. 이름 규칙

```
<서비스명>.yml              2계층
<서비스명>-<환경>.yml         4계층
application.yml            1계층
application-<환경>.yml       3계층
```

**`<서비스명>` 은 그 서비스의 `spring.application.name` 과 정확히 같아야 합니다.**

```
auth-service/src/main/resources/application.yml
  spring.application.name: auth-service
                                │
                                └──▶  config 저장소의 auth-service.yml 을 찾음
```

> **다르면 오류 없이 그 계층만 빠진 채 내려갑니다.** 증상은 **포트가 스프링
> 기본값 8080 으로 뜨는 것**입니다.

---

**환경 이름에 하이픈을 쓰지 않습니다.**

```
설정 서버는 파일명에서 서비스명과 환경을 하이픈으로 가름
        │
        └── auth-service-local.yml
              ↑ 어디까지가 서비스명이고 어디부터가 환경인가
                  우리 서비스명이 전부 하이픈을 포함하므로
                  환경까지 하이픈이 있으면 경계를 못 찾음
```

**그래서 `local` · `dev` · `prod` 입니다.** `local-test` 같은 이름은 안 됩니다.

<br><br>

---

## 2. 4계층 — 숫자가 큰 쪽이 이김

```
place-service 가 local 프로파일로 뜰 때

  1계층  application.yml               모든 서비스 · 모든 환경
           ddl-auto: validate
           flyway.locations · out-of-order
           kafka 직렬화기 · observation
           datasource.password: ${SERVICE_DB_PASSWORD}
           auditor.system-name: SYSTEM
           outbox.relay.enabled: false
           │
           ▼  같은 키가 있으면 아래가 덮음
  2계층  place-service.yml             place · 모든 환경
           server.port: 8084
           datasource.url · username
           outbox.relay.enabled: true       ← 1계층의 false 를 덮음
           │
           ▼
  3계층  application-local.yml         모든 서비스 · local 만
           app.datasource.host: ${DB_HOST}
           redis · kafka · eureka · zipkin 주소
           cookie.secure: false
           │
           ▼
  4계층  place-service-local.yml       place · local 만
           (파일 없음 — 건너뜀)
           │
           ▼
  최종   port 8084 · outbox true · host=환경변수 · kafka localhost:29092 · ddl validate
```

<br><br>

---

### 2-1. 세기 규칙은 두 겹입니다

```
"구체적인 파일이 이긴다" 가 아님. 규칙이 두 겹임

  ① 프로파일이 붙은 파일이 안 붙은 파일을 이김
        application-local.yml  >  place-service.yml
        (3계층이 2계층을 이김)

  ② 같은 조건 안에서는 서비스별 파일이 공통 파일을 이김
        place-service.yml       >  application.yml         (2 > 1)
        place-service-local.yml >  application-local.yml   (4 > 3)


  환경별 공통값을 특정 서비스만 다르게 하려면
        ⛔ 2계층에 적으면 3계층에 덮임
        ✅ 4계층에 적어야 함
```

---

**실제로 그 자리를 쓰고 있는 것이 `eureka-server` 입니다.**

```yaml
# 2계층 — eureka-server.yml     (환경 무관)
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false

# 4계층 — eureka-server-local.yml
eureka:
  server:
    my-url: http://localhost:8761/eureka/

# 4계층 — eureka-server-dev.yml
eureka:
  server:
    my-url: http://eureka-server:8761/eureka/
```

**3계층(`application-local.yml`)에 두면 모든 서비스가 그 값을 받습니다.**
유레카 자신만 필요한 값이라 4계층이 맞습니다.

<br><br>

---

### 2-2. 각 계층에 무엇이 있나

**1계층 — `application.yml` (178줄)**

| 블록 | 무엇 |
|---|---|
| `spring.datasource.password` | `${SERVICE_DB_PASSWORD}` — 계정 10개가 공유 |
| `spring.cloud.refresh` | **DataSource 를 refresh 대상에 넣음** — [6장](#6-값이-언제-반영되나) |
| `spring.jpa` | `ddl-auto: validate` · `open-in-view: false` |
| `spring.flyway` | `locations` 두 곳 · `out-of-order: true` |
| `spring.kafka` | 직렬화기 · `observation-enabled` 둘 |
| `management` | 액추에이터 노출 · 추적 샘플링 1.0 |
| `app.auditor.system-name` | `SYSTEM` |
| `app.outbox.relay.enabled` | **`false`** — 발행하는 서비스만 2계층에서 켬 |

---

**2계층 — `<서비스명>.yml`**

```
포트                         전부
datasource.url · username   DB 를 쓰는 서비스만
outbox.relay.enabled: true   이벤트를 발행하는 서비스만
auditor.system-name          배치만 (ingest-batch · extract-batch)
그 서비스만의 값               auth 의 JWT · 메일 · OAuth
                            gateway 의 라우트 · 공개키
```

---

**3계층 — `application-{env}.yml`**

**환경마다 바뀌는 것은 사실상 "주소" 입니다.**

| 값 | `local` | `dev` | `prod` |
|---|---|---|---|
| `app.datasource.host` | `${DB_HOST}` | `${DB_HOST}` | TODO |
| Redis | `localhost:6379` | `redis:6379` | TODO |
| Kafka | `localhost:29092` | `kafka:9092` | TODO |
| 유레카 | `localhost:8761` | `eureka-server:8761` | TODO |
| Zipkin | `localhost:9411` | `zipkin:9411` | TODO |
| Loki | `localhost:3100` | `loki:3100` | TODO |
| `cookie.secure` | `false` | `false` | (아직) |

> **`local` 과 `dev` 가 갈리는 이유** — `local` 은 IntelliJ 가 도커 밖에서 돌아
> `localhost` 로 붙고, `dev` 는 컨테이너 안이라 **`localhost` 가 자기 자신**이라서
> 컨테이너 이름으로 붙습니다.

---

**4계층 — `<서비스명>-{env}.yml`**

**되도록 비워 둡니다.** 지금 유레카 둘뿐입니다.

<br><br>

---

### 2-3. 유레카 등록 방식이 환경마다 다릅니다

```yaml
# application-local.yml
eureka:
  instance:
    hostname: host.docker.internal
    prefer-ip-address: false

# application-dev.yml
eureka:
  instance:
    prefer-ip-address: true
```

```
local   IntelliJ 가 호스트에서 돎
          │
          └──▶  host.docker.internal 로 등록
                  *컨테이너 안의 게이트웨이가 이 이름으로 호스트를 찾음
                    한 이름이 호스트·컨테이너 양쪽에서 통함

dev     컨테이너 안에서 돎
          │
          └──▶  도커 네트워크 IP 로 등록
                  같은 네트워크라 IP 로 바로 닿음
```

**이것 덕분에 IntelliJ 로 띄운 서비스도 게이트웨이가 찾아냅니다.**

<br><br>

---

### 2-4. 실제로 받은 값 확인하기

```
http://localhost:8888/{서비스명}/{환경}
```

**macOS**

```bash
curl http://localhost:8888/place-service/local
```

**Windows (PowerShell)**

```powershell
curl.exe http://localhost:8888/place-service/local
```

응답의 `propertySources` 배열이 **어느 파일에서 온 값인지까지 보여 주며,
배열 앞이 우선순위가 높은 쪽**입니다.

```json
{
  "name": "place-service",
  "profiles": ["local"],
  "propertySources": [
    { "name": "...config/application-local.yml", "source": { } },
    { "name": "...config/place-service.yml",     "source": { } },
    { "name": "...config/application.yml",       "source": { } }
  ]
}
```

> **`.yml` · `.properties` · `.json` 주소는 쓸 수 없습니다.**
>
> ```
> http://localhost:8888/place-service-local.yml     →  400
> ```
>
> 설정 서버가 그 주소에서 **서비스명과 환경을 하이픈으로 가르는데**
> 우리 서비스명이 전부 하이픈을 포함하고 있습니다.

---

**`${...}` 는 치환되지 않은 채로 내려옵니다.**

```json
"spring.datasource.password": "${SERVICE_DB_PASSWORD}"
```

**설정 서버는 문자열 그대로 보내고 각 서비스가 자기 환경변수로 해석합니다.**
이것이 **공개 저장소로 둘 수 있는 전제**입니다.

<br><br>

---

## 3. 값을 어디에 둘지 정하기

```
새 값을 넣을 때

              서비스마다 다른가?
                      │
          ┌───────────┴───────────┐
         예                    아니오
          │                       │
          ▼                       ▼
  환경마다 다른가?        환경마다 다른가?
          │                       │
    ┌─────┴─────┐           ┌─────┴─────┐
   예        아니오        예        아니오
    │           │           │           │
    ▼           ▼           ▼           ▼
  4계층       2계층       3계층       1계층
{svc}-{env}   {svc}     app-{env}      app


  애매하면 번호가 작은 쪽에 둘 것
  나중에 큰 번호에서 덮어쓰는 것이 반대보다 쉬움
```

<br><br>

---

### 3-1. 실제 사례로 보기

| 값 | 서비스마다? | 환경마다? | 어디에 |
|---|---|---|---|
| `ddl-auto: validate` | 아니오 | 아니오 | **1계층** |
| `server.port: 8084` | 예 | 아니오 | **2계층** |
| Redis 주소 | 아니오 | 예 | **3계층** |
| `eureka.server.my-url` | 예 | 예 | **4계층** |

---

**`app.outbox.relay.enabled` 가 좋은 예입니다.**

```
1계층   enabled: false          기본값 — 대부분의 서비스는 발행을 안 함
          │
          ▼
2계층   enabled: true           발행하는 서비스만 켬
                                auth · place · policy · pet · report
```

**기본값을 "꺼짐" 으로 둔 이유** — 켜는 것을 잊으면 이벤트가 5초 늦게 나갈 뿐이지만,
**끄는 것을 잊으면 여러 인스턴스가 같은 행을 집어 순서 보장이 깨집니다.**

---

**`app.auditor.system-name` 도 같은 모양입니다.**

```
1계층   SYSTEM              전부
2계층   ingest-batch        ingest 만
        extract-batch       extract 만
```

<br><br>

---

### 3-2. 기본값을 박지 않습니다

```yaml
# ⛔ 이렇게 쓰지 않습니다
password: ${SERVICE_DB_PASSWORD:1234}
host: ${DB_HOST:localhost}
```

```
환경변수를 빠뜨려도 그냥 붙어 버림
        │
        └── 누락이 영영 안 드러남
              EC2 로 옮길 때 환경변수를 안 넣은 사람이
              조용히 로컬 DB 에 붙게 됨 — 그편이 더 나쁨
```

**환경변수가 없으면 기동이 실패하는 편이 낫습니다.**

> `${CONFIG_HOST:localhost}` 만 예외입니다. 그건 **저장소가 아니라 서비스의
> `application.yml`** 에 있고, **설정 서버 주소는 로컬에서 언제나 `localhost:8888`**
> 이라 기본값이 정답입니다.

<br><br>

---

### 3-3. 값이 다른 키를 참조할 수 있습니다

```yaml
# 2계층 — place-service.yml
spring:
  datasource:
    url: jdbc:postgresql://${app.datasource.host}:5432/place_db

# 3계층 — application-local.yml
app:
  datasource:
    host: ${DB_HOST}
```

```
합친 뒤에 풀림
        │
        └── 2계층의 url 이 3계층의 host 를 쓸 수 있음
              계층 순서와 무관하게 최종 결과에서 해석됨
```

**`app.datasource.host` 한 줄이 DB 주소의 단일 출처입니다.**
DB 를 옮길 때 고치는 자리가 **3계층의 그 한 줄뿐**입니다.

<br><br>

---

## 4. 비밀값은 여기 두지 않습니다

**⚠ `paw-trail/config` 는 공개 저장소입니다.**

```
비밀번호 · 개인키 · API 시크릿
        │
        └── 어느 계층에도 넣지 않음
              자리만 ${환경변수} 로 남김
```

<br><br>

---

### 4-1. 무엇이 갈리나

| | config 저장소 | 환경변수 |
|---|---|---|
| DB 비밀번호 | | ✓ `SERVICE_DB_PASSWORD` |
| RS256 **개인키** | | ✓ `AUTH_JWT_PRIVATE_KEY_B64` |
| RS256 **공개키** | ✓ PEM 원본 그대로 | |
| Gmail 앱 비밀번호 | | ✓ `AUTH_MAIL_PASSWORD` |
| 구글 클라이언트 **시크릿** | | ✓ `AUTH_OAUTH_GOOGLE_CLIENT_SECRET` |
| 구글 클라이언트 **ID** | ✓ 값 그대로 | |
| DB 호스트 | | ✓ `DB_HOST` (사람마다 다름) |
| 포트 · 만료 시간 · 경로 | ✓ | |

---

**기준은 "새면 무엇을 할 수 있나" 입니다.**

```
공개키       확인만 되고 만들 수는 없음        →  공개돼도 무해
개인키       누구나 유효한 토큰을 만들 수 있음   →  절대 안 됨

클라이언트 ID  authorize URL 에 실려 주소창에 뜸
             남이 알아도 등록된 redirect URI 로만 되돌아감  →  무해
클라이언트 시크릿  토큰 교환에 쓰임                        →  절대 안 됨
```

---

**공개키는 블록 스칼라로 둡니다.**

```yaml
# gateway-server.yml
app:
  jwt:
    public-key: |
      -----BEGIN PUBLIC KEY-----
      MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
      -----END PUBLIC KEY-----
```

> **들여쓰기가 일정해야 합니다.** 어긋나면 게이트웨이가 키 파싱에 실패하고
> 기동이 막힙니다.
>
> **Base64 한 줄로 벗기지 않는 이유** — 헤더·개행을 손대다 한 글자만 틀려도
> *"서명 검증 실패"* 로만 나타나 원인 추적이 고약합니다.

<br><br>

---

### 4-2. 실수로 넣었다면

```
한 번 커밋한 값은 지워도 이력에 남음
        │
        └── 되돌리는 것으로 끝나지 않음
              그 키를 새로 발급해야 함
```

| 무엇 | 어떻게 |
|---|---|
| DB 비밀번호 | `infra/.env` 를 바꾸고 DB 를 다시 만듦 |
| RS256 키 | **새 쌍을 만들고 개인키·공개키를 함께 교체** — `gateway-server` README 7-2 |
| Gmail 앱 비밀번호 | 구글 계정에서 폐기하고 재발급 |
| 구글 시크릿 | 클라우드 콘솔에서 재발급 |

<br><br>

---

## 5. 새 서비스 등록하기

**서비스를 만드는 사람이 이 저장소에서 하는 일은 이것뿐입니다.**

```
① config 저장소를 clone            처음 한 번
        │
        ▼
② <서비스명>.yml 을 루트에 만듦      2계층
        │
        ▼
③ gateway-server.yml 에 라우트 추가
        │
        ▼
④ 인증 없이 열 경로가 있으면 permit-all 에도
        │
        ▼
⑤ main 에 커밋 · push              이슈·PR 없이
        │
        ▼
⑥ 확인   curl :8888/<서비스명>/local
```

<br><br>

---

### 5-1. ② 서비스 파일 만들기

**DB 를 쓰는 서비스**

```yaml
# =============================================================================
# 2계층 — place-service
# =============================================================================
# 장소 담당임
#
# 호스트는 3계층의 app.datasource.host 에서 오고 비밀번호는 1계층에 있음
# 계정 10개가 같은 비밀번호를 쓰므로 여기에는 계정명만 둠
# =============================================================================

server:
  port: 8084

spring:
  datasource:
    url: jdbc:postgresql://${app.datasource.host}:5432/place_db
    username: place_svc

app:
  outbox:
    relay:
      # place.updated 를 발행함
      # 단일 인스턴스라 켜도 같은 행을 두 번 집을 일이 없음
      enabled: true
```

**DB 를 안 쓰는 서비스**

```yaml
server:
  port: 8086
```

---

| 값 | 규칙 |
|---|---|
| `server.port` | 아래 배정표 |
| `username` | **`<서비스>_svc`** — `_user` 가 아닙니다 |
| `url` 의 DB 이름 | `<서비스>_db` (`ingest` 만 `raw_db`) |
| `outbox.relay.enabled` | 이벤트를 **발행하는** 서비스만 `true` — auth · place · policy · pet · report |
| `auditor.system-name` | **배치만** — `ingest-batch` · `extract-batch` |

---

**포트 배정표입니다.** 이 저장소의 `<서비스명>.yml` 이 포트의 단일 출처입니다.

```
플랫폼                      도메인
  8080  gateway-server        8081  auth        8088  ingest
  8761  eureka-server         8082  user        8089  extract
  8888  config-server         8083  pet         8090  congestion
                              8084  place       8091  route
                              8085  policy      8092  report
                              8086  verdict     8093  notification
                              8087  search      8094  review
                                                8095  template (검증용)
```

> **`docker-compose.yml` 의 포트 매핑과 Prometheus 타깃도 같은 번호를 씁니다.**
> 여기서 바꾸면 그 둘도 함께 봐야 합니다.

> **주석을 다는 형태를 지킵니다.** 파일 첫머리에 계층 · 서비스명 · 담당 · 값의
> 출처를 적습니다. 나중에 보는 사람이 **"이 값이 어디서 오나" 를 파일 안에서
> 알 수 있어야 합니다.**

<br><br>

---

### 5-2. ③ 게이트웨이 라우트

```yaml
spring:
  cloud:
    gateway:
      server:
        webflux:
          routes:
            - id: auth-service
              uri: lb://auth-service
              predicates:
                - Path=/api/v1/auth/**

            - id: place-service                        # ← 추가
              uri: lb://place-service
              predicates:
                - Path=/api/v1/places/{placeId},/api/v1/places/{placeId}/documents
```

| 항목 | 규칙 | 어기면 |
|---|---|---|
| `uri` | `lb://` + 그 서비스의 `spring.application.name` | **503** |
| `predicates` | 보낼 경로 | 안 적으면 **404** |
| 관리자 경로 | `/api/v1/admin/{리소스}/**` 로 따로 | — |

> **`/api/v1/places/` 아래에는 `/**` 를 쓰지 않습니다.** 서비스 6개가 섞여 있어
> 첫 라우트가 전부 먹습니다. `gateway-server` README 3-2 참고.
>
> **YAML 은 들여쓰기가 곧 구조입니다.** `- id:` 앞 공백이 한 칸이라도 다르면
> **오류 없이 무시됩니다.** 위 항목을 복사해 값만 바꾸는 편이 안전합니다.

<br><br>

---

### 5-3. ④ 인증 예외

```yaml
# gateway-server.yml
app:
  gateway:
    permit-all:
      - /api/v1/auth/signup
      # ... 9줄
```

**auth 는 자기 목록도 갖고 있습니다.**

```
gateway-server.yml   app.gateway.permit-all    게이트웨이가 토큰 없이 통과시킴
auth-service.yml     app.auth.permit-all       auth 보안 체인이 열어 둠
```

**같은 9줄이 양쪽에 있어야 합니다.** 한쪽에만 빠지면 401 이 납니다.

> **auth 만 그렇습니다.** auth 가 자기 `SecurityFilterChain` 을 정의해
> 공통 모듈의 규칙이 물러나기 때문입니다. 다른 서비스는 게이트웨이 쪽만 적으면 됩니다.

<br><br>

---

## 6. 값이 언제 반영되나

```
① config 저장소에 push
        │
        │  설정 서버는 요청이 올 때마다 GitHub 을 읽음
        │  *재시작이 필요 없음
        ▼
② 이미 떠 있는 서비스에 반영 — 둘 중 하나
        │
        ├──▶  POST /actuator/refresh        재시작 없이
        └──▶  서비스 재기동
        │
        ▼
③ 확인   curl :8888/<서비스명>/local
```

<br><br>

---

### 6-1. 무엇이 refresh 로 되고 무엇이 안 되나

| 바꾼 것 | 필요한 작업 |
|---|---|
| config 저장소의 값 | **push 하면 끝.** 설정 서버는 그대로 |
| 이미 떠 있는 서비스 | `POST /actuator/refresh` 또는 재기동 |
| 서비스 저장소의 `application.yml` | **재배포** |
| 환경변수 | **컨테이너 재시작** |
| 액추에이터 엔드포인트 등록 | **재기동** — 등록은 기동 시점에 일어남 |

```bash
curl -X POST http://localhost:8084/actuator/refresh
```

**바뀐 키 목록이 배열로 돌아옵니다.**

```json
["app.datasource.host", "server.port"]
```

> **빈 배열이면 아무것도 안 바뀐 것입니다.** push 를 안 했거나 값이 같습니다.

<br><br>

---

### 6-2. refresh 가 DataSource 까지 다시 만듭니다

```yaml
# 1계층 — application.yml
spring:
  cloud:
    refresh:
      extra-refreshable: javax.sql.DataSource,com.zaxxer.hikari.HikariDataSource
      never-refreshable: ""
```

```
이 두 줄이 없으면
        │
        ├──▶  프로퍼티만 다시 바인딩됨
        └──▶  ⛔ 커넥션 풀은 옛 주소를 그대로 물고 있음
                  refresh 응답이 정상이고 바뀐 키가 나와도 그러함
```

**DB 를 옮겼을 때 주소만 바꾸고 재배포 없이 전환하기 위한 것입니다.**
**지우지 않습니다.**

> `app.datasource.host` 한 줄을 3계층에서 고치고 refresh 하면
> **14개 서비스가 새 DB 를 봅니다.**

<br><br>

---

### 6-3. 포트로는 판별이 안 되는 서비스가 있습니다

| 서비스 | 설정 미수신 판별 |
|---|---|
| 도메인 서비스 | 포트가 **8080** 으로 뜸 (정상 포트가 8081~8095) |
| eureka-server | 포트가 **8080** 으로 뜸 (정상 8761) |
| **gateway-server** | ⛔ **정상 포트가 8080 이라 판별 불가** |

**게이트웨이는 `/actuator/gateway/routes` 가 비어 있는 것으로 가려냅니다.**

```bash
curl http://localhost:8080/actuator/gateway/routes
```

<br><br>

---

## 7. 막히기 쉬운 자리

<br><br>

---

### 7-1. 값이 안 내려올 때

| 증상 | 원인 |
|---|---|
| 포트가 8080 으로 뜸 | **2계층 파일을 못 찾음.** 파일명이 `spring.application.name` 과 같은지 |
| `propertySources` 가 비어 있음 | 같음 |
| `${DB_HOST}` 가 그대로 들어감 | **환경변수를 안 넣음.** `UnknownHostException: ${DB_HOST}` |
| `:8888/서비스명-local.yml` 이 400 | **확장자 주소는 못 씀.** `/서비스명/local` 로 |
| 고쳤는데 그대로 | push 를 안 했거나 refresh 를 안 함 |
| 게이트웨이 라우트가 0개 | config 미수신 |

**대조하는 것이 요령입니다.**

```
:8888/place-service/local              내가 쓴 것
:8084/actuator/env                     서비스가 실제로 들고 있는 것
```

<br><br>

---

### 7-2. YAML 을 고칠 때

| 하려는 것 | 주의 |
|---|---|
| 최상위 키를 추가 | **같은 키가 두 번 나오면 앞의 블록이 통째로 무시됨** |
| 라우트 추가 | `- id:` 앞 공백이 한 칸만 달라도 무시됨 |
| 공개키 붙여넣기 | 블록 스칼라 안의 들여쓰기가 일정해야 함 |
| `prod` 의 TODO 주석 풀기 | **`app` · `spring` 등이 두 번 나오지 않게 하나의 트리로** |

```yaml
# ⛔ 이렇게 되면 앞의 app 블록이 통째로 무시됨
app:
  auth:
    cookie:
      secure: false

app:                    # ← 두 번째 app
  datasource:
    host: 10.0.0.0
```

---

**검사하는 법**

```bash
python3 -c "import yaml; yaml.safe_load(open('place-service.yml', encoding='utf-8')); print('OK')"
```

**설정 서버로도 확인됩니다.**

```bash
curl http://localhost:8888/place-service/local
```

**500 이 나오면 YAML 이 깨진 것입니다.**

<br><br>

---

### 7-3. 새 값에 검증을 붙일 때

**서비스의 `Properties` 클래스에 *"비면 기동을 막는"* 검증을 추가하면
세 곳을 함께 고쳐야 합니다.**

```
config/<서비스명>.yml                              실제 값
<서비스>/src/test/resources/application.yml        테스트용 사본   ← 놓치기 쉬움
<서비스>의 Properties 클래스                        검증
```

```
테스트는 spring.cloud.config.enabled: false 라
config 저장소 값이 하나도 안 내려옴
        │
        └── config 에만 넣고 테스트 리소스를 안 고치면
              contextLoads 가 실패해 빌드가 깨짐
```

> **실제로 겪었습니다.** `app.jwt.claim.type` 을 넣을 때 auth·게이트웨이 양쪽에서
> 테스트 리소스를 빠뜨렸습니다.

<br><br>

---

### 7-4. 순서가 중요한 경우

**서비스에 새 검증이 들어갈 때는 config 를 먼저 올립니다.**

```
① config 저장소에 새 값 push
        │
        ▼
② docker compose restart config-server      (컨테이너로 돌 때)
        │
        ▼
③ 서비스 배포
```

**①을 안 하고 ③을 하면 새 검증에 걸려 기동이 막힙니다.** 의도한 동작입니다.

<br><br>

---

## 8. 왜 이렇게 만들었나

<br><br>

---

### 8-1. 왜 저장소를 따로 두나

| 고름 | 버림 |
|---|---|
| config 저장소 + 설정 서버 | 각 서비스의 `application.yml` |
| **값을 바꾸면 push 로 끝** | 빌드 → 이미지 → 배포 한 사이클 |
| 값이 한 곳에 모임 | 저장소 17곳에 흩어짐 |
| 환경별 차이가 파일로 갈림 | 코드에 분기 |

**결정적인 자리가 둘입니다.**

```
DB 승격        3계층의 app.datasource.host 한 줄 + refresh
                재배포 없이 14개 서비스가 새 DB 를 봄

라우트 개방     앞으로 14번 일어남
                게이트웨이 코드에 있으면 그때마다 blue-green 배포
```

<br><br>

---

### 8-2. 왜 공개 저장소인가

```
전제 — 설정 서버는 ${...} 를 치환하지 않고 문자열 그대로 내려보냄
        │
        └── 비밀값이 애초에 이 저장소에 존재하지 않음
              각 서비스가 자기 환경변수로 해석함
```

**공개로 두면 얻는 것입니다.**

| | |
|---|---|
| 설정 서버가 인증 없이 읽음 | private 이면 토큰을 설정 서버에 넣어야 함 |
| 팀원이 바로 봄 | 권한 관리가 없음 |
| 심사에서 그대로 보여줄 수 있음 | |

> ⚠ **대신 실수의 대가가 큽니다.** 한 번 커밋하면 지워도 이력에 남고
> **키를 새로 발급해야 합니다.**

<br><br>

---

### 8-3. 왜 라우팅을 여기 두나

**게이트웨이 코드가 아니라 config 2계층에 있습니다.**

```
라우트 개방이 앞으로 14번 일어남
        │
        ├── 코드에 있으면   빌드 → 이미지 push → blue-green 배포
        └── config 에 있으면  push + /actuator/refresh
```

| | config yml | 자바 `RouteLocator` |
|---|---|---|
| 오타 검증 | 없음 | 컴파일 |
| 오타 증상 | **404 — 첫 호출에서 드러남** | — |
| 환경별 차이 | 4계층으로 | 코드 분기 |

> **대가는 영향 범위입니다.** 지금 config 오타는 *"그 서비스 하나"* 가 포트를
> 못 잡는 정도인데, 라우트가 들어오면 **오타 하나로 특정 API 전체가 404** 가 됩니다.

<br><br>

---

### 8-4. 왜 `config-server.yml` 이 없나

```
config-server 만 닭-달걀 문제가 있음
        │
        └── 저장소 주소를 알아야 저장소를 읽을 수 있음
              그 값을 저장소에서 받을 수 없음
```

**그래서 설정 서버는 자기 저장소 안에 설정을 둡니다.**

**예외를 그것 하나로 줄인 것이 이득입니다.** 게이트웨이·유레카는 그 문제가 없어
여기서 받습니다.

> **컨테이너로 띄울 때 config-server 만 주소 환경변수를 직접 받습니다.**
> `EUREKA_HOST` · `LOKI_HOST` 인데, **dev 프로파일이 주소를 바꿔 주는 경로가
> 없기 때문**입니다.
>
> 빠뜨리면 `localhost` 로 남는데 **컨테이너 안에서 그것은 자기 자신이라
> 유레카 등록이 영영 실패**하고, 등록 실패가 기동을 막지 않아
> **유레카 화면에 안 보이는 것으로 알아차리게 됩니다.**

<br><br>

---

### 8-5. 왜 `main` 에 직접 커밋하나

```
이 저장소에는
        코드가 없음         빌드도 CI 도 없음
        리뷰할 것이 없음     YAML 23개
        설정 서버가 main 을 읽음
```

**이슈·PR·브랜치를 만들면 얻는 것 없이 단계만 늘어납니다.**

> **도메인 서비스는 다릅니다.** 거기는 1이슈-1브랜치-1PR 이고 CodeRabbit 리뷰가 붙습니다.

<br><br>

---

## 9. 아직 안 한 것

<br><br>

---

### 9-1. `prod` 가 거의 비어 있습니다

```yaml
# application-prod.yml — 지금 살아 있는 것은 이것뿐
logging:
  level:
    com.pawtrail: INFO
```

**나머지는 주석 처리된 TODO 입니다.**

| 채울 것 | 언제 |
|---|---|
| `app.datasource.host` | **AWS 노드 구성이 정해지면** — data-primary 사설 IP |
| `app.logging.loki.url` | 관측 노드 사설 IP |
| `spring.data.redis.host` | Redis 가 뜨는 노드 |
| `spring.kafka.bootstrap-servers` | Kafka 노드 |
| `eureka.client.service-url` | edge 노드 |
| `management.tracing...zipkin` | 관측 노드 |
| `app.auth.cookie.secure: true` | **nginx 에 인증서를 붙일 때** |
| `app.oauth.*` | **도메인이 생길 때** — 구글은 IP 와 HTTP 를 거부 |

> ⚠ **채우기 전에 `prod` 프로파일로 띄우지 않습니다.**
> 값이 없으면 **1계층 값이 그대로 내려가 local 주소로 붙으려 시도하며
> 오류 없이 이상하게 동작합니다.**

---

**주석을 풀 때 주의합니다.**

```
최상위 키(app · spring · eureka · management)가 두 번 나오지 않게 할 것
        │
        └── YAML 중복 키가 되면 앞의 블록이 통째로 무시됨
              지금은 하나의 트리로 이어 두었으므로 블록째 주석만 풀면 됨
```

<br><br>

---

### 9-2. 시점이 정해진 것

| 언제 | 무엇 |
|---|---|
| **EC2 를 세울 때** | `prod` 주소 전부 · `application-dev.yml` 의 DB 주석 재검토 |
| **nginx 를 붙일 때** | `cookie.secure: true` · OAuth 배포 주소 |
| **AWS 배포 때** | **RS256 키 페어를 새로 만들고 `gateway-server-prod.yml` 에 공개키** |
| user·pet 착수 시 | 각 서비스 파일에 `outbox.relay.enabled` 확인 |

> ⚠ **키 페어를 바꿀 때 짝이 어긋나면 전 요청이 401 입니다.**
> 개인키(auth 환경변수)와 공개키(여기)를 **함께** 바꿔야 합니다.

<br><br>

---

### 9-3. 판단만 남은 것

| 무엇 | 상태 |
|---|---|
| 4계층을 더 쓸지 | 지금 유레카 둘뿐. **되도록 비워 두는 것이 방침** |
| `prod` 를 어떻게 검증할지 | EC2 가 서기 전에는 확인할 방법이 없음 |
| 조직 전용 BOM | Boot 버전이 21개 레포에 사본으로 있음 |

<br><br>

---

## 10. 용어

공통 용어는 `service-template` README 11장에 있습니다.
**여기는 설정에서만 쓰는 말**입니다.

| 용어 | 뜻 |
|---|---|
| **설정 서버** | 이 저장소를 읽어 각 서비스에 설정을 내려 주는 서버. `config-server` 저장소 |
| **계층** | 설정 파일의 우선순위 단계. 1~4이고 **숫자가 큰 쪽이 이김** |
| **프로파일** | 환경 이름. `local` · `dev` · `prod` |
| **`propertySources`** | 설정 서버 응답에서 **어느 파일에서 온 값인지** 보여 주는 배열. 앞이 우선순위가 높음 |
| **`${...}`** | 플레이스홀더. 설정 서버는 치환하지 않고 **문자열 그대로 내려보냄** |
| **블록 스칼라 (`\|`)** | 여러 줄 문자열을 그대로 담는 YAML 문법. 공개키가 이 형태 |
| **`/actuator/refresh`** | 서비스가 설정을 다시 읽는 엔드포인트. `POST` |
| **`extra-refreshable`** | refresh 할 때 **다시 만들 빈**을 지정. DataSource 가 여기 있음 |
| **닭-달걀 문제** | 설정 서버가 자기 설정을 저장소에서 못 받는 것. **저장소 주소를 알아야 저장소를 읽음** |
| **`lb://`** | 게이트웨이 라우트에서 *"유레카에서 이 이름으로 찾아라"* 는 표시 |
| **`permit-all`** | 인증 없이 통과시킬 경로 목록. 게이트웨이와 auth 양쪽에 같은 9줄 |
| **`out-of-order`** | Flyway 가 **번호 순서를 건너뛴 스크립트도 실행**하게 하는 옵션. 공통 대역 때문에 켜 둠 |
