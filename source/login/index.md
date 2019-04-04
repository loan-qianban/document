**名称说明：业务方指钱伴/快贷，合作机构为外部导流渠道方**

# 联合登录

## 版本

2019.03.25

## 概述

- 本文档对接入合作机构做联合登录进行说明，联合登录定义为如果用户在合作机构 App 中已登录，跳转到业务方的 h5 流程后如果为`全新用户`不再需要用户手动登录，`全新用户`定义为没有登录过业务方 App 的用户

## 合作机构接入流程

1. 合作机构提交合作意向
2. 合作机构需求确定
3. 签订合作协议
4. 合作机构从业务方获取测试环境 appKey、appSecret等参数，并加载业务方提供的 SDK
5. 合作机构依据本文档进行系统研发（包括 App 开发）、自测、联调（业务方提供技术支持）
6. 合作机构自测并联调完毕后，提供 App 测试包，申请测试
7. 业务方测试人员进行全流程验证和测试
8. 测试通过，申请正式环境的 appKey 和 appSecret
9. 上线，并观察线上数据
10. 后续优化

## 流程图

- ![image](http://s1.wacdn.com/wis/532/592bc9b61f0732af_1215x711.png)

## 业务方 SDK 使用简介

- 合作机构通过业务方[SDK](https://github.com/wacai/wacai-open-sdk)调用业务方 API 接口。并且，合作机构收到业务方的反馈消息后，也通过业务方 SDK 进行验签，确保消息是来自业务方的合法消息

### maven 依赖

```java
<dependency>
    <groupId>com.wacai</groupId>
    <artifactId>wacai-openAllInOne-sdk</artifactId>
    <version>1.5.0</version>
</dependency>
​```
```

### 代码案例

#### 构建Client和Request

```java
WacaiOpenApiClient wacaiOpenApiClient = new WacaiOpenApiClient("${appKey}", "${appSecret}");
// 测试联调时使用，将调用请求指向挖财的沙箱环境，生产环境请删除
wacaiOpenApiClient.setGatewayEntryUrl("http://guard.ngrok.wacaiyun.com/gw/api_entry");
wacaiOpenApiClient.init();
WacaiOpenApiRequest wacaiOpenApiRequest = new WacaiOpenApiRequest("${API名称}", "${API版本号}");
wacaiOpenApiRequest.putBizParam("约定的参数1", "34121141242144");
wacaiOpenApiRequest.putBizParam("约定的参数2", "");
```

#### Response：

```java
OrderDeleteResponseObject wacaiOpenApiResponse = wacaiOpenApiClient.invoke(wacaiOpenApiRequest, new TypeReference<OrderDeleteResponseObject>() {});
```

### 使用业务方 SDK 的参数说明

| 参数名     | 类型   | 说明                                      |
| ---------- | ------ | ----------------------------------------- |
| APP_KEY    | string | 使用业务方 SDK 时用，相当于用户名，业务方提供 |
| APP_SECRET | string | 使用业务方 SDK 时用，相当于密码，业务方提供   |
| API 名称   | string | 业务方分配给合作机构的 API 名，业务方提供     |
| API 版本号 | string | 业务方分配给合作机构的 API 版本号，业务方提供 |

### 传给业务方的参数说明

| 参数名     | 类型       | 是否必填 | 说明                                                         |
| ---------- | ---------- | -------- | ------------------------------------------------------------ |
| openId     | string     | yes      | 标志合作机构的用户 uid                                       |
| merchantNo | string     | yes      | 标志合作机构的商户号，业务方提供                             |
| mob        | string     | yes      | 用户手机号                                                   |
| sourceMc   | string     | yes      | 业务方分配给合作机构市场码，业务方提供                       |
| sourceAf   | string     | yes      | 业务方分配给合作机构的渠道号，业务方提供                     |
| map        | JsonObject | no       | 扩展参数需放在 map 里传给业务方，如姓名手机号等，代码示例<br/>  `wacaiOpenApiRequest.putBizParam("map", {"name": "xxxxx", "idNo": "4242424242424234234"});` |

### response说明

- 成功时

```
  {
   "data": "url",
   "success": true
  }
```

- 返回异常时

```
	{
		"errorCode": "U0029",
		"errorMsg": "参数异常",
		"success": false
	}
```

| 错误码 | 错误信息 | 备注 |
| ------ | -------- | ---- |
| U0029  | 参数错误 |      |
| U9999  | 服务异常 |      |
| U0076  | 没有权限 |      |

  

### 业务方 SDK 文档

- [业务方 API 网关SDK文档](https://github.com/wacai/wacai-open-sdk)

## 前端交互

### 合作机构跳转到业务方 h5 产品后流程图

![image](https://s1.wacdn.com/wis/532/f9cde654e9ad952b_735x576.png)
- 没有登录过业务方 App 的用户被定义为全新用户，只有全新用户才会自动登录
- 用户退出后如果没有在业务方 App 登录过，再次进入还会被判定为全新用户，走自动登录

## 联调方案

- 业务方向合作机构提供相关参数
- 合作机构需提供带业务方 h5 产品入口的测试 App，如果需要开白名单等操作要给出相应文档
- 如果合作机构要接入业务方内网测试联调，需向业务方提供姓名、手机号，合作机构的同学可通过手机验证码登录临时 vpn，接入业务方内网