# 친구톡 API

1. [친구톡 구성](#1-친구톡-구성)
2. [API 환경](#2-API-환경)
3. [API](#3-API)
4. [발송 API 타입별 참고 사항](#4-발송-API-타입별-참고-사항)
5. [WebHook](#5-WebHook)

## 참고 : 변경사항

| 일시       | 변경 내역                                                                                                                                                                                                                |
| ---------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2021.2.18  | `발송 API` 수정<br/>- ad_flag 추가<br/>- cate_cd 설명 수정                                                                                                                                                                |
| 2021.2.5   | `발송 API` 수정<br/>- message_group_id 추가<br/>- message_id 추가<br/>`WebHook` 추가                                                                                                                                      |
| 2021.1.    | 친구톡 API 초안 작성                                                                                                                                                                                                     |


## 1. 친구톡 구성

### 1.1 에이전트(Lunasoft)

파트너로부터 메시지 전송 권한을 위임받아서 카카오 유저에게 식별값을 통하여 알림 메시지 전달을 대행하는 사업체입니다.

### 1.2 파트너

유저에게 에이전트를 통해 알림 메시지를 발송할 수 있는 계정입니다.

### 1.3 유저

카카오톡에 가입한 사용자며 친구톡 알림 메시지를 수신할 대상입니다.

## 2. API 환경

친구톡 API 환경은 크게 2가지로 나뉘며, 친구톡 API를 사용하기 위해 파트너는 별도의 API키를 발급받아야 합니다. (`{api_key}` 발급)

- 운영 환경

  - 호스트 :
    - 이미지 API : `https://io-image.lunasoft.co.kr/`
    - 발송 API : `https://io-send.lunasoft.co.kr/`
  - 발급된 `{api_key}`를 이용하여 API를 호출합니다.
  - 전화번호의 실 사용자에게 메시지가 전송되기 때문에, 반드시 메시지 발송 대상의 전화번호에 유의해야 합니다.
  - 자세한 과금 기준과 조건은 제휴 담당자를 통해서 확인하세요.

- 개발 환경

  - **개발 환경은 한시적으로 제공되며, 테스트 종료 후에는 삭제될 수 있습니다.**
  - 호스트 :
    - 이미지 API : `https://test-io-image.lunasoft.co.kr/`
    - 발송 API : `https://test-io-send.lunasoft.co.kr/`
  - 발급된 `{api_key}`를 이용하여 API를 호출합니다.
  - 개발 환경에서 보내기 API 성공 건에 대해서는 과금되지 않으며, 앞으로도 과금되지 않습니다.
  - 친구톡 API의 기능 검증 용도로만 사용해야 합니다.
  - 안정적인 운영을 위해 10 TPS 이하 및 하루 1,000건 이하로 사용해야 합니다.
  - 추후 새로운 기능은 개발 환경에 먼저 배포되며, 피드백을 반영한 뒤에 운영 환경에 배포될 예정입니다.
  - **개발 환경에서는 테스트 카카오톡 채널 @clsrnxhrxptmxm (친구톡테스트) 로만 테스트 발송이 가능합니다.**

- 운영 환경과 개발 환경은 데이터를 공유하지 않습니다.

- **발송 제약 시간(20시 ~ 익일 08시) 존재합니다.**

- **친구톡 발송 결과 코드는 `WebHook` 를 통해서만 받을수 있습니다.** (`WebHook Url` 루나소프트에 사전 등록 필요)

API는 아래와 같이 나뉩니다.

- [이미지 API](#31-이미지-API)
- [발송 API](#32-발송-API)

## 3. API 

### 3.1 이미지 API

`이미지 API`는 메시지 발송 API에 사용될 이미지를 업로드 합니다.

- 이미지 API
  - 권장 사이즈 : 720px * 720px
  - 제한 사이즈 : 가로 500px 미만 또는 가로:세로 비율이 2:1 미만 또는 3:4 초과시 업로드 불가
  - 파일형식 및 크기 : jpg, png / 최대 500KB
- WIDE 이미지 API
  - 제한 사이즈 : 800px * 600px
  - 파일형식 및 크기 : jpg, png / 최대 2MB

#### Request

- path : 
  - 이미지 API : /api/Upload/Image
  - WIDE 이미지 API : /api/Upload/WideImage
- method : `POST`
- header :
  - Content-Type : multipart/form-data
- form

| 키             | 타입         | 필수 | 설명                       | 예제                                     |
| -------------- | ----------- | ---- | ------------------------- | ---------------------------------------- |
| member_id      | text(50)    | Y    | 고객 ID (파트너 ID)        | lunasoft                                 |
| api_key        | text(50)    | Y    | 발급받은 API키             | abcdefghijklmnopqrstuvwxyz               |
| image          | byte[]      | Y    | 이미지 파일 스트림          |                                          |

#### Response

- body :

```json
-- 이미지 업로드 성공시
{
  "code": "0000",
  "iamge_url": "http://mud-kage.kakao.com/dn/c0CM0k/btqQXBQxAJh/bkykLs2M2pOuDRQkL3byn0/img_l.jpg"
}

-- 오류 발생시 
{
  "code": "1099",
  "message": "member_id은(는) 필수값입니다. / api_key은(는) 필수값입니다."
}
```

| 키        | 타입 | 필수 | 설명                                          | 예제        |
| --------- | ---- | ---- | ------------------------------------------- | ----------- |
| image_url | text | N    | 업로드된 이미지 URL (업로드 성공 시 존재하는 값) |             |
| code      | text | Y    | 결과 코드                                    | 0000        |
| message   | text | N    | 오류 메시지 (오류 시 존재하는 값)               |             |


\[결과 코드-Code\]

| Code        | Message                                                                           |
| ----------- | --------------------------------------------------------------------------------- |
| 0000        | 성공                                                                              |
| 1001        | 유효하지 않은 Request Form                                                         |
| 1002        | 유효하지 않은 MemberId                                                             |
| 1003        | 유효하지 않은 ApiKey                                                               |
| 1004        | (ex) FailedToUploadImageException(InvalidImageSizeException- 발송 할 수 없는 이미지 크기입니다. <br/>: 가로:세로 비율은 2:1 이상 또는 3:4 이하여야 합니다 |
| 1099        | 잘못된 요청 파라미터. ex : member_id은(는) 필수값입니다. / api_key은(는) 필수값입니다. |
| 9999        | ServiceException (서비스에서 알 수 없는 문제가 발생)                                 |

### 3.2 발송 API

`발송 API`는 친구를 맺은 이용자 대상으로 고객사의 회원 전화번호 기반으로 광고성 메시지를 전송할 수 있습니다.

#### Request

- path : /api/FriendTalk
- method : `POST`
- header :
  - Content-Type : application/json
- body :

```json
{
  "member_id": "text",
  "api_key": "text",
  "cate_cd": number,
  "message_group_id": "text",
  "messages": [
    {
      "phone_number": "text",
      "app_user_id": "text",
      "message_id": "text",
      "message": "text",
      "ad_flag": "text",
      "wide": "text",
      "attachment": {
        "button": [
          {
            "name": "text",
            "type": "text",
            "url_mobile": "text",
            "url_pc": "text"
          }
        ],
        "image": {
          "img_url": "text",
          "img_link": "text"
        }
      }
    }
  ]
}
```
| 키                           | 타입         | 필수   | 설명                                                                                       | 예제                                  |
| ---------------------------- | ----------- | ------ | ------------------------------------------------------------------------------------------ | ------------------------------------ |
| member_id                    | text(50)    | Y      | 고객 ID (파트너 ID)                                                                         | lunasoft                             |
| api_key                      | text(50)    | Y      | 발급받은 API 키                                                                             |                                      |
| cate_cd                      | number      | N      | 카테고리 코드 (0: 일반 친구톡, 1: 데이타라이즈 도넛, 2: 데이타라이즈 스마트메시지)<br/>기본값 0        | 0                                    |
| message_group_id             | text(50)    | N      | 메세지에 대한 업체별 그룹 아이디 (*** `WebHook` 에서 전달 받음)                                  | message_group_id_1                    |
| messages[]                   | array(1000) | Y      | 메시지 목록. 최대 1,000개                                                                    |                                      |
| messages[].<br/>phone_number | text(16)    | -      | 사용자 전화번호 (*** phone_number 혹은 app_user_id 둘 중 하나는 반드시 있어야 하며, phone_number와 app_user_id의 정보가 동시에 요청된 경우 phone_number로만 발송) | 01012345678 |
| messages[].<br/>app_user_id  | text        | -      | 앱 유저 아이디                                                                              | 12345                                |
| messages[].<br/>message_id   | text(50)    | -      | 메세지에 대한 업체별 유니크 아이디 (*** `WebHook` 에서 필수값)                                   | message_id_1                         |
| messages[].<br/>message      | text(1000)  | Y      | 사용자에게 전달될 메시지<br/>(공백 포함 1,000자 제한)                                           |                                      |
| messages[].<br/>ad_flag      | text(1)     | N      | 광고성 메시지 표기 여부 (Y, N)<br/>기본값 Y                                                   | Y                                    |
| messages[].<br/>wide         | text(1)     | Y      | 와이드 이미지 사용 여부 (Y, N)<br/>텍스트만 보낼 경우 N                                         | N                                    |
| messages[].<br/>attachment   | object      | N      | 메시지 첨부 내용 (버튼 + 이미지)                                                              |                                      |
| attachment.<br/>button[]     | array(5)    | N      | 버튼 목록                                                                                  |                                      |
| button[].<br/>name           | text(28)    | Y      | 버튼 제목                                                                                  |                                      |
| button[].<br/>type           | text(2)     | Y      | 버튼 타입                                                                                  | WL(웹링크)<br/>AL(앱링크)              |
| button[].<br/>url_mobile     | text        | -      | mobile 환경에서 버튼 클릭 시 이동할 URL                                                       | https://lunasoft.co.kr               |
| button[].<br/>url_pc         | text        | -      | pc 환경에서 버튼 클릭 시 이동할 URL                                                           | https://lunasoft.co.kr               |
| button[].<br/>scheme_android | text        | -      | mobile android 환경에서 버튼 클릭 시 실행 할<br/>application custom scheme                    | "scheme://xxx.xxx"                   |
| button[].<br/>scheme_ios     | text        | -      | mobile ios 환경에서 버튼 클릭 시 실행 할<br/>application custom scheme                        | "scheme://xxx.xxx"                   |
| attachment.<br/>image        | object      | N      | 노출할 이미지 정보 (*** 와이드 이미지 타입의 경우 [텍스트 메시지(76자 제한) + 링크 버튼(1개) + 이미지] 발송 가능) |                             |
| image.<br/>img_url           | text        | Y      | 이미지 API를 통해 업로드된 결과 이미지 URL                                                     |                                       |
| image.<br/>img_link          | text        | N      | 이미지 클릭시 이동할 URL. 미설정 시 카카오톡 내 이미지 뷰어 사용                                   |                                       |

##### 버튼 타입 별 속성

- 필수 파라미터를 모두 입력하셔야 정상적인 발송이 가능합니다.
- 버튼타입이 `AL`일 경우 scheme_ios, scheme_android, url_mobile 중 2가지를 필수로 입력해야 발송이 가능합니다.

| 버튼타입<br/>(button[].type) | 속성           | 타입      | 필수  | 설명                                              | 예제                                       |
| --------------------------- | -------------- | ---------| ---- | ------------------------------------------------- | ---------------------------------------- |
| WL                          | url_mobile     | text     | Y    | 버튼 클릭 시 이동할 mobile web URL                  | https://lunasoft.co.kr                   |
|                             | url_pc         | text     | N    | 버튼 클릭 시 이동할 pc web URL                      | https://lunasoft.co.kr                   |
| AL                          | scheme_android | text     | -    | mobile android 환경에서 버튼 클릭 시 실행 할 application custom scheme   | "scheme://xxx.xxx"    |
|                             | scheme_ios     | text     | -    | mobile ios 환경에서 버튼 클릭 시 실행 할 application custom scheme   | "scheme://xxx.xxx"        |
|                             | url_mobile     | text     | -    | mobile 환경에서 버튼 클릭 시 이동 할 URL              | https://lunasoft.co.kr                   |
|                             | url_pc         | text     | N    | pc 환경에서 버튼 클릭 시 이동 할 URL                  | https://lunasoft.co.kr                   |

#### Response

- body :

```json
-- 정상적으로 발송 요청 성공 시 Http Status 200 및 body 내용 없음

-- 오류 발생 시 
{
  "code": "1099",
  "message": "member_id은(는) 필수값입니다. / api_key은(는) 필수값입니다."
}
```

| 키             | 타입         | 필수  | 설명                             |
| -------------- | ----------  | ----  | ------------------------------ |
| code           | text        | Y     | 오류 코드                       | 
| message        | text        | Y     | 오류 메시지 (오류 시 존재하는 값)   |


\[오류 코드-Code\]

| Code        | Message                                                                           |
| ----------- | --------------------------------------------------------------------------------- |
| 1002        | 유효하지 않은 MemberId                                                              |
| 1003        | 유효하지 않은 ApiKey                                                                |
| 1099        | 잘못된 요청 파라미터. ex : member_id은(는) 필수값입니다. / api_key은(는) 필수값입니다.     |
| 1100        | 친구톡 발송 제약 시간입니다. (20시 ~ 익일 08시)                                      |
| 9999        | 알수없는 예외가 발생했습니다. 관리자에게 문의해주세요.                                     |

## 4. 발송 API 타입별 참고 사항

**발송 제약 시간(20시 ~ 익일 08시)이 존재합니다.**

### 4.1 텍스트 타입

```json
{
  "member_id": "lunasoft",
  "api_key": "abcdefghijklmnopqrstuvwxyz",
  "cate_cd": 1,
  "message_group_id": "message_group_id_text1",
  "messages": [
    {
      "phone_number": "01012341234",
      "message_id": "message_id_text1",
      "message": "친구톡 발송 (텍스트) - TEST",
      "ad_flag": "Y",
      "wide": "N",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 2",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 3",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 4",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 5",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ]
      }
    },
    {
      "phone_number": "01012341234",
      "message_id": "message_id_text2",
      "message": "친구톡 발송 (텍스트) - TEST",
      "ad_flag": "Y",
      "wide": "N",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ]
      }
    }
  ]
}
```
- 텍스트 타입의 경우 텍스트 메시지 + 링크 버튼(5개) 발송이 가능합니다.
- 텍스트 문구(messages[].message)는 1,000자로 제한됩니다.

### 4.2 이미지 타입

```json
{
  "member_id": "lunasoft",
  "api_key": "abcdefghijklmnopqrstuvwxyz",
  "cate_cd": 1,
  "message_group_id": "message_group_id_image1",
  "messages": [
    {
      "phone_number": "01012341234",
      "message_id": "message_id_image1",
      "message": "친구톡 발송 (이미지) - TEST",
      "ad_flag": "Y",
      "wide": "N",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 2",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 3",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 4",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          },
          {
            "name": "웹링크 버튼 - 5",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ],
        "image": {
            "img_url": "http://mud-kage.kakao.com/dn/c0CM0k/btqQXBQxAJh/bkykLs2M2pOuDRQkL3byn0/img_l.jpg",
            "img_link": "http://bizmessage.kakao.com"
        }
      }
    },
    {
      "phone_number": "01012341234",
      "message_id": "message_id_image2",
      "message": "친구톡 발송 (이미지) - TEST",
      "ad_flag": "Y",
      "wide": "N",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ],
        "image": {
            "img_url": "http://mud-kage.kakao.com/dn/c0CM0k/btqQXBQxAJh/bkykLs2M2pOuDRQkL3byn0/img_l.jpg",
            "img_link": "http://bizmessage.kakao.com"
        }
      }
    }
  ]
}
```
- 기본적으로 텍스트 타입과 동일하나 attachment 필드에 이미지를 추가하여 발송 가능합니다.
- 반드시 이미지 API를 통해 업로드한 이미지의 URL을 사용해야 합니다.
- messages[].wide를 'N'으로 설정합니다.
- 이미지 타입의 경우 텍스트 메시지 + 링크 버튼(5개) + 이미지 발송이 가능합니다.
- 텍스트 문구(messages[].message)는 400자로 제한됩니다.

### 4.3 WIDE 이미지 타입

```json
{
  "member_id": "lunasoft",
  "api_key": "abcdefghijklmnopqrstuvwxyz",
  "cate_cd": 1,
  "message_group_id": "message_group_id_wideimage1",
  "messages": [
    {
      "phone_number": "01012341234",
      "message_id": "message_id_wideimage1",
      "message": "친구톡 발송 (WIDE 이미지) - TEST",
      "ad_flag": "Y",
      "wide": "Y",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ],
        "image": {
            "img_url": "http://mud-kage.kakao.com/dn/bNu9jZ/btqQ7U2ahPx/MY1qouk5tl62n7IkipmcCK/img_l.jpg",
            "img_link": "http://bizmessage.kakao.com"
        }
      }
    },
    {
      "phone_number": "01012341234",
      "message_id": "message_id_wideimage2",
      "message": "친구톡 발송 (WIDE 이미지) - TEST",
      "ad_flag": "Y",
      "wide": "Y",
      "attachment": {
        "button": [
          {
            "name": "웹링크 버튼 - 1",
            "type": "WL",
            "url_pc": "https://lunasoft.co.kr",
            "url_mobile": "https://lunasoft.co.kr"
          }
        ],
        "image": {
            "img_url": "http://mud-kage.kakao.com/dn/bNu9jZ/btqQ7U2ahPx/MY1qouk5tl62n7IkipmcCK/img_l.jpg",
            "img_link": "http://bizmessage.kakao.com"
        }
      }
    }
  ]
}
```
- 기본적으로 텍스트 타입과 동일하나 attachment 필드에 이미지를 추가하여 발송 가능합니다.
- 반드시 이미지 API를 통해 업로드한 이미지의 URL을 사용해야 합니다.
- messages[].wide를 'Y'로 설정합니다.
- 이미지 타입의 경우 텍스트 메시지 + 링크 버튼(1개) + 이미지 발송이 가능합니다.
- 텍스트 문구(messages[].message)는 76자로 제한됩니다.

## 5. WebHook 

- `WebHook Url`을 루나소프트에 사전 등록을 해주셔야 합니다.
- **`발송 API` messages[].message_id 를 필수로 설정해야 WebHook으로 친구톡 발송 결과를 받을수 있습니다.**

#### Request

- path : `루나소프트에 사전 등록된 WebHook Url`
- method : `POST`
- header :
  - Content-Type : application/json
- body :

```json
{
  "type": "text",  
  "message_group_id": "text",
  "message_id": "text",
  "code": "text",
  "response_dt": "text"
}
```
| 키                | 타입         | 필수 | 설명                                                   | 예제                                    |
| ----------------- | ----------- | ---- | ---------------------------------------------------- | ---------------------------------------- |
| type              | text(20)    | Y    | 메세지 타입 (friendtalk 고정)                          | friendtalk                              |
| message_group_id  | text(50)    | N    | 메세지에 대한 업체별 그룹 아이디<br/>(*** `발송 API` 에서 설정)  | message_group_id_1              |
| message_id        | text(50)    | Y    | 메세지에 대한 업체별 유니크 아이디<br/>(*** `발송 API` 에서 설정) | message_id_1                   |
| code              | text        | Y    | 친구톡 발송 결과 코드                                  | 1000                                     |
| response_dt       | datetime    | Y    | 친구톡 API 발송 응답 시간                              | 2021-02-05 09:05:30                      |

- Sample
```json
{
  "type": "friendtalk",
  "message_group_id": "message_group_id_1",
  "message_id": "message_id_1",
  "code": "1000",
  "response_dt": "2021-02-05 09:05:30"
}
```

#### 친구톡 발송 결과 코드-Code

| Code    | Message                                                                                                                       |
| ------- | ------------------------------------------------------------------------------------------------------------------------------|
| 1000    | 성공                                                                                                                           |
| 1013    | 유효하지 않은 app연결                                                                                                             |
| 1015    | 유효하지 않은 app user id 요청                                                                                                    |
| 1020    | 전화번호 or app user id가 유효하지 않거나 미입력 요청                                                                                |
| 1030    | 잘못된 파라미터 요청                                                                                                              |
| 2000    | 전송 시간 초과                                                                                                                   |
| 2001    | 예기치 않은 오류 발생 (카카오 시스템)                                                                                               |
| 3022    | 메시지 발송 가능한 시간이 아님 (친구톡 / 마케팅 메시지는 08시부터 20시까지 발송 가능)                                                      |
| 3023    | Request Body가 Json형식이 아님                                                                                                   |
| 3024    | 발신 프로필 키가 유효하지 않음                                                                                                     |
| 3027    | 전화번호 오류 (카카오톡을 사용하지 않는 사용자 / 050안심번호)                                                                           |
| 3031    | 메시지가 비어 있음                                                                                                                |
| 3032    | 메시지 길이 제한 오류<br/>(텍스트 타입 1000자 초과, 이미지 타입 400자 초과, 와이드 이미지 타입 76자 초과)                                   |
| 3046    | 닫힘 상태의 카카오톡 채널                                                                                                          |
| 3049    | 내부 시스템 오류로 메시지 전송 실패 (카카오 시스템)                                                                                    |
| 3050    | 메시지를 전송할 수 없음<br/>- 카카오톡을 사용하지 않는 사용자<br/>- 친구톡의 경우 친구가 아닌 경우<br/>- 카카오톡을 당일 설치 했거나 일주일 동안 사용을 안한 유저의 경우<br/>- 구버전의 카카오톡을 사용하는 경우 |
| 3063    | btn_url오류 (http:// 또는 https:// 입력하지 않음)                                                                                 |
| 4100    | 메시지에 포함된 이미지를 전송할 수 없음 (이미지주소 또는 링크가 올바르지 않거나 이미지가 규격에 맞지 않음)                                     |
| 9101    | HttpRequestException 발생 시(SSL 문제로 인해 코드 추가) (루나 시스템)                                                                |
| 9102    | Gateway timeout (504) 발생 - (루나 시스템)                                                                                       |
| 9103    | Connected host has failed to response (카카오 서버의 연결이 많을 경우 발생) - (루나 시스템)                                           |
| 9501<br/>9502<br/>9503<br/>9504<br/>9521<br/>9522<br/>9541 | 루나 발송 파이프라인 시스템 오류로 메시지 전송 실패 (루나 시스템)                     |
| 9997<br/>9998<br/>9999 | 시스템에 문제가 발생하여 담당자가 확인하고 있는 경우 (카카오 시스템)                                                       |
