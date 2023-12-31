# Iplayable网盟平台运营配置指南
## 1. 概要
本文档旨在说明 Iplayable 网盟平台运营人员需要进行的详细操作以及需要配置的内容。本文档分为两个部分，第一部分为与下游对接并在 Iplayable 平台中配置相关内容，请参考第 2 节；第二部分为与上游/三方对接并在 Iplayable 平台中配置相关内容，请参考第 3 节。

## 2. 对接下游

运营需要配置两项内容，分别为 click_url 和 postback_url 。

### 2.1 配置 click_url 点击跟踪链接

该配置的 click_pattern 为下游在从 Iplayable 平台拉取 offer 单子时获取到的点击链接，详情可见 Downstream_Doc.md 文档。填写的数据库位置为 downstream 表的 click_pattern 字段，每个下游都需单独配置。

click_pattern 字段填写示例：

http://callback.flatmobi.com/api/v1/click?app_id={app_id}&offer_id={offer_id}&gaid={gaid}&idfa={idfa}

其中协议部分、域名部分、路径部分请保持固定，仅需配置查询参数部分，查询参数部分配置规制如下。
<!-- **注：**

a. 其中参数部分的参数名可以取任何名字，但需要和 2.2 节中配置的 postback_url 参数部分的参数值（宏）对应，例如：

| click_pattern 不同参数名 | click_pattern 配置 | postback_url 对应配置 |
| ---------------------- | ----------------- | -------------------- |
| 参数名为 offerid 时 | offerid={offer_id} | camp={offerid} |
| 参数名为 campid 时  | campid={offer_id} | camp={campid} |

b.  参数部分的参数值（宏）需要替换为下游渠道的宏，例如： -->

<!-- | 参数名 | click_pattern 配置的宏 | postback_url 对应配置 |
| ---------------------- | ----------------- | -------------------- |
| offerid | offerid={offer_id} | camp={offer_id} |
| offerid | offerid={camp_id} | camp={camp_id} |  -->

**Iplayable 平台支持的点击请求的参数：**

| 参数名称 | 对应宏参 | 参数说明 | 是否必须 |
| -------- | ------ | -------- | ------ |
| app_id | {app_id} | 渠道 id | 是 |
| offer_id | {offer_id} | offerid | 是 |
| gaid | {gaid} | 谷歌广告 ID | android系统必传 |
| idfa | {idfa} | IOS idfa | ios系统必传 |

点击跟踪链接中的查询参数的名称和宏参必须对应（如上），并且不支持更改。


<!-- para1、para2、para3 为预留自定义参数，若下游需要在 postback 中回传一些额外内容可在此添加，添加格式如下方示例 url ，并需要 Iplayable 平台运营在 postback_url 中进行相关配置，配置细节将在 2.2 节中说明。 -->

其中app_id 和 offer_id 查询参数的值为 Iplayable 网盟平台进行填写，填写后在下游查询offer时返回给下游；gaid、idfa 查询参数的值为下游进行填写替换，替换为对应值后发送给 Iplayable 网盟平台，例如下游填写后发送给 Iplayable 网盟平台的内容应为以下**示例** ：

http://callback.flatmobi.com/api/v1/click?app_id=1000&offer_id=22&gaid=4716d154-7232-11ee-800e-e9037848532a&idfa={idfa}

* 没有使用宏替换的查询参数表示下游在该次点击请求中不提供该参数，例如 idfa={idfa}。

* 若下游需要添加其他额外参数，与下游沟通后请在 click_pattern 字段配置时添加，并在 postback url 中添加。例如，下游想在点击跟踪链接中添加 clickid 参数，需要修改以下两处：

* 1）配置 downstream 表的 click_pattern 字段，添加 clickid={click_id} ，其中参数名 clickid 可以任意内容，宏参 {click_id} 也可为任意，但宏参需要与 postback url 中对应，修改示例：

http://callback.flatmobi.com/api/v1/click?app_id={app_id}&offer_id={offer_id}&gaid={gaid}&idfa={idfa}&clickid={click_id}

* 2）配置 postback url ，具体细节会在 2.2.x 节中说明。


### 2.2 配置 postback url 回传url

这里配置的 postback_url 为 Iplayable 平台需要回传给下游的信息，填写位置为 downstream 表的 postback_url 字段和 event_postback_url 字段。其中 postback_url 字段为安装事件回传链接，event_postback_url 为后链路事件回传链接。每个下游要单独配置并填写这两个字段，若暂时无后链路事件，event_postback_url可暂时为空。

**Iplayable 网盟平台支持的回调链接宏参：**

所支持的宏参可分为以下几类：

**a.** 点击跟踪链接信息回传宏参

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {app_id} | 渠道 ID，分配给渠道的唯一标识 | 否 |
| {offer_id} | 单子唯一标识 | 否 |
| {gaid} | 谷歌广告 ID | 否 |
| {idfa} | IOS idfa | 否 |

* Iplayable 网盟平台支持将点击跟踪链接请求的的信息回传给下游。宏参{app_id}、{offer_id}、{gaid}、{idfa} 对应的值来源于下游发送的 click_url ，见第4节，分别对应第 4 节中的宏参 {app_id}、{offer_id}、{gaid}、{idfa}。

**b.** 上游/三方安装信息回传宏参

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {insts} | 安装时间戳，仅支持安装事件回传 | 否 |
| {breason} | Block reason，仅支持安装事件回传 | 否 |
| {bsub} | Block sub reason，仅支持安装事件回传 | 否 |
| {bvalue} | Block value，仅支持安装事件回传 | 否 |

* 该部分宏参对应的值来自于上游/三方的安装回调，因此仅支持安装事件信息的回传。

**c.** 上游/三方后链路事件信息回传宏参

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {ename} | 后链路事件，注册或者付费等，仅支持后链路事件回传 | 否 |
| {erev} | 后链路事件收益，仅支持后链路事件回传 | 否 |
| {etime} | 后链路事件时间戳，仅支持后链路事件回传 | 否 |
| {evalue} | 后链路事件value，仅支持后链路事件回传 | 否 |

* 该部分宏参对应的值来自于上游/三方的后链路事件回调，因此仅支持后链路事件信息的回传。

**d.** 其他宏参

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {payout} | Offer 收益，货币为美金 | 否 |

#### 2.2.1 配置 postback_url 字段

该字段配置位置为 downstream 表的 postback_url 字段，当回传事件为安装行为时，系统会使用该字段配置的url，对于安装行为，Iplayable 系统支持 a、b、d 三部分宏参，不支持配置和回传 c 部分宏参。

**配置示例：**

http://your.domain.com/your/route?your_aff_para={app_id}&your_camp_para={campaign_id}&your_gaid_para={gaid}&your_blockreason_para={breason}&your_payout_para={payout}&other_fix=fixvalue

* 配置时请将 your.domain.com、your/route 替换为下游的真实回调域名和路径；your_aff_para、your_camp_para、your_gaid_para、your_blockreason_para、your_payout_para 请替换为下游的查询参数名称；other_fix为可配置的固定回传参数，配置时请替换为下游要求的固定查询参数名称，并配置固定查询参数值。

* 若下有想要回传其他不包含在 a 中的参数信息，例如，下游想要回传 clickid 参数，该参数的宏参需要与 2.1 节中对应，例如 2.1 节 click_pattern 使用的是 {click_id} 宏参，这里也必须配置使用该宏参，在 postback_url 中添加 your_click_para={click_id}，your_click_para 请替换为下游的参数名称。

* 要配置 b 中的相关参数需要在 upstream 表中的 postback_pattern 字段配置对应内容；示例链接配置的是 {breason} 参数，若下游想要回传其他从上游/三方得到的安装信息，也可以在 upstream 表中的 postback_pattern 字段配置，前提是上游/三方支持回传该参数信息。详细配置规则将在 3.2.x.1 中说明。




#### 2.2.2 配置 event_postback_url 字段

该字段配置位置为 downstream 表的 event_postback_url 字段，当回传事件为后链路行为时，系统会使用该字段配置的url，对于后链路行为，Iplayable 系统支持a、c、d三部分宏参，不支持配置和回传b部分宏参。

**配置示例：**

http://your.domain.com/your/route?your_aff_para={app_id}&your_camp_para={campaign_id}&your_gaid_para={gaid}&your_eventname_para={ename}&your_payout_para={payout}&other_fix=fixvalue

* 配置时请将 your.domain.com、your/route 替换为下游的真实回调域名和路径；your_aff_para、your_camp_para、your_gaid_para、your_eventname_para、your_payout_para 请替换为下游的查询参数名称；other_fix为可配置的固定回传参数，配置时请替换为下游要求的固定查询参数名称，并配置固定查询参数值。

* 若下有想要回传其他不包含在 a 中的参数信息，例如，下游想要回传 clickid 参数，该参数的宏参需要与 2.1 节中对应，例如 2.1 节 click_pattern 使用的是 {click_id} 宏参，这里也必须配置使用该宏参，在 event_postback_url 中添加 your_click_para={click_id}，your_click_para 请替换为下游的参数名称。

* 要配置 c 中的相关参数需要在 upstream 表中的 postback_event_pattern 字段配置对应内容；示例链接配置的是 {ename} 参数，若下游想要回传其他从上游/三方得到的后链路信息，也可以在 upstream 表中的 postback_event_pattern 字段配置，前提是上游/三方支持回传该参数信息。详细配置规则将在 3.2.x.2 中说明。




<!-- **postback_url 字段填写示例：**

http://some.downstream.domain.com/postback?your_click_para={clickid}&your_aff_para={appid}&your_camp_para={offerid}&your_blockreason_para={breason}&your_payout_para={payout}&other_fix=fixvalue&other_fix2=fixvalue2&down_para_name={para1}

**postback_url 注解：**

在真实单子中配置时，请将 some.downstream.domain.com、postback 替换为下游的回调主域名和路径；down_click_para、down_aff_para、down_camp_para、down_blockreason_para、down_payout_para 请替换为下游的参数名；other_fix、other_fix2 为可配置的固定回传参数，配置时请替换为下游的固定参数名称；down_para_name 为第 2.1 节中接收到的预留自定义参数，请替换为下游的自定义参数名称。

例如，替换后的 postback_url（填写到数据库中的内容）可以是：

http://www.baidu.com/postback?click={clickid}&aff={app_id}&camp={offer_id}&blockreason={breason}&payout={payout}&goods=fixvalue&update=fixvalue2&third_party={para1}

**Iplayable 网盟平台支持的回调链接宏参数：**

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

**注：** 仅支持以上宏参

Iplayable 的运营人员配置好 downstream 表的 click_pattern 和 postback_url 字段后就完成了对下游的 url 配置对接工作。文档尾部的**附录1**会对与下游实际的交互细节举例说明。 -->

<!-- Iplayable 的运营人员配置好 downstream 表的 click_pattern 和 postback_url 字段后就完成了对下游的 url 配置对接工作。文档尾部的**附录1**会对与下游实际的交互细节举例说明。 -->


## 3. 对接上游/三方

### 3.1 对接上游

#### 3.1.1 offer 拉取 url 配置

支持的宏参：

| Placeholder | Description | 是否必须 |
| --- | ------- | ------- |
| affname | 账户名 | 是 |
| afftoken | 密码 | 是 |
| osup | O小写的操作系统 ios android  | 否 |
| oslo | Offer 大写的操作系统 IOS ANDROID | 否 |
| cc2lo | 2位小写countrycode | 否 |
| cc3lo | 3位小写countrycoded | 否 |
| cc2up | 3位小写countrycode  | 否 |
| cc2up | 3位小写countrycode  | 否 |



**示例：**

假设，上游的域名和路由为 some.upstream.domain.com、getoffer；上游接收账户名的参数名为 aff_id；上游接收密码的参数名为 aff_token；上游接收操作系统平台的参数名为platform，并且接收的参数值是小写的操作系统；上游接收国家的参数名为 countries，并且接收的参数值是2位大写格式，运营需要进行如下配置：

http://some.upstream.domain.com/getoffer?aff_id={affname}&aff_token={afftoken}&platform={oslo}&countries={cc2up}

其中 {oslo}、{cc2up} 实际中需要使用的 placeholder 请与上游沟通后并参考上述表格。

### 3.2 对接三方

现有三方平台主要有 appsflyer 和 adjust，因此我们将其分为两个小节介绍。需要配置的位置为pg数据库的 upstream 表的 postback_pattern 和 postback_event_pattern ，分别对应安装信息回调和后链路事件信息回调，下面将详细说明。

#### 3.2.1 对接 appsflyer

#### 3.2.1.1 配置安装回调 pattern

该项配置的是 upstream 表的 postback_pattern 字段，配置**示例：**

http://callback.flatmobi.com/install/appsflyer?click_id={clickid}&blocked_reason={breason}&blocked_reason_value={bvalue}&blocked_sub_reason={bsub}&install_unix_ts={insts}

* 配置该部分需要与 2.2.1 节配置的 postback_url 对齐，使用相同宏参。例如，postback_url 的查询参数需要回传 {breason} 宏参，这里配置时就必须配置 {breason}。

支持的宏参：

参数名称 | 宏参 | 参数说明 | 是否必须 |
| ----- | --- | ------- | ------- |
| click_id | {clickid} | Iplayable 平台生成的click id | 是 |
| install_unix_ts | {insts} | 安装时间戳，仅支持安装事件回传 | 否 |
| blocked_reason | {breason} | Block reason，仅支持安装事件回传 | 否 |
| blocked_sub_reason | {bsub} | Block sub reason，仅支持安装事件回传 | 否 |
| blocked_reason_value | {bvalue} | Block value，仅支持安装事件回传 | 否 |

* 若下游或者 Iplayable 平台需要回传其他不包含在以上参数中的信息，需要 appsflyer 三方平台支持该参数，并且在 postback_pattern 字段的 url 中添加配置该参数。若是下游需要该额外参数则只需与 2.2.1 节配置的 postback_url 对齐，使用相同的宏参；若是 Iplayable 平台需要该额外参数则需修改代码落到日志。


#### 3.2.1.2 配置后链路事件回调 pattern

该项配置的是 upstream 表的 postback_event_pattern 字段，配置**示例：**

http://callback.flatmobi.com/install/appsflyer?click_id={clickid}&event_name={ename}&event_value={evalue}

* 配置该部分需要与 2.2.2 节配置的 event_postback_url 对齐。例如，event_postback_url 的查询参数需要回传{ename}宏参，这里配置时就必须回调{ename}。

支持的宏参：

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {clickid} | Iplayable 平台生成的click id | 是 |
| {ename} | 后链路事件，注册或者付费等，仅支持后链路事件回传 | 否 |
| {erev} | 后链路事件收益，仅支持后链路事件回传 | 否 |
| {etime} | 后链路事件时间戳，仅支持后链路事件回传 | 否 |
| {evalue} | 后链路事件value，仅支持后链路事件回传 | 否 |

* 若下游或者 Iplayable 平台需要回传其他不包含在以上参数中的信息，需要 appsflyer 三方平台支持该参数，并且在 postback_event_pattern 字段的 url 中添加配置该参数。若是下游需要该额外参数则只需与 2.2.1 节配置的 event_postback_url 对齐，使用相同的宏参；若是 Iplayable 平台需要该额外参数则需修改代码落到日志。

#### 3.2.1.3 配置 callback

配置位置为 appsflyer 三方平台的页面

install callback 示例：

http://callback.flatmobi.com/api/v1/install/appsflyer?site_id={Site ID}&app_id={App ID}&blocked_reason={Blocked reason}&blocked_reason_value={Blocked reason value}&blocked_sub_reason={Blocked sub reason}&bundle_id={Bundle ID}&campaign_id={Campaign ID}&install_unix_ts={Install time}&click_id={Click ID}

event callback 示例：

http://callback.flatmobi.com/api/v1/install/appsflyer?site_id={Site ID}&app_id={App ID}&click_id={Click ID}

#### 3.2.1.3 配置 click_track_url

示例：




#### 3.2.2 对接adjust

#### 3.2.2.1 配置安装回调pattern

该项配置的是 upstream 表的 postback_pattern 字段，配置示例：

http://callback.flatmobi.com/install/adjust?click_id={clickid}&installed_at={insts}&rejection_reason={breason}

* 配置该部分需要与 2.2.1 节配置的 postback_url 对齐。例如，postback_url 的查询参数需要回传{breason}宏参，这里配置时就必须回调{breason}，其他。

支持的宏参：

| 参数名称 | 宏参 | 参数说明 | 是否必须 |
| ------- | --- | ------- | ------- |
| click_id | {clickid} | Iplayable 平台生成的click id | 是 |
| installeed_at | {insts} | 安装时间戳，仅支持安装事件回传 | 否 |
| rejection_reason | {breason} | reject reason，仅支持安装事件回传 | 否 |

#### 3.2.2.2 配置后链路事件回调pattern

该项配置的是 upstream 表的 postback_event_pattern 字段，配置示例：

http://callback.flatmobi.com/install/adjust?click_id={clickid}&event_name={ename}&event_value={evalue}

* 配置该部分需要与 2.2.2 节配置的 event_postback_url 对齐。例如，event_postback_url 的查询参数需要回传{ename}宏参，这里配置时就必须回调{ename}。

支持的宏参：

| 宏参 | 参数说明 | 是否必须 |
| --- | ------- | ------- |
| {clickid} | Iplayable 平台生成的click id | 是 |
| {ename} | 后链路事件，注册或者付费等，仅支持后链路事件回传 | 否 |
| {erev} | 后链路事件收益，仅支持后链路事件回传 | 否 |
| {etime} | 后链路事件时间戳，仅支持后链路事件回传 | 否 |
| {evalue} | 后链路事件value，仅支持后链路事件回传 | 否 |

<!-- ## 附录1：

在与下游正式交互时， Iplayable 平台会根据下游发送的点击以及上游或三方传来的回调数据提取宏参，然后 IPlayable 平台会使用提取的填充 postback url 并使用填充后的 postback url 对下游进行get请求，**示例如下：**

### a. Iplayable 系统配置postback_url，例如：

http://your.domain.com/your/route?your_click_para={clickid}&your_aff_para={app_id}&your_camp_para={offer_id}&your_blockreason_para={breason}&your_payout_para={payout}&other_fix=fixvalue&other_fix2=fixvalue2&your_name={para1}


### b. Iplayable 网盟平台从下游发送的的 Click_url 以及上游或者三方的回调提取宏参，例如：

假设 Iplayable 平台接收到下游发送的 Click_url 如下所示：

http://callback.flatmobi.com/api/v1/click?app_id=1000&offer_id=22&clickid=abc&gaid=4716d154-7232-11ee-800e-e9037848532a&android={android}&idfa={idfa}&subid={subid}&affsub=testabc&para1=somevalue1&para2=somevalue2

假设 Iplayable 平台接收到的上游或者三方的回调如下所示：

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

### c. Iplayable 平台系统使用步骤 b 提取到的宏参填充步骤a中配置的 postback url ，并使用填充后的 url 进行 get 请求，填充结果示例如下：

http://your.domain.com/your/route?your_click_para=abc&your_aff_para=1000&your_camp_para=22&your_blockreason_para=test1&your_payout_para=1.5&&other_fix=fixvalue&other_fix2=fixvalue2&your_name=somevalue1 -->