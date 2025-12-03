# 合作伙伴 API 对接文档

## 1. 概述

本文档详细描述了合作伙伴对接平台的接口规范，包括授权流程、接口定义、参数说明、安全规范等内容。系统提供了完整的用户授权与支付流程，支持第三方应用接入和调用。

### 1.1 功能特性

- **用户授权**：完整的 OAuth2.0 授权流程，保证用户信息安全
- **支付服务**：支持创建订单、查询订单状态、退款等完整支付流程
- **账户管理**：提供余额查询、提现申请等账户管理功能
- **安全保障**：采用签名验证、时间戳防重放等多重安全机制
- **实时通知**：支持异步回调通知，确保数据实时同步

### 1.2 接入流程

1. **申请合作伙伴资格**：联系我们申请成为合作伙伴
2. **获取 API 密钥**：获得 apiKey 和 appSecret
3. **配置回调地址**：配置支付通知、提现通知等回调地址
4. **接口开发**：根据本文档进行接口开发
5. **联调测试**：在测试环境进行接口联调
6. **上线发布**：通过测试后切换到生产环境

### 1.3 环境配置

#### 测试环境

- **域名**：`http://merchantapitest.nexaexworth.com/`
- **特点**：用于开发测试，数据定期清理
- **限制**：接口调用频率限制较宽松

#### 生产环境

- **域名**：`https://merchantapi.nexaexworth.com/`
- **特点**：正式运营环境，数据真实有效
- **限制**：严格的接口调用频率限制和安全检查

## 2. 授权流程

### 2.1 流程图

```
┌───────────┐          ┌───────────┐          ┌───────────┐          ┌───────────┐
│           │          │           │          │           │          │           │
│  第三方H5  │          │ 系统APP   │          │ 系统服务端  │          │ 第三方服务器 │
│           │          │           │          │           │          │           │
└─────┬─────┘          └─────┬─────┘          └─────┬─────┘          └─────┬─────┘
      │                      │                      │                      │
      │   1.唤起APP授权       │                      │                      │
      │─────────────────────>│                      │                      │
      │                      │                      │                      │
      │                      │  2.登录用户确认授权    │                      │
      │                      │                      │                      │
      │                      │  3.请求授权码         │                      │
      │                      │─────────────────────>│                      │
      │                      │                      │                      │
      │                      │  4.返回授权码         │                      │
      │                      │<─────────────────────│                      │
      │                      │                      │                      │
      │   5.返回授权码(code)  │                      │                      │
      │<─────────────────────│                      │                      │
      │                      │                      │                      │
      │   6.将授权码发送给服务器                      │                      │
      │──────────────────────────────────────────────────────────────────>│
      │                      │                      │                      │
      │                      │                      │  7.请求access_token   │
      │                      │                      │<─────────────────────│
      │                      │                      │                      │
      │                      │                      │  8.返回access_token   │
      │                      │                      │─────────────────────>│
      │                      │                      │                      │
      │   9.返回登录成功      │                      │                      │
      │<──────────────────────────────────────────────────────────────────│
      │                      │                      │                      │
```

### 2.2 流程说明

1. 第三方 H5 唤起系统 APP 的授权页面
2. 用户在系统 APP 中确认授权
3. 系统 APP 向系统服务端请求授权码
4. 系统服务端生成并返回授权码
5. 系统 APP 将授权码返回给第三方 H5
6. 第三方 H5 将授权码发送给第三方服务器
7. 第三方服务器向系统服务端请求 access_token
8. 系统服务端验证授权码并返回 access_token
9. 第三方服务器完成授权，返回登录成功

## 3. API 接口

### 3.1 获取授权码

- **接口说明**：由系统 APP 调用，用于获取用户授权后的授权码
- **接口路径**：`POST /partner/api/partner/user/auth/code`
- **请求参数**：

nexaauth这个是自定义协议webView如果检测到会做处理authorize是授权类型，会调起app授权逻辑
nexaauth://oauth/authorize?apikey=NEXA1925092532172697601&redirect_uri=https://nexa.tg.poker
这个是进入页面时候 唤起app授权


```json
{
  "apiKey": "第三方应用的API Key",
  "nonce": "随机字符串，防重放",
  "timestamp": "当前时间戳，毫秒级",
  "sign": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": "授权码，有效期5分钟，只能使用一次"
}
```

### 3.2 获取访问令牌

- **接口说明**：由第三方服务器调用，使用授权码获取访问令牌
- **接口路径**：`POST /partner/api/openapi/access_token/auth`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "code": "通过用户授权获取的授权码",
  "nonce": "随机字符串，防重放",
  "timestamp": "当前时间戳，毫秒级",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "session_key": "会话密钥，用于API调用的凭证",
    "expires_in": 7200
  }
}
```

### 3.3 刷新访问令牌

- **接口说明**：由第三方服务器调用，使用刷新令牌获取新的访问令牌
- **接口路径**：`POST /partner/api/openapi/access_token/refresh`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "sessionKey": "当前会话密钥",
  "nonce": "随机字符串，防重放",
  "timestamp": "当前时间戳，毫秒级",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "session_key": "新的会话密钥",
    "expires_in": 7200
  }
}
```

### 3.4 获取用户信息

- **接口说明**：使用访问令牌获取当前用户信息
- **接口路径**：`POST /partner/api/openapi/user/info`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "sessionKey": "会话密钥",
  "nonce": "随机字符串，防重放",
  "timestamp": "当前时间戳，毫秒级",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "nickname": "用户昵称",
    "avatar": "头像URL",
    "openid": "用户在此合作伙伴的唯一标识",
    "binding_time": "绑定时间"
  }
}
```

### 3.5 创建支付订单

- **接口说明**：创建支付订单，用于发起支付流程
- **接口路径**：`POST /partner/api/openapi/payment/create`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "orderNo": "合作伙伴订单号，确保唯一性",
  "openid": "用户在此合作伙伴的唯一标识",
  "amount": "支付金额，单位为元，保留两位小数",
  "currency": "货币类型，默认USDT",
  "callbackUrl": "支付结果页面回调地址",
  "notifyUrl": "支付结果异步通知地址",
  "returnUrl": "支付完成后的跳转地址",
  "description": "订单描述信息",
  "subject": "商品信息",
  "timestamp": "当前时间戳，毫秒级",
  "nonce": "随机字符串，防重放",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "timestamp": "当前时间戳，毫秒级",
    "nonce": "随机字符串，防重放",
    "signType": "MD5",
    "paySign": "支付签名，用于验证支付请求的合法性",
    "apiKey": "合作伙伴的API Key",
    "orderNo": "系统生成的唯一订单号"
  }
}
```

nexaauth这个是自定义协议webView如果检测到会取做处理order是类型就是支付，调起支付页面
创建支付订单之后 返回的参数 拼接协议就能唤起支付页面：
nexaauth://order?orderNo=T1991048326502010882&paySign=0076fa6482c2f17b240405381e0693c7&signType=MD5&apiKey=NEXA1925092532172697601&nonce=d0fdaf6e76154f1b8ca63b7febd15e06&timestamp=1763537887&redirectUrl=https://nexa.tg.poker



### 3.6 获取支付订单详情

- **接口说明**：查询支付订单的详细信息
- **接口路径**：`POST /partner/api/openapi/payment/query`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "orderNo": "订单号",
  "timestamp": "当前时间戳，毫秒级",
  "nonce": "随机字符串，防重放",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "orderNo": "订单号",
    "amount": "支付金额",
    "currency": "货币类型",
    "status": "订单状态",
    "createTime": "创建时间",
    "payTime": "支付时间，仅支付成功时返回"
  }
}
```

### 3.7 申请提现

- **接口说明**：向负责人账户申请提现
- **接口路径**：`POST /partner/api/openapi/account/withdraw`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "amount": "提现金额，单位为元",
  "currency": "货币类型，如：USDT",
  "userId": "用户ID",
  "notifyUrl": "提现结果通知地址",
  "remark": "提现备注说明（可选）",
  "timestamp": "当前时间戳，毫秒级",
  "nonce": "随机字符串，防重放",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "withdrawalNo": "提现单号",
    "amount": "提现金额",
    "currency": "货币类型",
    "status": "提现状态",
    "userId": "用户ID",
    "createTime": "创建时间"
  }
}
```

### 3.8 查询提现记录

- **接口说明**：查询提现申请的详细信息
- **接口路径**：`POST /partner/api/openapi/account/withdrawal/query`
- **请求参数**：

```json
{
  "apiKey": "第三方应用的API Key",
  "withdrawalNo": "提现单号",
  "timestamp": "当前时间戳，毫秒级",
  "nonce": "随机字符串，防重放",
  "signature": "签名，详见签名规则"
}
```

- **响应结果**：

```json
{
  "code": "0",
  "message": "success",
  "data": {
    "withdrawalNo": "提现单号",
    "amount": "提现金额",
    "currency": "货币类型",
    "status": "提现状态",
    "userId": "用户ID",
    "createTime": "创建时间",
    "completeTime": "完成时间",
    "remark": "备注信息"
  }
}
```



## 4. 签名规则

### 4.1 签名生成步骤

1. 将所有参数（除 signature 外）按照参数名 ASCII 码从小到大排序
2. 按照"参数名=参数值&参数名=参数值..."的格式拼接
3. 在最后拼接上"&key=密钥"，其中密钥为 appSecret
4. 对拼接后的字符串进行 SHA-256 哈希计算，并转为大写

### 4.2 签名示例

```
原始参数：
{
    "apiKey": "testAppKey",
    "timestamp": "1615887873123",
    "nonce": "abc123"
}

参数排序并拼接：
apiKey=testAppKey&nonce=abc123&timestamp=1615887873123&key=testAppSecret

SHA-256计算：
9F86D081884C7D659A2FEAA0C55AD015A3BF4F1B2B0B822CD15D6C15B0F00A08

最终signature值：
9F86D081884C7D659A2FEAA0C55AD015A3BF4F1B2B0B822CD15D6C15B0F00A08
```

### 4.3 签名验证规则

1. 请求超时时间为 5 分钟，超过该时间的请求将被拒绝
2. nonce 参数用于防止重放攻击，同一个 nonce 只能使用一次
3. 所有接口都必须携带签名参数，除非特别说明

## 5. 安全规范

### 5.1 API 密钥安全

- apiKey 可在前端使用，但应注意保护
- appSecret 严禁在前端使用，仅用于服务器间通信
- 建议定期轮换 appSecret

### 5.2 会话安全

- session_key 有效期为 2 小时
- 用户修改密码或主动退出登录时，所有 token 立即失效
- 异常登录行为可能导致 token 提前失效

### 5.3 最佳实践

- 所有请求使用 HTTPS 加密传输
- 重要操作需要二次验证
- 实现请求频率限制
- 记录所有授权操作日志
- 敏感接口增加 IP 白名单限制

## 6. 状态说明

### 6.1 支付订单状态

| 状态码   | 状态名称 | 说明                             |
| -------- | -------- | -------------------------------- |
| PENDING  | 待支付   | 订单已创建，等待用户支付         |
| SUCCESS  | 支付成功 | 用户支付已完成，交易成功         |
| FAILED   | 支付失败 | 支付过程中发生错误，交易失败     |
| CANCELED | 已取消   | 订单已被取消，无法再次支付       |
| EXPIRED  | 已过期   | 订单已超过支付时间，无法再次支付 |

### 6.2 退款订单状态

| 状态码   | 状态名称   | 说明                       |
| -------- | ---------- | -------------------------- |
| PENDING  | 退款处理中 | 退款申请已提交，正在处理   |
| SUCCESS  | 退款成功   | 退款已完成，资金已原路返回 |
| FAILED   | 退款失败   | 退款处理失败，请联系客服   |
| CANCELED | 已取消     | 退款申请已被取消           |

### 6.3 提现状态说明

| 状态码     | 状态名称 | 说明                           |
| ---------- | -------- | ------------------------------ |
| PENDING    | 待审核   | 提现申请已提交，等待审核       |
| APPROVED   | 已审核   | 提现申请已通过审核，准备转账   |
| PROCESSING | 处理中   | 提现正在处理，资金转账中       |
| SUCCESS    | 提现成功 | 提现已完成，资金已到账         |
| FAILED     | 提现失败 | 提现处理失败，资金已退回       |
| REJECTED   | 已拒绝   | 提现申请被拒绝，请查看拒绝原因 |

## 7. 支付流程

### 7.1 流程图

```
┌───────────┐          ┌───────────┐          ┌───────────┐          ┌───────────┐
│           │          │           │          │           │          │           │
│  用户前端  │          │ 系统APP   │          │ 系统服务端  │          │ 第三方服务器 │
│           │          │           │          │           │          │           │
└─────┬─────┘          └─────┬─────┘          └─────┬─────┘          └─────┬─────┘
      │                      │                      │                      │
      │                      │                      │   1.创建支付订单     │
      │                      │                      │<─────────────────────│
      │                      │                      │                      │
      │                      │                      │   2.返回订单和签名信息│
      │                      │                      │─────────────────────>│
      │                      │                      │                      │
      │   3.请求支付         │                      │                      │
      │<─────────────────────────────────────────────────────────────────>│
      │                      │                      │                      │
      │   4.获取支付参数     │                      │                      │
      │──────────────────────────────────────────────────────────────────>│
      │                      │                      │                      │
      │                      │                      │   5.获取支付参数     │
      │                      │                      │<─────────────────────│
      │                      │                      │                      │
      │                      │                      │   6.返回支付参数     │
      │                      │                      │─────────────────────>│
      │                      │                      │                      │
      │   7.返回支付参数     │                      │                      │
      │<──────────────────────────────────────────────────────────────────│
      │                      │                      │                      │
      │   8.唤起支付         │                      │                      │
      │─────────────────────>│                      │                      │
      │                      │                      │                      │
      │                      │   9.发起支付请求     │                      │
      │                      │─────────────────────>│                      │
      │                      │                      │                      │
      │                      │   10.支付结果        │                      │
      │                      │<─────────────────────│                      │
      │                      │                      │                      │
      │   11.支付结果回调    │                      │                      │
      │<─────────────────────│                      │                      │
      │                      │                      │                      │
      │                      │                      │   12.支付结果通知    │
      │                      │                      │─────────────────────>│
      │                      │                      │                      │
```

### 7.2 流程说明

1. 第三方服务器调用创建支付订单接口
2. 系统服务端返回订单信息和签名信息
3. 用户在前端发起支付请求
4. 用户前端通过第三方服务器获取支付参数
5. 第三方服务器调用获取支付参数接口
6. 系统服务端返回支付参数
7. 第三方服务器将支付参数返回给前端
8. 用户前端使用支付参数唤起系统 APP 进行支付
9. 系统 APP 向系统服务端发起实际支付请求
10. 系统服务端处理支付并返回结果给 APP
11. 系统 APP 将支付结果回传给用户前端
12. 系统服务端通过异步通知将支付结果通知给第三方服务器

## 8. 异步通知

### 8.1 支付结果通知

当支付订单状态发生变化时，系统会向创建订单时填写的`notifyUrl`发送 POST 请求。

#### 通知参数

```json
{
  "apiKey": "合作伙伴的API Key",
  "orderNo": "系统订单号",
  "partnerOrderNo": "合作伙伴订单号",
  "amount": "支付金额",
  "currency": "货币类型",
  "status": "订单状态",
  "payTime": "支付时间（支付成功时返回）",
  "timestamp": "通知时间戳",
  "nonce": "随机字符串",
  "signature": "签名"
}
```

#### 响应要求
接收到通知后，请返回字符串`success`表示接收成功，其他响应均视为失败。系统会进行重试，重试策略如下：

- 第 1 次：立即重试
- 第 2 次：5s 后重试
- 第 3 次：10s 后重试
- 第 4 次：30s 后重试
- 第 5 次：60s 后重试
- 第 6 次：120s 后重试



### 8.4 通知签名验证

通知的签名验证方式与 API 调用签名规则相同，请务必验证签名以确保通知的真实性。

## 9. 错误码说明

| 错误码   | 描述               | 处理建议           |
| -------- | ------------------ | ------------------ |
| 0        | 成功               | -                  |
| 1001     | 参数错误           | 检查请求参数       |
| 1002     | 签名验证失败       | 检查签名算法和参数 |
| 2001     | 合作伙伴不存在     | 检查 apiKey        |
| 2002     | 合作伙伴已禁用     | 联系管理员         |
| 2003     | 订单不可支付       | 检查订单状态       |
| 3001     | 权限不足           | 申请相应权限       |
| 3002     | 访问频率超限       | 降低访问频率       |
| 4000     | 参数错误           | 检查必填参数       |
| 4001     | 未登录或登录已过期 | 重新登录           |
| 4010     | 请求已超时         | 检查时间戳         |
| 5001     | 订单已存在         | 请勿重复创建       |
| 5002     | 订单不存在         | 检查订单号         |
| 10000001 | 合作伙伴不存在     | 检查 apiKey        |
| 10000002 | 签名错误           | 检查签名参数和算法 |
| 10000003 | 合作伙伴配置已过期 | 更新配置           |
| 10000004 | 用户不存在         | 检查用户标识       |

## 10. 开发示例

### 10.1 Java 示例代码

#### 签名生成示例

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.*;

public class SignatureUtil {

    public static String generateSignature(Map<String, String> params, String appSecret) {
        // 1. 参数排序
        TreeMap<String, String> sortedParams = new TreeMap<>(params);

        // 2. 构建签名字符串
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, String> entry : sortedParams.entrySet()) {
            if (!"signature".equals(entry.getKey()) && entry.getValue() != null) {
                sb.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
            }
        }
        sb.append("key=").append(appSecret);

        // 3. SHA-256加密
        return sha256(sb.toString()).toUpperCase();
    }

    private static String sha256(String input) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            byte[] hash = digest.digest(input.getBytes(StandardCharsets.UTF_8));
            StringBuilder hexString = new StringBuilder();
            for (byte b : hash) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) {
                    hexString.append('0');
                }
                hexString.append(hex);
            }
            return hexString.toString();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### API 调用示例

```java
import java.util.*;

public class PartnerApiClient {

    private String apiKey;
    private String appSecret;
    private String baseUrl;

    // 创建支付订单
    public String createPaymentOrder(PaymentOrderRequest request) {
        Map<String, String> params = new HashMap<>();
        params.put("apiKey", apiKey);
        params.put("amount", request.getAmount());
        params.put("currency", request.getCurrency());
        params.put("orderNo", request.getOrderNo());
        params.put("notifyUrl", request.getNotifyUrl());
        params.put("returnUrl", request.getReturnUrl());
        params.put("timestamp", String.valueOf(System.currentTimeMillis()));
        params.put("nonce", generateNonce());

        // 生成签名
        String signature = SignatureUtil.generateSignature(params, appSecret);
        params.put("signature", signature);

        // 发送HTTP请求
        return HttpUtil.post(baseUrl + "/partner/api/openapi/payment/create", params);
    }

    private String generateNonce() {
        return UUID.randomUUID().toString().replace("-", "").substring(0, 16);
    }
}
```

### 10.2 PHP 示例代码

```php
<?php
class PartnerApiClient {

    private $apiKey;
    private $appSecret;
    private $baseUrl;

    public function __construct($apiKey, $appSecret, $baseUrl) {
        $this->apiKey = $apiKey;
        $this->appSecret = $appSecret;
        $this->baseUrl = $baseUrl;
    }

    // 生成签名
    public function generateSignature($params) {
        // 移除signature参数
        unset($params['signature']);

        // 参数排序
        ksort($params);

        // 构建签名字符串
        $signStr = '';
        foreach ($params as $key => $value) {
            if ($value !== null && $value !== '') {
                $signStr .= $key . '=' . $value . '&';
            }
        }
        $signStr .= 'key=' . $this->appSecret;

        // SHA256加密并转大写
        return strtoupper(hash('sha256', $signStr));
    }

    // 创建支付订单
    public function createPaymentOrder($orderData) {
        $params = [
            'apiKey' => $this->apiKey,
            'amount' => $orderData['amount'],
            'currency' => $orderData['currency'],
            'orderNo' => $orderData['orderNo'],
            'notifyUrl' => $orderData['notifyUrl'],
            'returnUrl' => $orderData['returnUrl'],
            'timestamp' => (string)(time() * 1000),
            'nonce' => $this->generateNonce()
        ];

        // 生成签名
        $params['signature'] = $this->generateSignature($params);

        // 发送请求
        return $this->httpPost('/partner/api/openapi/payment/create', $params);
    }

    private function generateNonce() {
        return substr(md5(uniqid()), 0, 16);
    }

    private function httpPost($path, $data) {
        $url = $this->baseUrl . $path;
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, [
            'Content-Type: application/json'
        ]);

        $result = curl_exec($ch);
        curl_close($ch);

        return $result;
    }
}
?>
```

### 10.3 通知处理示例

```java
@RestController
public class NotifyController {

    @PostMapping("/notify/payment")
    public String paymentNotify(@RequestBody PaymentNotifyRequest request) {
        try {
            // 1. 验证签名
            if (!verifySignature(request)) {
                return "signature error";
            }

            // 2. 处理业务逻辑
            handlePaymentNotify(request);

            // 3. 返回成功标识
            return "success";
        } catch (Exception e) {
            log.error("处理支付通知失败", e);
            return "error";
        }
    }

    private boolean verifySignature(PaymentNotifyRequest request) {
        Map<String, String> params = new HashMap<>();
        params.put("apiKey", request.getApiKey());
        params.put("orderNo", request.getOrderNo());
        params.put("amount", request.getAmount());
        // ... 其他参数

        String calculatedSign = SignatureUtil.generateSignature(params, appSecret);
        return calculatedSign.equals(request.getSignature());
    }
}
```

## 11. 常见问题(FAQ)

### 11.1 接入相关

**Q: 如何申请成为合作伙伴？**
A: 请联系我们的商务团队，提供您的公司信息和业务需求，我们会为您开通测试账号。

**Q: 测试环境和生产环境有什么区别？**
A: 测试环境用于开发调试，数据会定期清理；生产环境是正式运营环境，有更严格的安全检查和频率限制。

**Q: API 调用有频率限制吗？**
A: 是的，为保证系统稳定性，我们对 API 调用有频率限制。具体限制请联系技术支持获取。

### 11.2 授权相关

**Q: access_token 过期了怎么办？**
A: 可以使用刷新令牌接口获取新的 access_token，或者重新进行用户授权流程。

**Q: 用户取消授权后，access_token 还能使用吗？**
A: 不能，用户取消授权后，相关的 access_token 会立即失效。

### 11.3 支付相关

**Q: 支付订单创建后多久过期？**
A: 默认 24 小时，过期后订单状态变为 EXPIRED，无法再次支付。

**Q: 为什么会收到重复的支付通知？**
A: 当您的回调接口没有正确返回"success"时，系统会进行重试。请确保处理成功后返回正确的响应。

**Q: 支持部分退款吗？**
A: 是的，退款金额可以小于等于原订单金额，支持多次部分退款，但累计退款金额不能超过原订单金额。

### 11.4 技术问题

**Q: 签名验证总是失败怎么办？**
A: 请检查以下几点：

1. 参数排序是否正确（按 ASCII 码排序）
2. 签名字符串构建是否正确
3. appSecret 是否正确
4. SHA-256 加密后是否转为大写

**Q: 接口返回"请求已超时"错误？**
A: 请检查 timestamp 参数，确保与服务器时间偏差不超过 5 秒钟

**Q: 如何处理网络异常？**
A: 建议实现重试机制，但要注意幂等性。对于订单创建等接口，建议先查询订单是否已存在。

## 12. 注意事项

### 12.1 安全注意事项

- 授权码有效期为 5 分钟，只能使用一次
- 访问令牌有效期为 2 小时
- 建议缓存 access_token，在过期前刷新
- 请求必须带上签名信息，防止篡改
- 请求时间戳与服务器时间偏差不能超过 5s 
- appSecret 严禁在前端代码中使用

### 12.2 业务注意事项

- 支付订单创建后默认有效期为 24 小时
- 同一个系统的订单号必须保证唯一性
- 异步通知可能会重复发送，请做好幂等性处理
- 重要业务数据建议通过查询接口主动获取，不要仅依赖异步通知

### 12.3 开发建议

- 建议在测试环境充分测试后再上线
- 实现完善的日志记录，便于问题排查
- 对关键接口实现监控告警
- 定期检查 token 有效性，及时刷新
