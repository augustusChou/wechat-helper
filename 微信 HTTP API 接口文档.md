# 微信 HTTP API 接口文档

> 微信小助手提供的 HTTP 服务，用于通过微信机器人发送消息。
> 默认监听端口：`19088`（可在 `weixin_config.txt` 中通过 `http_api_port` 修改）。

---

## 一、通用说明

### 基础地址
```
http://<主机>:19088
```
- 本机调用：`http://127.0.0.1:19088`
- 局域网调用：`http://<局域网IP>:19088`

### 请求格式
- 所有接口均为 **POST** + **JSON** 请求体；
- 请求头：`Content-Type: application/json`。

### 响应格式（统一 JSON）
```json
{
  "code": 0,        // 0=成功，非0=失败
  "msg": "success"  // 提示信息
}
```

### 返回码说明

| code | 含义 |
|------|------|
| 0    | 成功 |
| 1    | 请求体必须是 JSON |
| 2    | 缺少必要参数（或参数不合法） |
| 3    | 微信发送/操作失败（详见 msg 中的错误码） |
| 4    | 服务器内部异常 |
| 500  | HTTP 状态码，配合 code 3/4 使用 |

---

## 二、接口列表

### 1. 发送文本消息

- **URL**：`/api/sendTextMsg`
- **方法**：`POST`

**请求体**：
```json
{
  "wxid": "目标wxid",
  "msg": "要发送的文本内容"
}
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| wxid | string | 是 | 接收者 wxid。个人为 `wxid_xxx`；群为 `xxx@chatroom`；文件传输助手为 `filehelper` |
| msg  | string | 是 | 要发送的文本内容 |

**响应示例**：
```json
{"code": 0, "msg": "success"}
```

**失败诊断**：`code: 3` 时，`msg` 会附带诊断信息（登录状态、PID 等），例如：
```json
{"code": 3, "msg": "发送失败，错误码 1，登录状态=1，PID=12345，PID存活=True"}
```

---

### 2. 发送图片消息（本地路径）

- **URL**：`/api/sendImagesMsg`
- **方法**：`POST`

**请求体**：
```json
{
  "wxid": "目标wxid",
  "imagePath": "C:\\path\\to\\image.jpg"
}
```

**参数说明**：

| 参数       | 类型   | 必填 | 说明                 |
|------------|--------|------|----------------------|
| wxid       | string | 是   | 接收者 wxid          |
| imagePath  | string | 是   | 本地图片的绝对路径   |

**注意**：`imagePath` 必须是服务端（运行 `微信小助手` 的机器）能访问到的本地路径，且文件必须存在。

**响应示例**：
```json
{"code": 0, "msg": "success"}
```

---

### 3. 转发消息

- **URL**：`/api/forwardMsg`
- **方法**：`POST`

**请求体**：
```json
{
  "wxid": "目标wxid",
  "msgId": 1234567890
}
```

**参数说明**：

| 参数  | 类型   | 必填 | 说明                       |
|-------|--------|------|----------------------------|
| wxid  | string | 是   | 接收者 wxid                |
| msgId | int    | 是   | 要转发的消息 ID（从回调获取） |

**响应示例**：
```json
{"code": 0, "msg": "success"}
```


---

## 三、调用示例

### curl
```bash
# 发送文本
curl -X POST http://127.0.0.1:19088/api/sendTextMsg \
  -H "Content-Type: application/json" \
  -d '{"wxid": "filehelper", "msg": "hello"}'

# 发送图片
curl -X POST http://127.0.0.1:19088/api/sendImagesMsg \
  -H "Content-Type: application/json" \
  -d '{"wxid": "filehelper", "imagePath": "C:\\img\\a.jpg"}'

# 转发消息
curl -X POST http://127.0.0.1:19088/api/forwardMsg \
  -H "Content-Type: application/json" \
  -d '{"wxid": "filehelper", "msgId": 1234567890}'
 
```

### Python
```python
import requests

# 发送文本
r = requests.post(
    "http://127.0.0.1:19088/api/sendTextMsg",
    json={"wxid": "filehelper", "msg": "hello"},
    timeout=10,
)
print(r.json())  # {"code": 0, "msg": "success"}
```

---

## 四、常见错误处理

| 现象 | 原因 | 处理 |
|------|------|------|
| code:3, 错误码 1 | 微信未登录 / PID 失效 / 注入失败 | 确认微信已登录、注入成功；查看诊断信息 |
| code:4, 未找到微信进程 | 微信未运行 | 启动微信后再调用 |
| 连接拒绝（WinError 10061） | 服务未启动或端口不对 | 确认服务已启动、端口为 http_api_port |
| 返回 500 HTML | 服务端未处理异常 | 查看服务日志（已配置全局错误处理，返回 JSON） |

---

## 五、相关配置

见 `weixin_config.txt`：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| http_api_port | 19088 | HTTP API 服务端口 |
| notify_wxid | filehelper | 启动通知目标 |
| login_success_msg | 来了 | 启动时发送的通知内容 |
