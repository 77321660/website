 ## 开发前必读
开发者在接入平台服务端API之前，需要先获取access_token

对于每个API请求，都需要携带access_token，提供以下两种方式将access_token包含在请求中：

- 在HTTP Header中包含Authorization

通常使用的方法是在HTTP请求的Authorization头域中包含access_token。需要注意，Authorization的值为：`"Bearer " + access_token`

- 在URL中包含access_token

用户也可以把access_token放在HTTP请求Query String的access_token参数中

### 三方企业应用获取access_token

#### 第一步：计算API签名

把timestamp+"\n"+random当做签名字符串，clientSecret做为签名秘钥，使用HmacSHA256算法计算签名，然后进行Hex.getHexString获取最后签名结果signature。

**签名参数说明**

| 参数      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| clientId  | 平台分配的第三方应用ID                                       |
| timestamp | 当前时间戳，格式：yyyyMMddHHmmssSSS                          |
| random    | 随机数，字符串长度6                                          |
| signature | 以timestamp+"\n"+random为签名字符串，clientSecret为签名秘钥（平台分配的第三方应用密钥，与clientId是一对），使用算法HmacSHA256计算的签名值。 |

> **签名计算**

```java
String stringToSign = timestamp+"\n"+random
Mac mac = Mac.getInstance("HmacSHA256");
mac.init(new SecretKeySpec(clientSecret.getBytes("UTF-8"), "HmacSHA256"));
byte[] signData = mac.doFinal(stringToSign.getBytes("UTF-8"));
return Hex.getHexString(signData);
```

#### 第二步：访问服务端API  
请求方式:`POST`
请求URI: `/auth/rc/token`
请求地址: `http://ip:port/auth/rc/token?signature=87d51b7xxxxx&timestamp=20181105164756977&random=643338&clientId=5bd952xxxx`

##### 返回结果示例

```
{
    "access_token": "xxxx",
    "expires_in": 7200
}
```

##### 结果参数说明

| 参数         | 说明                       |
| ------------ | -------------------------- |
| access_token | 授权方access_token         |
| expires_in   | 授权方access_token超时时间 |

## 设置塔吊信息

描述：`只有设置过塔吊基本信息，才可以上报实时数据帧`
请求方式：`POST`
请求URI:：`/admin/craneInfo/setting`

### 请求参数

| 名称       | 类型   | 是否必填 | 描述                     |
| :--------- | ------ | -------- | ------------------------ |
| craneno    | String | 是       | 设备编号，唯一，不可修改 |
| craneAlias | String | 是       | 设备别名                 |
| armLength  | Double | 是       | 臂长，单位米             |
| tailLength | Double | 是       | 尾臂长，单位米           |
| height     | Double | 是       | 塔高，单位米             |
| safeTorque | Double | 是       | 额定力矩，单位焦耳       |
| safeWeight | Double | 是       | 安全起重量，单位吨       |

### 响应参数

| 名称    | 类型    | 是否必填 | 描述                     |
| :------ | ------- | -------- | ------------------------ |
| status  | Integer | 是       | 200成功 500失败          |
| message | String  | 否       | 失败时必填，详细错误信息 |

### 报文示例

- 请求

{
​	"armLength":37.5,
​	"craneAlias":"610378",
​	"craneno":"610378",
​	"height":13.2,
​	"safeTorque":60.8,
​	"safeWeight":1.2,
​	"tailLength":11.2
}

- 响应

{
​	"message":"success",
​	"status":200
}

## 上报实时数据帧

描述：`实时数据帧是在设备相应传感器有变化时发送的，所以针对时间间隔无法获知，但最低时间间隔不会低于 10s`
请求方式：`POST`
请求URI:：`/admin/craneRecord/upload`

### 请求参数

| 名称            | 类型   | 是否必填 | 描述                     |
| :-------------- | ------ | -------- | ------------------------ |
| craneno | String | 是       |设备编号 |
| card | String | 是 |司机卡号 |
| name | String | 是 |司机姓名 |
| safePerson | String | 是 |安全责任人姓名 |
| createTime | String | 是 |时间，格式yyyyMMddHHmmss |
| geoCoordinateY | Double | 是 |经度，单位度 |
| geoCoordinateX | Double | 是 |维度，单位度 |
| height | Double | 是 |高度，单位米 |
| amplitude | Double | 是 |幅度，单位米 |
| rotary | Double | 是 |回转，单位度 |
| weight | Double | 是 |实时重量，单位吨 |
| torque | Double | 是 |实时力矩，单位焦耳 |
| windSpeed | Double | 是 |风速，单位m/s |
| angleX | Double | 是 |倾角 X，单位度 |
| angleY | Double | 是 |倾角 Y，单位度 |
| safeTorque | Double | 是 |额定力矩，单位焦耳 |
| safeWeight | Double | 是 |安全起重量，单位吨 |
| currentRate | Double | 是 |倍率 |
| overweight | Boolean | 是 |是否超重，true：超重，false：不超重 |
| overmoment | Boolean | 是 |是否超力矩，true：超力矩，false不超力矩 |
| powerStatus | String | 是 |控制状态 |
| sensorStatus | String | 是 |传感器状态 |
| warnType | String | 是 |预警告码 |
| alertFlag | String | 是 |报警告码 |

- 控制状态说明

举例：1000000000010001 表示：锁定，高度减速，回转左限位被置位。

| Bit15    | Bit14      | Bit13      | Bit12    | Bit11      | Bit10      | Bit9       | Bit8       |
| -------- | ---------- | ---------- | -------- | ---------- | ---------- | ---------- | ---------- |
| 锁定     | 保留       | 风速报警   | 风速预警 | 幅度换速   | 幅度内限位 | 幅度外限位 | 幅度预减速 |
| Bit7     | Bit6       | Bit5       | Bit4     | Bit3       | Bit2       | Bit1       | Bit0       |
| 高度换速 | 高度下限位 | 高度上限位 | 高度减速 | 回转右减速 | 回转右限位 | 回转左减速 | 回转左限位 |

- 传感器状态说明

举例：0000000000010001 表示：风速，重量被置位。

| Bit15 | Bit14 | Bit13 | Bit12 | Bit11 | Bit10 | Bit9 | Bit8 |
| ----- | ----- | ----- | ----- | ----- | ----- | ---- | ---- |
| 保留  | 保留  | 保留  | 保留  | 保留  | 保留  | 保留 | 保留 |
| Bit7  | Bit6  | Bit5  | Bit4  | Bit3  | Bit2  | Bit1 | Bit0 |
| 保留  | 保留  | 倾角  | 风速  | 幅度  | 回转  | 高度 | 重量 |

- 预警告码、报警告码说明

举例：00000000000100010000000000010001 表示：左限位，上限位，倾斜， 风速几个报警或预警，如果是 warnType 是预警，如果是 alertFlag 是报警。

| Bit31      | Bit30      | Bit29      | Bit28      | Bit27  | Bit26  | Bit25  | Bit24  |
| ---------- | ---------- | ---------- | ---------- | ------ | ------ | ------ | ------ |
| 保留       | 保留       | 保留       | 保留       | 保留   | 保留   | 保留   | 保留   |
| Bit23      | Bit22      | Bit21      | Bit20      | Bit19  | Bit18  | Bit17  | Bit16  |
| 保留       | 保留       | 右限位     | 左限位     | 内限位 | 外限位 | 下限位 | 上限位 |
| Bit15      | Bit14      | Bit13      | Bit12      | Bit11  | Bit10  | Bit9   | Bit8   |
| 区域保护右 | 区域保护左 | 区域保护后 | 区域保护前 | 右碰撞 | 左碰撞 | 后碰撞 | 前碰撞 |
| Bit7       | Bit6       | Bit5       | Bit4       | Bit3   | Bit2   | Bit1   | Bit0   |

### 响应参数

| 名称      | 类型    | 是否必填 | 描述                    |
| :-------- | ------- | -------- | ----------------------- |
|status		|Integer|	是	|200成功 500失败|
|message	|	String|	否	|失败时必填，详细错误信息|

### 报文示例

- 请求

{
​	"alertFlag":"00000000000100010000000000010001",
​	"amplitude":50.7,
​	"angleX":0.0,
​	"angleY":0.0,
​	"card":"00000001",
​	"craneno":"610378",
​	"createTime":"20181229112430",
​	"currentRate":2.0,
​	"geoCoordinateX":0.0,
​	"geoCoordinateY":0.0,
​	"height":29.4,
​	"name":"李四",
​	"overmoment":false,
​	"overweight":false,
​	"powerStatus":"1000000000010001",
​	"rotary":0.0,
​	"safePerson":"张三",
​	"safeTorque":60.8,
​	"safeWeight":1.2,
​	"sensorStatus":"0000000000010001",
​	"torque":1.2,
​	"warnType":"00000000000100010000000000010001",
​	"weight":0.1,
​	"windSpeed":0.0
}

- 响应

{
​	"message":"success",
​	"status":200
}
