---

layout: page
category: Protocol
title: 红外转发器接口文档
---

<h1><p align="center"> 红外转发器接口文档 </p></h1>

| Version | Date | Author | Log |
| --- | --- | --- | --- |
| V1.7 | 2018.10.30 | Jason | 重写文档。。。。。。。统一全部红外接口，废弃原红外协议，太多东西，请比对更新 |
| V1.8 | 2018.10.31 | Jason | 增加服务器场景控制的定义定义红外转发器对象及查询修改接口 |
| V1.9 | 2018.11.1 | Jason | 增加空调一键匹配functionTimeout置0为提前cancel |
| V2.0 | 2018.11.5 | Jason | *DeviceInfoList*补充缺失的品牌ID字段*brandId*顺带把及其碍眼又迷糊的t字段改为*deviceType*一键匹配遥控方案<br>绑定码库方案，新建自定义遥控器 补充缺失请求*serialId*字段<br>MQTT返回结果补充*serialId*<br>服务器场景action定义 增加name用于直接显示遥控方案名 |
| V2.1 | 2018.12.18 | Jason | 增加下发码库function定义,空调key&byte定义及拼接定义,服务器增删下载本地方案API |
| V2.2 | 2018.12.27 | Jason | 更改key&byte: v1的风速恒ff<br>示例细分服务器解析byte与实控byte<br>统一按v3控制 |

---
* TOC
{:toc}

[TOC]

---

```单品红外转发器和（obox+红外）方案对于前端来说保持接口和流程的统一：
区别在于学习按键、删除按键（仅有以套为单位删除方式）、确认选取码库、删除码库的时候，服务器根据设备类型决定是否和设备交互（单品不需交互，二合一需要交互）
进入到码库分支时沿用rf时期红外转发器的接口，区别在于:
1. 进入自动对码模式由原先的蓝牙传输指令修改为调用新增服务器设置进入对码模式接口，自动对码模式匹配到的测试码由mqtt返回，格式保持与原先的rf匹配返回测试码一致
2. 手动选择品牌匹配的测试码依然在http中返回. 格式保持与原先的rf匹配返回测试码一致
3. 增加发送测试码接口，用于点击测试码时发送数据给红外转发器
```

# 设备对象

> 红外转发器设备作为WiFi单品形式存在，交互数据为json数据体，与插座单品类似，相关接口仍延续。

<a name="红外设备标准化定义"></a>
## 红外设备标准化定义

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| deviceId | string | 设备唯一id | 红外设备唯一的序列号 |
| name | string | 设备名称 | 入网默认为 IR transponder |
| type | string | 设备类型Hex | 红外设备为\"51\";即对应81类型 |
| online | bool | 是否在线 |   |
| action | jsonArray | 设备具备的能力 | 参考以下定义，由前端激活入网时定义 |
| state | jsonArray | 设备当前状态，与action对应 |   |

## 红外设备功能action定义

` 由前端激活入网时定义 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| functionId | int | 功能点ID | 由1起 |
| function | string | 功能标准化字符串 | 参考以下定义值 |
| functionTag | string | 功能标识 | 参考以下定义值 |
| functionName | string | 功能名称 | 参考以下定义值 |
| dataType | string | 数据类型 | 参考以下定义值 |
| dataTranType | array | 数据传输类型，包含string | upload 或 download |

## 红外设备状态state定义

` 控制转发 或 学习上报 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| functionId | int | 功能点ID | 与action对应 |
| data | 依据action的dataType定义 | 数据 | 当前均为 hexString |

## 定义1: 控制红外转发

` Function1 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 1 | 控制转发 |
| function | string | send |   |
| functionTag | string | control |   |
| functionName | string | send |   |
| dataType | string | raw | 红外发射码为hexString |
| dataTranType | array | [\"download\"] | 只下发 |

` 发送示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 1 | 控制转发 |
| data | hexString | \"11ffeemmdd\" | 红外发射码 |

## 定义2: 学习红外上传

` Function2 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 2 | 学习上传 |
| function | string | receive |   |
| functionTag | string | control |   |
| functionName | string | receive learning |   |
| dataType | string | raw | 接收到的红外编码为hexString |
| dataTranType | array | [\"upload\"] | 只上传 |
| zipType | string | bool | 接受红外编码是否为压缩 |

` 接收上传示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 2 | 学习上传 |
| data | hexString | \"11ffeemmdd\" | 接收的红外码 |

## 定义3: 一键匹配红外上传

` Function3 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 3 | 匹配上传 |
| function | string | receive |   |
| functionTag | string | control |   |
| functionName | string | receive pairing |   |
| dataType | string | raw | 接收到的红外编码为hexString |
| dataTranType | array | [\"upload\"] | 只上传 |

` 接收上传示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 3 | 匹配上传 |
| data | hexString | \"11ffeemmdd\" | 接收的红外码 |

## 定义4: 进入学习接收模式

` Function4 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 4 | 设置学习模式 |
| function | string | learning |   |
| functionTag | string | config |   |
| functionName | string | learning |   |
| dataType | string | int | 学习模式超时时间 /秒 默认90秒 |
| dataTranType | array | [\"upload\",\"download\"] | 下发，回复 |

` 设置示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 4 | 设置学习模式 |
| data | int | 30 | 30秒超时 |

` 回复设置ack示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 4 | 设置学习模式 |
| data | int | 下发的超时时长，0为失效 |   |

## 定义5: 进入一键匹配接收模式

` Function5 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 5 | 设置匹配模式 |
| function | string | pairing |   |
| functionTag | string | config |   |
| functionName | string | pairing |   |
| dataType | string | int | 匹配模式超时时间 /秒 默认90秒 |
| dataTranType | array | [\"upload\",\"download\"] | 下发，回复 |

` 设置示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 5 | 设置匹配模式 |
| data | int | 30 | 30秒超时 |

` 回复设置ack示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 5 | 设置匹配模式 |
| data | int | 下发的超时时长，0为失效 |   |

## 定义6: 设置本地码库增删

` Function6 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 6 | 设置本地码库增删 |
| function | string | depositary |   |
| functionTag | string | config |   |
| functionName | string | depositary |   |
| dataType | string | int | 1.新增方案, 2.删除方案  |
| dataTranType | array | [\"upload\",\"download\"] | 下发，回复 |
| indexType | string | int | 方案索引，非0有效  |

` 设置新增示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 6 | 设置本地码库新增 |
| data | int | 1 | 新增方案 |
| count | int | 100 | 可携带方案信息条数<br>一是用于确定方案大小，二是确定传输完成 |

` 回复新增ack示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 6 | 设置匹配模式 |
| data | int | 1 | 新增方案,失败回复0  |
| index | int | 1 | 本地存储索引，非0! |

> index 为转发器存储的地址，非零，控制存储数量由设备本身flash决定

` 设置删除示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 6 | 设置本地码库删除 |
| data | int | 2 | 删除方案 |
| index | int | 1 | 存储索引 |

` 回复删除ack示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 6 | 设置匹配模式 |
| data | int | 2 | 删除方案,失败回复0  |
| index | int | 1 | 存储索引 |

## 定义7: 下发本地码库数据

` Function7 `

| Action Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 7 | 下发本地码库数据 |
| function | string | codeDownload |   |
| functionTag | string | codeInfo |   |
| functionName | string | codeDownload |   |
| dataType | string | raw | src码  |
| dataTranType | array | [\"download\"] | 下发 |
| keyType | string | raw | 对应索引 |
| indexType | string | int | 方案索引  |
| countType | string | int | 方案信息索引 |

` 下发单条码库示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 7 | 下发本地码库 |
| data | raw | aabbccddee | src码数据 |
| index | int | 1 | 存储索引1 |
| count | int | 3 | 方案下发第3条信息 |
| key | raw | on / 01ffffffff | 按键描述索引 或 7Byte索引 |

` 回复下发ack示例 `

| State Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| functionId | int | 7 | 下发本地码库 |
| data | raw | 1 | 下发成功，0为失败，服务器此时需重传 |
| index | int | 1 | 存储索引1 |
| count | int | 3 | 方案下发第3条信息 |

# 设备接口

## 红外设备信息上传

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **upload\_config** |   |
| deviceName | string | 产品名字 | 从regist\_aliDev来 |
| productKey | string | 产品key | 从regist\_aliDev来 |
| config | string | [标准化设备定义](#红外设备标准化定义) |   |

## 查询红外设备

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **query\_ali\_dev** |   |
| **Response Key** |   |   |   |
| configs | jsonArray | 标准化设备定义数组 | 里面包含一切wifi单品设备，区分设备类型 |

## 删除设备（服务器需清空相应的码库表）

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **delete\_ali\_dev** |   |
| deviceId | String | 产品序列号 |   |

## <del>设置设备状态 （红外前端不会调用此接口）

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **set\_ali\_dev** |   |
| deviceId | String | 产品序列号 |   |
| value | jsonArray | state对象数组 |   |
| **Response Key** |   |   |   |
| value | jsonArray | 当前state对象数组 |   |

## <del>读取设备状态 （红外前端不会调用此接口）

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | read\_ali\_dev |   |
| deviceId | String | 产品序列号 |   |
| functionId | jsonArray | 包含功能ID数组 |   |
| **Response Key** |   |   |   |
| value | jsonArray | 对应state对象数组 |   |

# 业务对象

<a name="DeviceTypeList"></a>
## DeviceTypeList 遥控设备类型单元

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| t | int Enum | 被遥控设备类型ID | 1 机顶盒<br> 2电视<br> 3DVD<br> 5投影仪<br> 6风扇<br> 7空调<br> 8智能灯<br> 10互联网机顶盒<br> 12扫地机<br> 13音响<br> 15空气净化器<br> 0其他(自定义学习) |
| name | string | 被遥控设备类型名字 |   |

<a name="DeviceBrandList"></a>
## DeviceBrandList 遥控设备品牌单元

| Key | Value Type | Descriotion | Mark |
| --- | --- | --- | --- |
| bid | int | 品牌ID | 按遥控云的标准 |
| common | int<br>@optional | 常用品牌标识 |   |
| name | string | 品牌名字 | 按遥控云的标准 |

<a name="DeviceTypeList"></a>
## <mark>DeviceInfoList 遥控设备方案单元

` 遥控器对象，增加key和extends，区别以前协议 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| index | int | 遥控器索引ID | 非0，新增时调用其他接口 |
| name | string | 遥控器名 |   |
| deviceType | int | [遥控设备类型](#DeviceTypeList)ID | 与码库方案保持一致 |
| brandId | int | [遥控品牌类型](#DeviceBrandList)ID | 与码库方案保持一致 |
| keys | jsonArray<br>*CommonKeyList* | [标准按键业务](#CommonKeyList)对象列表  | 1. 类型非0（非其他），设备有既定规范页面，根据key判断按键是否可控 <br> 2. 类型0的自定义学习，只有空列表|
| extendsKeys | jsonArray<br>*ExtendKeyList* | [拓展按键业务](#ExtendKeyList)对象列表  | 定义类型0（其他）， 以及非0的更多拓展按键（注意其生成规则） |
| rid | string<br>@optional | 遥控云码库方案ID | 仅匹配有效，用于匹配遥控云码库 |
| rc\_command | jsonArray<br>@optional | 码库rc\_command数据单元数组<br>云需对此解析成相应的*CommonKeyList* | 仅匹配的方案有效 |
| rmodel | string<br>@optional | 遥控器型号 | 仅匹配的方案有效 |
| version | string<br>@optional | 遥控器版本 | 仅匹配的方案有效 |
| localaddr | int<br>@optional | 遥控器本地存储索引，非0有效 | 仅本地存储方案有效 |

<a name="CommonKeyList"></a>
## CommonKeyList标准按键业务对象

` 页面标准按键与云端建立对应关系的依据，预留给更多键值扩展 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| key | string | 按键的**唯一**描述用于控制及更改发射码参照遥控云<br>根据不同设备有不同规则 | 在开发过程中总结出json模板供统一使用 |

<a name="ExtendKeyList"></a>
## ExtendKeyList拓展按键业务对象

` 拓展按键与云端建立对应关系的依据，预留给更多键值扩展 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| key | string | 拓展按键的名称用于控制、删除、重命名前端数据截取\"`|`\"前用户输入的功能名用于显示 | 1. 根据**唯一**的extendsKey进行列表显示<br>2. 键名生成规则为：用户输入的功能键名 + \"`|`" + 时间戳From UTC 2001/1/1 00:00:00（秒),生成规则由前端控制，以保证数据唯一|

## rc\_command数据单元（云与遥控云间处理key的映射关系）

` 码库命令数据，参考原协议，仅用于匹配的遥控器 `

| Key | Value Type | Description | Mark |
| --- | --- | --- | --- |
| kn | String | 国际化键显示名 |   |
| src        | String | 原始码 |   |

# 业务接口

``` 
基本请求参数 Post: hostname/consumer/common?
注意：无@optional标志皆为必须项，且请求成功的回复均不可有null！ 
```

| access\_token | String | 访问口令token |
| --- | --- | --- |
| 基本回复参数 |
| success | int | http\_code请求成功与否 |

## 获取遥控云遥控类型

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | String | **query\_ir\_device\_type** |   |
| **Response Key** |   |   |   |
| rs | jsonArray | [DeviceTypeList](#DeviceTypeList)对象数组 | 遥控类型 |

## 获取遥控云品牌类型

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | String | **query\_ir\_brand** |   |
| deviceType | int | DeviceTypeList中类型ID |   |
| **Response Key** |   |   |   |
| rs | jsonArray | [DeviceBrandList](#DeviceBrandList)对象数组 | 对应遥控类型的品牌 |

## 获取红外遥控方案

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **query\_ir\_device** |   |
| serialId | string | 红外转发器序列号 |   |
| **Response Key** |   |   |   |
| rs | jsonArray | [DeviceInfoList](#DeviceInfoList)对象数组 | 转发器下对应的遥控方案 |

<a name="控制转发命令"></a>
## 控制转发命令

` 根据方案索引及按键Key查询转发的控制命令，发给转发器 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **control\_ir\_device** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |
| keyType | int | 0:标准按键<br>1:拓展按键<br>2:手动匹配测试按键 | 按键类型<br>对于2仅用于发送手动匹配的测试码库按键 |
| key | string | [标准按键](#CommonKeyList)或[拓展按键](#ExtendKeyList)的按键名称key | **唯一匹配**按键准则，如方案中不存在，返回失败 |

## 删除红外遥控方案

` 删除遥控方案，同时解绑匹配的遥控云码库 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **delete\_ir\_device** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | [DeviceInfoList](#DeviceInfoList)中遥控索引ID | 匹配方案 |

> 如此方案已下载至转发器内，需删除

## 重命名红外遥控方案

` 重命名遥控方案 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **rename\_ir\_device** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | [DeviceInfoList](#DeviceInfoList)中遥控索引ID | 匹配方案 |
| name | string | 命名遥控器方案 |  |

## 删除方案中特定按键

` 根据方案索引及Key删除特定按键 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **delete\_ir\_device\_key** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |
| keyType | int | 0:标准按键<br>1:拓展按键 | 按键类型 <u>一般不建议删除通过匹配码库生成的方案中的标准按键</u> |
| key | string | [标准按键](#CommonKeyList)或[拓展按键](#ExtendKeyList)的按键名称key | 唯一匹配按键准则，<mark>如方案中不存在，返回成功</mark> |

## 手动匹配遥控方案——测试码获取

` 根据品牌id，设备类型主动数据匹配 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **query\_ir\_testcode** |   |
| deviceType | int | DeviceTypeList中类型ID | 匹配类型 |
| brandId | int | DeviceBrandList中品牌ID | 匹配品牌 |
| serialId | string | 红外转发器序列号 |   |
| **Response Key** |   |   |   |
| rs | jsonArray | *[DeviceInfoList](#DeviceInfoList)*对象数组（<mark>仅用于匹配</mark>的对象） | 遥控云匹配测试的方案，rid, rmodel, version为必须 |

> 之后使用[控制转发命令](#控制转发命令)发送测试码库<br>完成测试后，使用[绑定码库方案](#绑定码库方案)完成方案的新建

## 一键匹配遥控方案——进入空调对码模式

` 设置某红外转发器进入空调一键对码模式，仅用于设备类型为空调(7) `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **pair\_ir\_remotecode** |   |
| serialId | string | 红外转发器序列号 |   |
| brandId | int | DeviceBrandList中品牌ID | 匹配空调品牌 |
| timeout | int | 超时时间/秒置0为提前取消 | 红外转发器只在此时间内处于接收原始码状态 |

> 首先按标准http返回是否成功进入一键对码模式
> <br>等待转发器上报接收到的遥控器码并将匹配到的[测试方案通过MQTT返回](#MQTT返回一键匹配)
> <br>之后使用[控制转发命令](#控制转发命令)发送测试码库<br>完成测试后，使用[绑定码库方案](#绑定码库方案)完成方案的新建

<a name="绑定码库方案"></a>
## 手动匹配/一键匹配遥控方案——绑定码库方案

` 完成手动匹配测试码库，绑定生成新的遥控方案对象 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **bind\_ir\_remotecode** |   |
| serialId | string | 红外转发器序列号 |   |
| deviceType | int | DeviceTypeList中类型ID | 匹配类型 |
| brandId | int | DeviceBrandList中品牌ID | 匹配品牌 |
| remoteId | string | 匹配的测试码库rid | 遥控云码库id |
| name | string@optional | 命名匹配成功的遥控器方案 | 可选，默认为类型加品牌名 |
| **Response Key** |   |   |   |
| remote | jsonDic | *[DeviceInfoList](#DeviceInfoList)对象* | **新建的**匹配成功遥控方案 |

## 学习遥控方案——新建自定义遥控器

` 创建自定义的遥控器，用于学习复制遥控器 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **create\_ir\_device** |   |
| serialId | string | 红外转发器序列号 |   |
| deviceType | int | DeviceTypeList中类型ID | 学习的设备类型 |
| brandId | int@optional | DeviceBrandList中品牌ID | 学习的品牌，如无置0 |
| name | string | 命名自定义遥控器方案 | 用户自定义输入，默认为自定义遥控器+类型名 |
| **Response Key** |   |   |   |
| remote | jsonDic | *[DeviceInfoList](#DeviceInfoList)对象* | **新建的**自定义遥控方案 |

## 学习遥控方案——进入按键学习模式

` 设置某红外转发器进入按键学习模式 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **learn\_ir\_device\_key** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |
| keyType | int | 0：标准按键1：拓展按键 | 按键类型 |
| key | string | [标准按键](#CommonKeyList)或[拓展按键](#ExtendKeyList)的按键名称key | **唯一**匹配按键准则，<mark>如方案中存在即为替换，否则为新增学习</mark>，拓展按键遵循key命名规则 |
| timeout | int | 超时时间/秒<br>置0为提前取消 | 红外转发器只在此时间内处于接收原始码状态 |

> 首先按标准http返回是否成功进入按键学习模式
> <br>等待转发器上报接收到的遥控器码并将[更新后的方案通过MQTT返回](#MQTT返回按键学习)

## 本地遥控方案——下载方案

` 下载码库方案至红外转发器 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **local\_ir\_device\_download** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |
| timeout | int | 超时时间/秒<br>置0为提前取消 | 红外转发器只在此时间内处于下载方案状态<br>约定为120s |

> 首先按标准http返回是否成功进入方案下载模式
> <br>等待转发器接收完完整方案并[通过MQTT返回下载方案结果](#MQTT返回方案下载结果)

## 本地遥控方案——删除方案

` 删除红外转发器中存储的方案 `

| Request Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| CMD | string | **local\_ir\_device\_delete** |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |

<a name="MQTT返回按键学习"></a>
## MQTT返回按键学习结果

` 确认按键学习结果 `

| **Response Key** | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| type | int | **20** | MQTT数据类型 |
| success | bool | 学习结果 |   |
| serialId | string | 红外转发器序列号 |   |
| remote | jsonDic | DeviceInfoList对象，失败无此键值 | 学习更新后的完整遥控方案 |

<a name="MQTT返回一键匹配"></a>
## MQTT返回一键匹配测试码库结果

` 确认按键学习结果 `

| **Response Key** | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| type | int | **21** | MQTT数据类型 |
| success | bool | 一键对码结果 |   |
| serialId | string | 红外转发器序列号 |   |
| rs | jsonArray | DeviceInfoList对象数组（仅用于匹配的对象） | 遥控云匹配测试的方案，rid, rmodel, version为必须 |

<a name="MQTT返回方案下载结果"></a>
## MQTT返回机内方案下载结果

` 确认按键学习结果 `

| **Response Key** | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| type | int | **22** | MQTT数据类型 |
| success | bool | 下载结果 |   |
| serialId | string | 红外转发器序列号 |   |
| index | int | DeviceInfoList中遥控索引ID | 下载的方案 |
| localaddr | int | 存储索引 | 转发器内方案存储索引 |

# 服务器场景action定义

` 红外转发器服务器场景中action格式 `

| Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| node\_type | string | **07** | 识别为红外转发器action |
| serialId | string | 红外转发器序列号 |   |
| action | jsonString | 以下序列化单元经json序列化为字符串 | 服务器反解析为map经控制接口直接触发转发器动作 |

` action的jsonString序列化对象单元 `

| Key | Value Type | Value | 说明 |
| --- | --- | --- | --- |
| index | int | DeviceInfoList中遥控索引ID | 匹配方案 |
| name | string | DeviceInfoList中遥控器名 | 用于界面直接显示 |
| keyType | int | 0:标准按键<br>1:拓展按键 | 按键类型 |
| key | string | 标准按键或拓展按键的按键名称key | 唯一匹配按键准则 |

# 空调Key&Bytes定义

` 空调Key枚举定义及7-bytes位定义`

## Key枚举定义

`v1为version==1，v3为version==3`

<a name="IRACMode"></a>
`模式定义`

| Mode | KeyString | byteHex | 说明 |
| --- | --- | --- | --- |
| 关机 | off | 0x00 | 直接定义 |
| 开机 | on | 0x01 | 直接定义<br>对应初始化制冷26度<br>如v1的“ar26”,v3的“r\_s0\_26\_u1\_l1\_p0” |
| 自动 | a | 0x11 | 拼接定义，无温度 |
| 制冷 | r | 0x21 | 拼接定义 |
| 抽湿 | d | 0x31 | 拼接定义，无温度 |
| 送风 | w | 0x41 | 拼接定义，v1无温度 |
| 制热 | h | 0x51 | 拼接定义，部分无制热功能 |

<a name="IRACTemp"></a>
`温度定义`

| Temp | KeyString | byteHex | 说明 |
| --- | --- | --- | --- |
| 16 | 16 | 0x10 | 拼接定义 |
| 17 | 17 | 0x11 | 拼接定义 |
| …… | …… | …… | …… |
| 29 | 29 | 0x1d | 拼接定义 |
| 30 | 30 | 0x1e | 拼接定义 |
| 无效 | _ | 0xff | 拼接定义,置空 |

<a name="IRACSpeed"></a>
`风量定义`

| Speed | KeyString | byteHex | 说明 |
| --- | --- | --- | --- |
| 自动 | a | 0x00 | key仅v1定义，拼接定义<br>v1固定自动 |
| 自动 | s0 | 0x00 | key仅v3定义，拼接定义<br>送风无自动 |
| 1档 | s1 | 0x01 | 仅v3定义，拼接定义<br>抽湿仅1档 |
| 2档 | s2 | 0x02 | 仅v3定义，拼接定义 |
| 3档 | s3 | 0x03 | 仅v3定义，拼接定义 |
| 无效 | _ | 0xff | 占位预留,**v1固定为ff**，v3必须定义 |

<a name="IRACUpSwing"></a>
`上下摆风定义`

| UpSwing | KeyString | byteHex | 说明 |
| --- | --- | --- | --- |
| 开启 | u1 | 0x01 | 仅v3定义，拼接定义 |
| 关闭 | u0 | 0x00 | 仅v3定义，拼接定义 |
| 无效 | _ | 0xff | v1无效，v3按需定义 |

<a name="IRACLeftSwing"></a>
`左右摆风定义`

| LeftSwing | KeyString | byteHex | 说明 |
| --- | --- | --- | --- |
| 开启 | l1 | 0x01 | 仅v3定义，拼接定义 |
| 关闭 | l0 | 0x00 | 仅v3定义，拼接定义 |
| 无效 | _ | 0xff | v1无效，v3按需定义 |

<mark>v3中有部分key包含摆风独立key: `*_*_*_u0_*_*`，暂定为v2，目前按不支持处理，即拼接规则置空摆风位`

## Key拼接定义

`KeyString: `

| v1 | |
| --- | --- |
| off | 关机 |
| on | 开机 |
| [{Speed}](#IRACSpeed) + [{Mode}](#IRACMode) + ([{Temp}](#IRACTemp)) | 拼接 |

| v3 | |
| --- | --- |
| off | 关机 |
| on | 开机 |
| [{Mode}](#IRACMode) + `_` + [{Speed}](#IRACSpeed) + `_` + [{Temp}](#IRACTemp) + `_` + [{UpSwing}](#IRACUpSwing) + `_` + [{LeftSwing}](#IRACLeftSwing) + `_p0` | 拼接 |

`Bytes: `

| byte0 | byte1 | byte2 | byte3 | byte4 | byte5 | byte6 |
| --- | --- | --- | --- | --- | --- | --- |
| [{Mode}](#IRACMode) | [{Speed}](#IRACSpeed) | [{Temp}](#IRACTemp) | [{UpSwing}](#IRACUpSwing) | [{LeftSwing}](#IRACLeftSwing) | 方案索引 | 预留(版本识别) |

`byte5(方案索引): 索引本地方案地址，预留给多方案设备，通过读取状态确定，当前线控器仅方案01`

`byte6(版本识别): v1:0x01, v2:0x02, v3:0x03, v2为v3的不支持摆风版本`

## 示例

<mark>服务器需按上述定义将**相应版本**的key解析为byte，前端则**统一按v3**的逻辑组成byte用于控制

| 定义 | Key | 服务器解析Bytes<br>(区分版本) | 线控按键及App控制Bytes<br>(统一为v3规则) |
| --- | --- | --- | --- |
| 关机 | “off” | All: 00ffffffff+{index}+{version} | 00ffffffff+{index}+03 |
| 开机 | “on” | All: 01ffffffff+{index}+{version} | 01ffffffff+{index}+03 |
| 自动,风速自动,上下左右摆风 | v1:“aa”<br>v3:"a\_s0\_\_u1\_l1\_p0"<br>v2:"a\_s0\_\_\_\_p0" | v1: 11ffffffff+{index}+01<br>v3: 1100ff0101+{index}+03<br>v2: 1100ffffff+{index}+02 | 1100ff0101+{index}+03  |
| 制冷,风速自动, 20度 | v1:“ar20”<br>v3:"r\_s0\_20\_u0\_l0\_p0"<br>v2:"r\_s0\_20\_\_\_p0" | 21ff14ffff+01+01<br>2100140000+01+03<br>210014ffff+01+02 | 2100140000+01+03 |
| 制冷,风速1档, 26度，上下摆 | “ar26”<br>"r\_s1\_26\_u1\_l0\_p0"<br>"r\_s1\_26\_\_\_p0" | 21ff1affff+0101<br>21011a0100+0103<br>21011affff+0102 | 21011a0100+0103 |
| 抽湿,风速1档, 左右摆 | “ad”<br>"d\_s1\_\_u0\_l1\_p0"<br>"d\_s1\_\_\_\_p0" | 31ffffffff+0101<br>3101ff0001+0103<br>3101ffffff+0102 | 3101ff0001+0103 |
| 送风,风速2档, 26度上下左右摆风 | “aw”<br>"w\_s2\_26\_u1\_l1\_p0"<br>"w\_s2\_26\_\_\_p0" | 41ffffffff+0101<br>41021a0101+0103<br>41021affff+0102 | 41021a0101+0103 |
| 制热,风速3档, 28度，左右摆风 | “ah28”<br>"h\_s3\_28\_u0\_l1\_p0"<br>"h\_s3\_28\_\_\_p0" | 51ff1cffff+0101<br>51031c0001+0103<br>51031cffff+0102 | 51031c0001+0103 |

<mark>硬件处理控制bytes时，**不应全匹配！**，须从byte0到byte4逐个字节比对服务器下发的索引，如该字节下发的索引值为0xff，则匹配该字节上的任意控制字符，直到确定唯一的索引！

```
如服务器下发的v1控制命令有：
01ffffffff+01+01 —— ON
21ff14ffff+01+01 —— 制冷,风速自动, 20度
当前端按v3控制发送开机，01ffffffff+01+03 仅匹配byte0确定为ON
当前端按v3控制发送 制冷,风速自动, 20度，上下摆 即2100140100+01+03,
其中对于byte1, byte3, byte4 都为0xff, 则直接忽略该位，即匹配得21ff14ffff，取得相应方案
```


