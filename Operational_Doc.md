# Iplayable网盟平台运营配置指南
## 1. 概要
本文档旨在说明 Iplayable 网盟平台运营人员需要进行的详细操作以及需要配置的内容。本文档分为两个部分，第一部分为与下游对接并在 Iplayable 平台中配置相关内容，第二部分为与上游/三方对接并在 Iplayable 平台中配置相关内容。

## 2. 对接下游

### 2.1 配置 click_url
发送get请求，请求地址：
```bash
http://callback.flatmobi.com/api/v1/click
```

支持的点击请求参数：

| 字段名称 | 对应宏参 | 参数说明 | 是否必须 |
| -------- | ------ | -------- | ------ |
| app_id | {app_id} | 渠道 id | 是 |
| offer_id | {offer_id} | offerid | 是 |
| clickid | {clickid} | 下游渠道生成的click id | 是 |
| gaid | {gaid} | 谷歌广告 ID | android系统必传 |
| idfa | {idfa} | IOS idfa | ios系统必传 |
| android | {android} | android信息 | 否 |
| affsub | {affsub} | 子渠道信息 | 否 |
| subid | {subid} | 子渠道 id | 否 |
| para1 | {para1} | 自定义参数1 | 否 |
| para2 | {para2} | 自定义参数2 | 否 |
| para3 | {para3} | 自定义参数3 | 否 |

示例：

在第3步查询 offer 响应内容示例中datas字段中的 click_url 为 ：

http://callback.flatmobi.com/api/v1/getoffer?app_id=1000&offer_id=22&clickid={clickid}&gaid={gaid}&android={android}&idfa={idfa}&subid={subid}&affsub={affsub}

点击上报时将 {click_id}，{gaid} 等替换为自己渠道的宏，替换后发送给 Iplayable 网盟平台，例如 ：

http://callback.flatmobi.com/api/v1/click?app_id=1000&offer_id=22&clickid=abc&gaid=4716d154-7232-11ee-800e-e9037848532a&android={android}&idfa={idfa}&subid={subid}&affsub=testabc&para1=somevalue1&para2=somevalue2

注：para1、para2、para3 为预留自定义参数，若需要在 postback 中回传一些额外内容可在此添加，添加格式如上方示例 url ，并与 Iplayable 平台客户经理联系在 postback 中配置，配置细节将在第 5 节中说明。



Iplayable 网盟平台支持的回调链接宏参数：

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {clickid} | 点击id，下游渠道生成的click id ，安装的唯一标识 | 是 |
| {app_id} | 渠道 ID，分配给渠道的唯一标识 | 是 |
| {offer_id} | 单子唯一标识 | 是 |
| {gaid} | 谷歌广告 ID | android系统必传 |
| {idfa} | IOS idfa | ios系统必传 |
| {android} | android信息 | 否 |
| {affsub} | 子渠道信息 | 否 |
| {subid} | 子渠道 id | 否 |
| {para1} | 自定义参数1 | 否 |
| {para2} | 自定义参数2 | 否 |
| {para3} | 自定义参数3 | 否 |
| {insts} | 安装时间戳 | 否 |
| {breason} | Block reason | 否 |
| {bsub} | Block sub reason | 否 |
| {bvalue} | Block value | 否 |
| {ename} | 后链路事件，注册或者付费等 | 否 |
| {erev} | 后链路事件收益 | 否 |
| {etime} | 后链路事件时间戳 | 否 |
| {evalue} | 后链路事件value | 否 |
| {payout} | Offer 收益，货币为美金 | 否 |

在进行安装回传时，需要三步，首先需要在 Iplayable 平台配置 postback url ，然后 Iplayable 平台会根据下游发送的点击以及上游或三方传来的回调数据提取宏参，最后 IPlayable 平台会使用提取的填充 postback url 使用填充后的 postback url 对下游进行get请求，**示例如下：**

a. Iplayable 系统配置postback url，例如：

http://your.domain.com/your/route?your_click_para={clickid}&your_aff_para={app_id}&your_camp_para={offer_id}&your_blockreason_para={breason}&your_payout_para={payout}&other_fix=fixvalue&other_fix2=fixvalue2&your_name={para1}

Postback url注解：

在真实生产环境中配置时，请将 your.domain.com、your/route 替换为自己的回调主域名和路径；your_click_para、your_aff_para、your_camp_para、your_blockreason_para、your_payout_para 请替换为自己的参数名；other_fix、other_fix2 为可配置的固定回传参数，配置时请替换为自己的固定参数名称；your_name 为第 4 步中接收到的预留自定义参数，请替换为自己的自定义参数名称；

b. Iplayable 网盟平台从下游发送的的 Click_url 以及上游或者三方的回调提取宏参，例如：

Iplayable 平台接收到下游发送的 Click_url 如第4步所示：

http://callback.flatmobi.com/api/v1/click?app_id=1000&offer_id=22&clickid=abc&gaid=4716d154-7232-11ee-800e-e9037848532a&android={android}&idfa={idfa}&subid={subid}&affsub=testabc&para1=somevalue1&para2=somevalue2

Iplayable 平台接收到的上游或者三方的回调：

http://callback.flatmobi.com/our_rout?our_click_param=123&istall_time=2023-11-11&block_reason=test1&payout=1.5

继而提取到宏参如下：

| 宏参 | 宏参值 |
| --- | ------ |
| {app_id} | 1000 |
| {offer_id} | 22 |
| {clickid} | abc |
| {gaid} | 4716d154-7232-11ee-800e-e9037848532a |
| {affsub} | testabc |
| {insts} | 2023-11-11 |
| {breason} | test1 |
| {payout} | 1.5 |
| {para1} | somevalue1 |
| {para2} | somevalue2 |

c. Iplayable 使用步骤 b 提取到的宏参填充步骤a中配置的 postback url ，并使用填充后的 url 进行 get 请求，填充结果如下：

http://your.domain.com/your/route?your_click_para=abc&your_aff_para=1000&your_camp_para=22&your_blockreason_para=test1&your_payout_para=1.5&&other_fix=fixvalue&other_fix2=fixvalue2&your_name=somevalue1



## 3. 对接上游/三方

