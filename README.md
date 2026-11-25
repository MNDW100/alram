# 가변저항 값이 800을 넘으면 IFTTT 알림 받기

ThingSpeak 채널의 `Field 1`에 들어오는 가변저항 값이 `800`보다 커지면 IFTTT Webhook을 호출하고, 휴대폰 IFTTT 앱으로 알림을 보내는 설정입니다.

**동작 흐름:** 가변저항 → ThingSpeak `Field 1` → React 조건 검사 → ThingHTTP → IFTTT Webhook → 휴대폰 알림

## 설정 요약

| 항목 | 설정 |
|---|---|
| ThingSpeak 채널 | `3565693` |
| 측정 필드 | `Field 1` (가변저항 값) |
| React 조건 | Numeric, `Field 1 is greater than 800` |
| 검사 시점 | On Data Insertion |
| IFTTT 이벤트 | `TooCold` (IFTTT와 ThingHTTP에서 똑같이 사용) |
| 알림 예시 | 가변저항 값이 800을 넘었습니다. 현재 값: `{{Value1}}` |

## 1. IFTTT Applet 만들기

1. IFTTT에서 **Create**를 눌러 새 Applet을 만듭니다.
2. **If This → Webhooks → Receive a web request with a JSON payload**를 선택합니다.
3. Event Name에 `TooCold`를 입력합니다.
4. **Then That → Notifications → Send a notification from the IFTTT app**를 선택합니다.
5. 알림 문구를 입력하고 Applet을 저장합니다. 휴대폰 IFTTT 앱에도 같은 계정으로 로그인하고 알림을 허용합니다.

## 2. ThingHTTP에서 Webhook 호출 설정

ThingSpeak의 **Apps → ThingHTTP → New ThingHTTP**에서 다음과 같이 저장합니다.

- **Name:** IFTTT Pot Alert
- **URL:** `https://maker.ifttt.com/trigger/TooCold/json/with/key/본인의_WEBHOOK_KEY`
- **Method:** POST
- **Content Type:** `application/json`
- **Body:**

```json
{ "value1":"%%channel_3565693_field_1%%" }
```

Webhook URL의 `본인의_WEBHOOK_KEY`는 IFTTT Webhooks의 Documentation에서 확인한 개인 키로 바꿉니다. 이 키가 포함된 실제 URL은 공개 저장소나 스크린샷에 올리지 않습니다.

## 3. React 조건 등록

ThingSpeak의 **Apps → React → New React**에서 설정합니다.

- **Condition Type:** Numeric
- **Test Frequency:** On Data Insertion
- **If channel:** 본인의 가변저항 채널 (`3565693`)
- **Field:** Field 1
- **조건:** is greater than `800`
- **Action:** ThingHTTP → `IFTTT Pot Alert`
- **Options:** 테스트 중 매번 알림을 확인하려면 `Run action each time condition is met` 선택

매번 실행하도록 두면 800을 넘는 데이터가 기록될 때마다 알림이 옵니다. 알림을 한 번만 받고 싶다면 `Run action only the first time the condition is met`을 선택합니다. 다시 시험할 때는 먼저 800 이하의 값을 기록합니다.

## 4. Field 1 기록 및 알림 시험

ThingSpeak의 **Channels → API Keys**에서 Write API Key를 확인한 뒤, 아래 주소에서 `WRITE_API_KEY`를 본인 키로 바꾸고 브라우저에서 엽니다.

```text
https://api.thingspeak.com/update?api_key=WRITE_API_KEY&field1=801
```

응답으로 0보다 큰 기록 번호가 나오면 채널에 값이 기록된 것입니다. `801`은 `800` 초과 조건을 만족합니다. 무료 계정은 기록 사이에 최소 15초 간격을 둡니다. 시험이 끝나면 휴대폰에서 IFTTT 알림을 확인합니다.

## 확인할 점

알림이 오지 않으면 다음 순서로 확인합니다.

1. 채널 피드에 Field 1 값이 기록됐는지 확인합니다.
2. React의 채널, Field 1, `greater than 800` 조건과 실행 옵션을 확인합니다.
3. ThingHTTP URL의 이벤트명이 `TooCold`인지, JSON Body의 채널 ID와 Field 1 참조에 오타나 공백이 없는지 확인합니다.
4. IFTTT Activity에서 Webhook 트리거와 Notifications 액션 실행 기록을 확인합니다.
5. 휴대폰 IFTTT 앱의 로그인 계정과 알림 권한을 확인합니다.

**보안:** Write API Key와 Webhook Key는 공개하지 않습니다. 이 문서에는 실제 키를 포함하지 않았습니다.
