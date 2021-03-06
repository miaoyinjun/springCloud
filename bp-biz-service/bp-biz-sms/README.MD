## 短信发送微服务
本服务为拓道金服短信微服务，集成三种短信平台
- [互亿无线短信平台](http://www.ihuyi.com) 
- [创蓝短信](https://www.253.com/)(需报备访问IP)
- [梦网短信](http://www.montnets.com/)  

本服务指定了五个策略，以防止短息轰炸的入侵：
>1.访问IP限制  
>2.短信发送间隔时间；  
>3.同一个（批）手机号一天内能发送短信数;  
>4.同一客户端IP一天内能发送短信数;  
>5.同一个（批）手机号绑定客户端IP数。

每次请求如果请求失败，重试三次

## 相关配置：
帐号配置数据为DB配置：
```$xslt
-- ----------------------------
-- Table structure for sms_accout
-- ----------------------------
DROP TABLE IF EXISTS `sms_accout`;
CREATE TABLE `sms_accout` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `sms_servicer` int(1) NOT NULL COMMENT '短信服务商（1；互亿；2：创蓝）',
  `sms_account` varchar(255) NOT NULL COMMENT '帐号(APIID)',
  `sms_password` varchar(255) NOT NULL COMMENT '密码(APIKEY)',
  `sms_url` varchar(255) NOT NULL COMMENT '请求地址',
  `sms_status` int(1) NOT NULL COMMENT '状态（1:启用；2：禁用）',
  `gmt_create` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modify` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `gmt_creator` varchar(12) NOT NULL COMMENT '添加人',
  `gmt_modifier` varchar(12) DEFAULT NULL COMMENT '修改人',
  `is_del` int(1) DEFAULT '1' COMMENT '是否删除(0：已删除；1:正常)',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COMMENT='短信用户配置表';

-- ----------------------------
-- Records of sms_accout
-- ----------------------------
INSERT INTO `sms_accout` VALUES ('1', '1', 'C82556906', '8bdae916fa500da2ddd858587ecf06c2', 'http://106.ihuyi.com/webservice/sms.php?method=Submit', '2', '2017-08-17 15:35:39', '2017-08-23 17:14:08', 'admin', null, '1');
INSERT INTO `sms_accout` VALUES ('3', '2', 'VIP-tdjf', 'Tch792743', 'https://sapi.253.com/msg/HttpBatchSendSM', '1', '2017-08-17 17:23:46', '2017-08-23 17:15:17', 'admin', null, '1');
INSERT INTO `sms_accout` VALUES ('5', '3', 'JS1758', '668300', 'http://61.145.229.26:8086/MWGate/wmgw.asmx/MongateCsSpSendSmsNew', '2', '2017-08-21 09:38:54', '2017-08-23 17:15:21', 'admin', null, '1');

```

---
##请求：
本服务仅支持 **POST** 请求，请求地址为：localhost:50001/sms/sendSms，请求内容如下：
```****
POST /sendSms HTTP/1.1
Host: localhost:50001
Content-Type: application/json
{
  "mobiles":"手机号码手机号码（多个以";"隔开）",
  "content":"短信内容",
  "customerIp":"客户端请求IP(非Http请求IP)"
}
```   
 
参数说明：

参数         |   类型    |   长度  |   说明  
------------|-----------|--------|----------------------------------------
mobiles     |   String  |   11   |   手机号码（多个以";"隔开）
content     |   String  |   255  |   短信内容(需注意转义字"\")
customerIp  |   String  |   15   |   客户端请求IP(非Http请求IP)

## 返回结果
### 成功

```
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Tue, 16 May 2017 08:31:25 GMT
{
	"code": 100000,
	"msg": "成功",
	"content":{
		"successCount": 成功记录数,
		"errorCount": 失败记录数,
		"errorMap":{
			"手机号码": "失败原因",
			"15957178759": "117"
		}
	},
	"success": true
}
```

### 失败

```
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Tue, 16 May 2017 08:31:19 GMT
{
	"info": {
		"code": 错误代码,
		"msg": "错误原因"
	},
	"content": null
}
```

只有当 **code=100000** 时才表示发送成功

---
请求返回结果列表:
* 平台返回异常（互亿）   
 
code	|   msg
--------|------------
400	    |   非法ip访问
401	    |   帐号不能为空
402	    |   密码不能为空
403	    |   手机号码不能为空
4030	|   手机号码已被列入黑名单
404	    |   短信内容不能为空
405	    |   用户名或密码不正确
4050	|   账号被冻结
4051	|   剩余条数不足
4052	|   访问ip与备案ip不符
406	    |   手机格式不正确
407	    |   短信内容含有敏感字符
4070	|   签名格式不正确
4071	|   没有提交备案模板
4072	|   短信内容与模板不匹配
4073	|   短信内容超出长度限制
408	    |   您的帐户疑被恶意利用，已被自动冻结，如有疑问请与客服联系。  


* 创蓝异常

code	|   msg
--------|------------
0	|提交成功
101	|无此用户
102	|密码错误
103	|提交过快（提交速度超过流速限制）
104	|系统忙（因平台侧原因，暂时无法处理提交的短信）
105	|敏感短信（短信内容包含敏感词）
106	|消息长度错（>536或<=0）
107	|包含错误的手机号码
108	|手机号码个数错（群发>50000或<=0）
109	|无发送额度（该用户可用短信数已使用完）
110	|不在发送时间内
113	|extno格式错（非数字或者长度不对）
116	|签名不合法或未带签名（用户必须带签名的前提下）
117	|IP地址认证错,请求调用的IP地址不是系统登记的IP地址
118	|用户没有相应的发送权限（账号被禁止发送）
119	|用户已过期
120	|违反放盗用策略(日发限制) --自定义添加
121	|必填参数。是否需要状态报告，取值true或false
122	|5分钟内相同账号提交相同消息内容过多
123	|发送类型错误
124	|白模板匹配错误
125	|匹配驳回模板，提交失败
126	|审核通过模板匹配错误

 * 微服务异常

code	|   msg
--------|------------
100000  |	请求成功
200001	|   系统异常
200002	|   发送短信失败：功能未启用 
200003	|   短信发送太频繁，请稍候
200004	|   本号码当天短信发送量达到上限
200005	|   本IP当天短信发送量达到上限
200006	|   本号码1天内最多可绑定的IP发送短信达到上限
200007	|   互亿短信未配置或已禁用 
200008	|   短信合作平台异常
200009	|   返回结果解析异常
200010	|   电话号码不能为空
200011	|   电话号码格式错误
200012	|   短信内容不能为空
200013	|   短信内容最大300字
200014	|   客户端IP不能为空
200015	|   客户端IP格式错误
200016	|   短息类型不能为空
200017	|   短息类型只能为1(通知类)和2(营销类)
200018	|   访问IP受限,请与管理员联系
---