# OpenNCCU OAuth

---

# 使用方法

## 步驟 1：設定參數

### 參數

- client_id:
  應用程式的用戶端 ID。
- redirect_uri:
  決定 API 伺服器在使用者完成授權流程後將使用者重新導向的位置。
- response_type: `code`
- scope: 多個 scope 用`,`分隔
  - profile: "個人資料",
  - dept: "系所、入學年",
  - email: "電子郵件",
  - inccu: "iNCCU 全人資訊",
  - inccu_basic_profile: "iNCCU 個人資料 (學號)",
  - inccu_school_roll: "iNCCU 學籍資料",
  - inccu_course_score: "iNCCU 修業成績",
  - inccu_course_record: "iNCCU 修課紀錄、成績",
  - inccu_capabillity: "iNCCU 校內活動、能力展現",
  - inccu_healthy: "iNCCU 全人健康資訊",
  - inccu_honors: "iNCCU 全人榮譽資料（操行成績等）",
  - inccu_resume: "iNCCU 全人履歷資料",

### 建議使用

- state: 指定應用程式用來維持授權要求和授權伺服器回應狀態的任何字串值。

## 步驟 2：重新導向至 OpenNCCU 的 OAuth 2.0 伺服器

```
[xxx]表示這是一個變數，需要自行填入

https://oauth.opennccu.com/authorize?
scope=email,profile,dept&
response_type=code&
state=[state_parameter_passthrough_value]&
redirect_uri=[https%3A//oauth2.example.com/code]&
client_id=[client_id]
```

## 步驟 3：OpenNCCU 提示使用者提供同意聲明

在這個步驟中，使用者可以決定是否要授予應用程式要求的存取權。

## 步驟 4：處理 OAuth 伺服器回應

OAuth 伺服器會使用要求中指定的網址，回應應用程式的存取要求。

如果使用者核准存取要求，回應就會包含授權碼。如果使用者未核准要求，回應會包含錯誤訊息。傳回到網路伺服器的授權碼或錯誤訊息會顯示在查詢字串中，如下所示：

錯誤回應：

```
https://oauth2.example.com/code?
error=access_denied&
```

授權碼回應：

```
https://oauth2.example.com/auth?code=869e308c64a03f3e0e2d81db510f893c6ad3d063c8e90dafef7742ae30e87b27
```

完成 OAuth 流程後，系統應會將您重新導向至 http://localhost/oauth2callback

下一步是在使用者將使用者重新導向至應用程式時，傳回 URI 中傳回的資訊。

## 步驟 5：交換 refresh_token 和 access_token

請呼叫 https://oauth.opennccu.com/token 並帶上以下

- Content-Type: `application/x-www-form-urlencoded`
- body
  - client_id:
    應用程式的用戶端 ID。
  - client_secret:
    應用程式的用戶端密碼。
  - code:
    步驟 4 中取得的授權碼。
  - grant_type:
    `authorization_code`
  - redirect_uri:
    先前步驟中使用的網址。

範例：

```
[xxx]表示這是一個變數，需要自行填入
POST /token HTTP/1.1
Host: oauth.opennccu.com
Content-Type: application/x-www-form-urlencoded

code=[先前回傳的code]&
client_id=[your_client_id]&
client_secret=[your_client_secret]&
redirect_uri=[https%3A//oauth2.example.com/code]&
grant_type=authorization_code
```

回傳的 JSON 會包含下列資訊：

- access_token: 用來取得資料的 token，有效期限 1 小時
- expires_in: 有效期限，單位秒
- refresh_token: 用來更新 `access_token` 的 token，有效期限 30 天
- scope: 權限，以空白分隔
- token_type: `Bearer`

範例：

```json
{
  "access_token": "54f91e3551c7c88fc1c9461ecd27267686ad9e19a7d52a784d51d41f2cc143b9",
  "expires_in": 3600,
  "token_type": "Bearer",
  "scope": "profile dept",
  "refresh_token": "be0545b14ede593cfe30ea4a827cd53a9774bff620f8aed580eec30a48c9703b"
}
```

---

# 使用 access_token 取得資料

請呼叫 https://oauth.opennccu.com/data 並帶上以下

範例：

```
[xxx]表示這是一個變數，需要自行填入
GET /data/ HTTP/1.1
Host: oauth.opennccu.com
Authorization: Bearer [您的 access_token]
```

如果授權被用戶在 Open NCCU 取消，該個 key 會寫`"unauthorized"`
例如

```json
{
  "email": "xxxxx@gmail.com",
  "inccu": "unauthorized"
}
```

---

# 使用 refresh_token 更新 access_token

請呼叫 https://oauth.opennccu.com/token 並帶上以下

- Content-Type: `application/x-www-form-urlencoded`
- body
  - client_id:
    應用程式的用戶端 ID。
  - client_secret:
    應用程式的用戶端密碼。
  - code:
    步驟 4 中取得的授權碼。
  - grant_type:
    `authorization_code`
  - redirect_uri:
    先前步驟中使用的網址。

範例：

```
[xxx]表示這是一個變數，需要自行填入
POST /token HTTP/1.1
Host: oauth.opennccu.com
Content-Type: application/x-www-form-urlencoded

client_id=[your_client_id]&
client_secret=[your_client_secret]&
refresh_token=[refresh_token]&
grant_type=refresh_token
```

回傳的 JSON 會包含下列資訊：

- access_token: 用來取得資料的 token，有效期限 1 小時
- expires_in: 有效期限，單位秒
- scope: 權限，以空白分隔
- token_type: `Bearer`

範例：

```json
{
  "access_token": "54f91e3551c7c88fc1c9461ecd27267686ad9e19a7d52a784d51d41f2cc143b9",
  "expires_in": 3600,
  "token_type": "Bearer",
  "scope": "profile dept"
}
```

---

# 其他附註

1. auth_code 只能使用一次，使用後會失效
2. refresh_token 只能使用一次，使用後會失效
3. access_token 有效期限為 1 小時，過期後請使用 refresh_token 更新
4. refresh_token 有效期限為 30 天，每次更新 access_token 都會更新 refresh_token 的有效期限
