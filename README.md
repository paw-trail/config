# config

함께하개(paw-trail)의 모든 서비스가 사용하는 설정 저장소입니다. 자바 코드는 한 줄도 없고 YAML 파일만 들어 있습니다.

이 저장소를 읽어 각 서비스에 설정을 내려주는 애플리케이션은 `config-server` 이며 별도 저장소입니다. 이름이 비슷하므로 혼동하지 않도록 주의합니다.

```
paw-trail/config          이 저장소. YAML 파일 모음이며 실행되지 않습니다
paw-trail/config-server   이 저장소를 읽어 서비스에 내려주는 스프링 애플리케이션
```

**이 저장소는 공개되어 있습니다.** 비밀번호와 개인키는 어느 파일에도 넣지 않으며, 자리만 `${환경변수}` 형태로 남기고 실제 값은 각 컨테이너의 환경변수로 주입합니다. 자세한 목록은 5장에 있습니다.

---

## 1. 설정이 서비스에 도달하는 경로

### 1-1. 서비스가 직접 찾아옵니다

각 서비스는 기동할 때 자기 이름과 프로파일을 알리고 설정을 받아옵니다. 유레카를 거치지 않고 `config-server` 주소로 바로 찾아갑니다.

```
서비스 기동
   │
   ├─→ "나는 auth-service, 프로파일은 dev" 라고 요청
   │
   ├─→ config-server 가 이 저장소에서 해당 파일들을 찾아 합침
   │
   └─→ 서비스는 합쳐진 설정 한 벌을 받습니다
```

합치는 작업은 `config-server` 가 응답을 만들 때 끝내므로, 서비스 쪽에서는 계층이 보이지 않고 완성된 한 벌로 인식됩니다.

### 1-2. 서비스 저장소에 남는 것

각 서비스의 `src/main/resources/application.yml` 에는 세 줄만 남습니다.

```yaml
spring:
  application:
    name: auth-service
  config:
    import: "optional:configserver:http://${CONFIG_HOST:localhost}:8888"
  profiles:
    default: local
```

`optional:` 을 붙이는 이유는 `config-server` 가 떠 있지 않아도 서비스가 기동되게 하기 위함입니다. 서비스 하나만 IntelliJ로 띄워 확인하는 작업이 자주 있으므로 이 접두사가 없으면 매번 `config-server` 를 함께 띄워야 합니다.

`${CONFIG_HOST:localhost}` 에 기본값을 붙이는 것은 의도입니다. 이 세 줄은 **서비스 저장소**에 있는 값이며, 로컬에서는 언제나 `localhost:8888` 이므로 기본값이 정답입니다. 기본값이 없으면 개발자마다 IntelliJ 실행 구성에 환경변수를 넣어야 합니다. 컨테이너와 AWS에서만 `CONFIG_HOST` 를 지정해 덮어씁니다.

**이 저장소 안의 파일에는 기본값을 붙이지 않습니다.** 이유는 5장에 있습니다.

---

## 2. 4계층 구조

### 2-1. 덮어쓰기 순서

`config-server` 는 아래 순서로 파일을 찾아 **뒤엣것이 앞엣것을 덮어씁니다.** 구체적인 파일이 이깁니다.

| 순위 | 파일 | 적용 범위 |
|:---:|---|---|
| 1 | `application.yml` | 모든 서비스 · 모든 환경 |
| 2 | `application-{env}.yml` | 모든 서비스 · 해당 환경만 |
| 3 | `{서비스명}.yml` | 해당 서비스 · 모든 환경 |
| 4 | `{서비스명}-{env}.yml` | 해당 서비스 · 해당 환경만 |

### 2-2. 파일명은 `spring.application.name` 과 같아야 합니다

파일명이 다르면 **오류 없이 상위 계층 값만 내려갑니다.** 값이 반영되지 않는데 아무 메시지도 없다면 이것을 먼저 확인합니다.

이 이름은 저장소 이름, 컨테이너 이미지 이름, 유레카 등록 이름과도 같게 유지합니다.

```
저장소명 = 이미지명 = spring.application.name = 이 저장소의 파일명 = 유레카 서비스 ID
```

### 2-3. 4계층은 되도록 비웁니다

서비스 14개와 환경 3개를 곱하면 파일이 42개까지 늘어날 수 있습니다. 그러나 그중 대부분은 3계층에 있어야 할 값을 잘못 내려놓은 경우입니다.

예를 들어 `auth-service` 의 포트 8081은 환경이 바뀌어도 8081이므로 3계층에 둡니다. 4계층에 들어갈 값은 **이 서비스가 이 환경에서만 다르게 동작하는 것**뿐입니다.

---

## 3. 환경 프로파일

프로파일은 3개이며, 축의 기준은 **어디에서 실행되는가** 입니다.

| 프로파일 | 실행 위치 | 접속 대상의 예 |
|---|---|---|
| `local` | IntelliJ에서 직접 실행 | 호스트에서 컨테이너를 바라봄 (`localhost:29092`) |
| `dev` | 로컬 `docker compose` 의 `app` 프로파일 | 같은 도커 네트워크 안 (`kafka:9092`) |
| `prod` | AWS EC2 | 노드가 나뉘어 있어 사설 IP를 사용 |

`local` 과 `dev` 는 같은 PC에서 돌지만 접속 주소가 다릅니다. 한 프로파일로 묶으면 두 주소를 한 파일에 담을 수 없습니다.

프로파일을 지정하지 않으면 `local` 로 동작합니다. 서비스의 `application.yml` 에 `spring.profiles.default: local` 이 들어 있기 때문이며, 컨테이너에서는 Compose의 `SPRING_PROFILES_ACTIVE=dev` 가 이깁니다.

---

## 4. 값을 어디에 두는가

### 4-1. 판단 기준

값을 하나 추가할 때 아래 2가지만 판단합니다.

```
                  ┌─────────────────┬───────────────────────┐
                  │   환경 무관      │   환경마다 다름         │
 ┌────────────────┼─────────────────┼───────────────────────┤
 │  모든 서비스    │ application.yml │ application-{env}.yml │
 ├────────────────┼─────────────────┼───────────────────────┤
 │  서비스마다     │ {서비스명}.yml   │ {서비스명}-{env}.yml   │
 └────────────────┴─────────────────┴───────────────────────┘
```

애매하면 위 계층에 둡니다. 아래에서 덮어쓰는 것이 위로 끌어올리는 것보다 쉽습니다.

### 4-2. 계층별 예시

| 계층 | 들어가는 값 |
|---|---|
| `application.yml` | `ddl-auto`, Flyway `locations`, 로깅 패턴, 액추에이터 노출 범위, Kafka 직렬화기와 Observation, 서킷 브레이커 기본값 |
| `application-{env}.yml` | 데이터베이스 호스트, Kafka 부트스트랩 주소, Redis 주소, 유레카 주소, Loki · Zipkin 주소 |
| `{서비스명}.yml` | 포트, 데이터베이스 이름과 계정, `app.auditor.system-name`, `app.outbox.relay.enabled`, 구독 토픽 |
| `{서비스명}-{env}.yml` | 되도록 비워 둡니다 |

---

## 5. 이 저장소에 넣지 않는 것

### 5-1. 환경변수로 빼는 값

| 값 | 비고 |
|---|---|
| 서비스 계정 비밀번호 | 데이터베이스 10개분 |
| **RS256 개인키** | 유출되면 누구나 유효한 토큰을 만들 수 있습니다 |
| 카카오 REST 키 · 시크릿 | |
| 공공데이터 `serviceKey` | 호출 쿼터가 붙어 있습니다 |
| SMTP 앱 비밀번호 | |
| S3 자격증명 | |
| `local` 프로파일의 데이터베이스 호스트 | 공인 IP이며 5432가 열려 있습니다 |

**RS256 공개키는 이 저장소에 둡니다.** 서명 검증에만 쓰이므로 공개되어도 무해하며, 게이트웨이가 이 저장소를 통해 공개키를 받습니다.

작성 형태는 다음과 같습니다.

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

**이 저장소의 파일에는 기본값을 함께 적지 않습니다.** `${DB_PASSWORD:1234}` 처럼 쓰면 환경변수를 빠뜨려도 접속이 되어 버려 누락이 영영 드러나지 않습니다. 환경변수가 없으면 기동이 실패하는 편이 낫습니다.

서비스 저장소의 `${CONFIG_HOST:localhost}` 처럼 비밀이 아니면서 로컬 값이 정해져 있는 것은 예외이며, 그 판단은 1-2에 적었습니다.

### 5-2. 커밋한 값은 지워도 이력에 남습니다

되돌리려면 이력을 다시 써야 하고, 그 전에 저장소를 내려받은 사람에게는 그대로 남습니다. 비밀 값을 실수로 커밋했다면 되돌리는 것으로 끝나지 않으며 **해당 키를 새로 발급해야 합니다.**

저장소 설정에서 Secret Scanning과 Push Protection을 켜 두었습니다. 다만 카카오·AWS 토큰처럼 형식이 알려진 값만 걸러내며, 임의의 문자열로 된 비밀번호는 잡지 못합니다.

---

## 6. 파일 목록

파일은 모두 저장소 루트에 둡니다. 하위 디렉터리를 쓰려면 `config-server` 에 탐색 경로 설정이 추가로 필요하고, 잘못 지정하면 **오류 없이 파일을 찾지 못합니다.**

### 6-1. 공통

```
application.yml
application-local.yml
application-dev.yml
application-prod.yml
```

### 6-2. 플랫폼

```
eureka-server.yml
gateway-server.yml
```

`config-server` 는 자기 설정을 이 저장소에서 받지 못합니다. 저장소 주소를 알아야 저장소를 읽을 수 있기 때문입니다. `config-server` 의 설정은 그 저장소의 `application.yml` 과 환경변수에 둡니다.

### 6-3. 도메인

| 파일 | 포트 | 데이터베이스 |
|---|---|---|
| `auth-service.yml` | 8081 | `auth_db` |
| `user-service.yml` | 8082 | `user_db` |
| `pet-service.yml` | 8083 | `pet_db` |
| `place-service.yml` | 8084 | `place_db` |
| `policy-service.yml` | 8085 | `policy_db` |
| `verdict-service.yml` | 8086 | 없음 |
| `search-service.yml` | 8087 | `search_db` |
| `ingest-service.yml` | 8088 | `raw_db` |
| `extract-service.yml` | 8089 | 없음 |
| `congestion-service.yml` | 8090 | 없음 |
| `route-service.yml` | 8091 | 없음 |
| `report-service.yml` | 8092 | `report_db` |
| `review-service.yml` | 8094 | `review_db` |
| `notification-service.yml` | 8093 | `notif_db` |

---

## 7. 값을 바꾸면 언제 반영되는가

| 바꾼 것 | 필요한 작업 |
|---|---|
| 이 저장소의 값 | `main` 에 커밋하면 끝입니다. `config-server` 를 다시 띄우지 않아도 됩니다 |
| 이미 떠 있는 서비스에 반영 | 해당 서비스에 `POST /actuator/refresh` 또는 재기동 |
| `config-server` 자신의 설정 | 컨테이너 재시작 (이미지는 그대로) |

`config-server` 는 요청을 받을 때마다 저장소를 다시 읽습니다. 따라서 설정 변경이 곧 배포는 아닙니다.

---

## 8. 트러블슈팅

### 값을 바꿨는데 서비스에 반영되지 않습니다

커밋했는지 먼저 확인합니다. `config-server` 는 작업 디렉터리가 아니라 저장소를 읽으므로 커밋하지 않은 변경은 보이지 않습니다.

커밋했다면 서비스가 설정을 다시 읽지 않은 것입니다. `POST /actuator/refresh` 를 호출하거나 재기동합니다.

`config-server` 에서 실제로 내려가는 값은 아래로 확인합니다.

```bash
curl http://localhost:8888/auth-service/dev
```

### `{서비스명}.yml` 을 만들었는데 읽히지 않습니다

파일명이 그 서비스의 `spring.application.name` 과 정확히 같은지 확인합니다. 다르면 오류 없이 무시되고 상위 계층 값만 내려갑니다.

### 서비스가 기동할 때 설정을 받지 못합니다

`spring.config.import` 에 `optional:` 이 붙어 있는지 확인합니다. 없으면 `config-server` 가 떠 있지 않을 때 기동 자체가 실패합니다.

### 비밀 값을 실수로 커밋했습니다

되돌리는 것만으로는 부족합니다. 이력에 남아 있으므로 **해당 키나 비밀번호를 새로 발급해야 합니다.** 발급 후 환경변수를 교체하고, 이 저장소에는 `${환경변수}` 형태만 남았는지 다시 확인합니다.

---

## 9. 작업 규칙

1. 브랜치를 나누지 않고 `main` 에 직접 커밋합니다.
2. **메모장으로 편집하지 않습니다.** IntelliJ 또는 VS Code를 사용합니다. 줄바꿈이나 인코딩이 깨지면 값이 조용히 어긋나며 오류 메시지가 나오지 않습니다.
3. 값을 옮길 때는 **원래 있던 곳에서 지웁니다.** 같은 값이 서비스 저장소와 이 저장소에 모두 있으면 어느 쪽이 이기는지 매번 확인해야 합니다.
4. 값을 추가하면 4장의 기준으로 계층을 고릅니다.

---

## 10. 디렉터리 구조

```
config/
├── application.yml               모든 서비스 공통
├── application-local.yml         IntelliJ 실행
├── application-dev.yml           로컬 컨테이너
├── application-prod.yml          AWS
├── eureka-server.yml
├── gateway-server.yml
├── auth-service.yml              도메인 서비스 14개
├── ...
├── .gitattributes
├── .editorconfig
├── .gitignore
├── .coderabbit.yaml
└── .github/
    ├── ISSUE_TEMPLATE/issue_template.md
    └── pull_request_template.md
```
