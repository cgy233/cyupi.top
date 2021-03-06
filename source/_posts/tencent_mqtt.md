---
title: 腾讯物联网开发平台
date: 2021-12-30 12:49:11
categories: 
- 物联网
tags:
- 物联网
---
# 物联网开发平台 SDK

物联网开发平台 SDK 是基于[物理网通信 SDK](https://github.com/tencentyun/qcloud-iot-sdk-embedded-c)的基础能力上，对数据的进一步的抽象封装，形成[数据模板协议](https://cloud.tencent.com/document/product/1081/34916?!preview&!editLang=zh)。平台基于数据模板的协议，提供丰富的数据展示、调试及应用开发等技术资源，将应用端和设备的开发模板化，加速产品开发进程。

# 快速开始

本节将讲述如何在物联网开发平台制台申请设备,创建模板， 并结合SDK 快速体验设备基于数据模板的数据收发、事件上报及设备调试。

## 一. 控制台创建设备

#### 1. 注册/登录腾讯云账号

访问[腾讯云登录页面](https://cloud.tencent.com/login?s_url=https%3A%2F%2Fcloud.tencent.com%2F), 点击[立即注册](https://cloud.tencent.com/register?s_url=https%3A%2F%2Fcloud.tencent.com%2F), 免费获取腾讯云账号，若您已有账号，可直接登录。

#### 2. 访问物联网开发控制台

登录后点击右上角控制台，进入控制台后，鼠标悬停在云产品上，弹出层叠菜单，点击物联网开发平台。

或者直接访问[物联网开发平台控制台](https://console.cloud.tencent.com/iotexplorer)

#### 3. 创建产品和设备

3.1 创建项目
![](https://main.qcloudimg.com/raw/50dd24cfa44fbb063d4337eb94ae5d0f.jpg)

3.2.创建并选择和产品比较相近的模板产品，参阅[产品定义](https://cloud.tencent.com/document/product/1081/34739?!preview&!editLang=zh)。
![](https://main.qcloudimg.com/raw/c8bd19dee90765762bd6c1f98bd0dd2c.jpg)
![](https://main.qcloudimg.com/raw/4b9fe8f5df7a20ebbab1f0e3693ded8a.jpg)

3.3 定义产品的数据和事件模板，参阅[数据模板创建](https://cloud.tencent.com/document/product/1081/34739?!preview&!editLang=zh#.E6.95.B0.E6.8D.AE.E6.A8.A1.E6.9D.BF)，数据模板的说明参见[数据模板协议](https://cloud.tencent.com/document/product/1081/34916?!preview&!editLang=zh)。
![](https://main.qcloudimg.com/raw/17ef8daac4da6f9775ea02bddf988ca2.jpg)

3.4 完成产品创建和数据模板定义后，创建设备，则每一个创建的设备都具备这个产品下的数据模板属性，如下图示。
![](https://main.qcloudimg.com/raw/937698b945dfd2c4f34bffd40ff5165d.jpg)

3.5 查询产品和设备信息
![](https://main.qcloudimg.com/raw/13972a011bb5382d00d73545534af91a.jpg)
![](https://main.qcloudimg.com/raw/17ef8daac4da6f9775ea02bddf988ca2.jpg)

3.6 导出数据模板json文件，并通过脚本工具生成数据模板的配置文件
![](https://main.qcloudimg.com/raw/0951d7c3f540ca716442e08651a0efa5.jpg)

将下载的json文件拷贝到tools目录，执行./codegen.py -c xx/config.json -d  ../targetdir/ 命令,则会根据json文件在target目录生成所定义产品的数据模板及事件的配置文件：

```bash
./codegen.py -c light.json -d ../samples/data_template/
加载 light.json 文件成功
文件 ../samples/data_template/data_config.c 生成成功
文件 ../samples/data_template/events_config.c 生成成功
```

## 二. 编译示例程序

#### 1. 下载SDK

登录 Linux, 运行如下命令从 github 克隆代码, 或者访问最新[下载](https://github.com/tencentyun/qcloud-iot-sdk-embedded-c/releases)地址, 将下载到的压缩包在 Linux 上解压缩

`git clone https://github.com/tencentyun/qcloud-iot-sdk-embedded-c.git`

#### 2. 填入设备信息

src/platform/os/linux/HAL_OS_linux.c 提供了设备信息读写HAL层接口，量产产品需实现设备信息在非易失存储介质的读写接口，即HAL层接口适配。为方便调试，使能该文件调试宏 **DEBUG_DEV_INFO_USED**，则设备信息读写的HAL层接口会读取调试的设备信息，编辑 src/platform/os/linux/HAL_OS_linux.c 文件中如下代码段, 填入之前创建产品和设备步骤中得到的 **产品ID**，**设备名称**，若使能了动态注册功能，需要赋值**产品密钥**：

```c
/* 产品名称, 与云端同步设备状态时需要  */
static char sg_product_id[MAX_SIZE_OF_PRODUCT_ID + 1]  = "YOUR_PRODUCT_ID";
/* 产品密钥, 与云端同步设备状态时需要  */
static char sg_product_secret[MAX_SIZE_OF_PRODUCT_KEY + 1]  = "YOUR_PRODUCT_SECRET";
/* 设备名称, 与云端同步设备状态时需要 */
static char sg_device_name[MAX_SIZE_OF_DEVICE_NAME + 1]  = "YOUR_DEVICE_NAME";
```

1. 若使用**证书认证**加密方式，赋值客户端证书文件名和私钥文件名 **sg_device_cert_file_name** 和 **sg_device_privatekey_file_name** 并将文件放置在根目录下 certs 目录中。将根目录下 make.settings 文件中的配置项 FEATURE_AUTH_MODE 设置为 CERT，FEATURE_AUTH_WITH_NOTLS 设置为 n。

```bash
FEATURE_AUTH_MODE        = CERT   # MQTT/CoAP接入认证方式，使用证书认证：CERT；使用密钥认证：KEY
FEATURE_AUTH_WITH_NOTLS  = n      # 接入认证是否不使用TLS，证书方式必须选择使用TLS，密钥认证可选择不使用TLS
```

2. 若使用**秘钥认证**加密方式，赋值 **sg_device_secret**。将根目录下 make.settings 文件中的配置项 FEATURE_AUTH_MODE 设置为 KEY，FEATURE_AUTH_WITH_NOTLS 设置为 n 时通过 TLS 密钥认证方式连接，设置为 y 时，则通过 HMAC-SHA1 加密算法连接。

```c
#ifdef AUTH_MODE_CERT
/* 客户端证书文件名  非对称加密使用, TLS 证书认证方式*/
static char sg_device_cert_file_name[MAX_SIZE_OF_DEVICE_CERT_FILE_NAME + 1]      = "YOUR_DEVICE_NAME_cert.crt";
/* 客户端私钥文件名 非对称加密使用, TLS 证书认证方式*/
static char sg_device_privatekey_file_name[MAX_SIZE_OF_DEVICE_KEY_FILE_NAME + 1] = "YOUR_DEVICE_NAME_private.key";
#else
/* 设备密钥, TLS PSK认证方式*/
static char sg_device_secret[MAX_SIZE_OF_DEVICE_SERC + 1] = "YOUR_IOT_PSK";
#endif
```

#### 3. 基于数据模板的业务逻辑开发

数据模板示例data_template_sample.c，已实现数据、事件收发及响应的通用处理逻辑。但是具体的数据处理的业务逻辑需要用户自己根据业务逻辑添加，上下行业务逻辑添加的入口函数分别为deal_up_stream_user_logic 、deal_down_stream_user_logic，如果有字符串或json类型的数据模板，用户需要在函数update_self_define_value中完成数据的解析（其他数据类型会根据模板定义自动更新），可以参考基于场景的示例light_data_template_sample.c添加业务处理逻辑。

下行业务逻辑实现：

```bash
/*用户需要实现的下行数据的业务逻辑,pData除字符串变量已实现用户定义的所有其他变量值解析赋值，待用户实现业务逻辑*/
static void deal_down_stream_user_logic(ProductDataDefine   * pData)
{
 Log_d("someting about your own product logic wait to be done");
}
```

上行业务逻辑实现：

```c
/*用户需要实现的上行数据的业务逻辑,此处仅供示例*/
static int deal_up_stream_user_logic(DeviceProperty *pReportDataList[], int *pCount)
{
 int i, j;
 
     for (i = 0, j = 0; i < TOTAL_PROPERTY_COUNT; i++) {       
        if(eCHANGED == sg_DataTemplate[i].state) {
            pReportDataList[j++] = &(sg_DataTemplate[i].data_property);
   sg_DataTemplate[i].state = eNOCHANGE;
        }
    }
 *pCount = j;

 return (*pCount > 0)?AT_ERR_SUCCESS:AT_ERR_FAILURE;
}
```

#### 4. 编译 SDK 产生示例程序

在根目录执行

```bash
make clean
make
```

编译成功完成后, 生成的样例程序在当前目录的 output/release/bin 目录下，其中data_template_sample、event_sample、light_data_template_sample为物联网开发平台示例，其他为物联网通信平台示例。

```bash
output/release/bin/
├── data_template_sample
├── event_sample
├── light_data_template_sample
├── certs
│   ├── README.md
│   ├── TEST_CLIENT_cert.crt
│   └── TEST_CLIENT_private.key
├── coap_sample
├── door_coap_sample
├── door_mqtt_sample
├── gateway_sample
├── mqtt_sample
├── ota_mqtt_sample
└── shadow_sample
```

## 三. 运行示例程序

#### 1. 执行 data_template_sample 示例程序

数据模板示例实现通用的数据模板和事件处理的框架，数据上下行和事件上报的日志如下：

```bash
 ./output/release/bin/data_template_sample 
INF|2019-05-16 19:43:47|device.c|iot_device_info_set(65): SDK_Ver: 3.0.0, Product_ID: 4BZCJGQ1U1, Device_Name: dev1
DBG|2019-05-16 19:43:47|HAL_TLS_mbedtls.c|HAL_TLS_Connect(204): Connecting to /4BZCJGQ1U1.iotcloud.tencentdevices.com/8883...
DBG|2019-05-16 19:43:48|HAL_TLS_mbedtls.c|HAL_TLS_Connect(209): Setting up the SSL/TLS structure...
DBG|2019-05-16 19:43:48|HAL_TLS_mbedtls.c|HAL_TLS_Connect(251): Performing the SSL/TLS handshake...
INF|2019-05-16 19:43:48|HAL_TLS_mbedtls.c|HAL_TLS_Connect(269): connected with /4BZCJGQ1U1.iotcloud.tencentdevices.com/8883...
INF|2019-05-16 19:43:48|mqtt_client.c|IOT_MQTT_Construct(114): mqtt connect with id: E8J7b success
DBG|2019-05-16 19:43:48|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$log/operation/result/4BZCJGQ1U1/dev1|packet_id=53424|pUserdata=(null)
DBG|2019-05-16 19:43:48|log_mqtt.c|_log_mqtt_event_handler(97): subscribe success, packet-id=53424
DBG|2019-05-16 19:43:48|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$log/operation/4BZCJGQ1U1/dev1|payload={"type": "get_log_level", "clientToken": "4BZCJGQ1U1-1"}
DBG|2019-05-16 19:43:49|log_mqtt.c|_log_level_sub_cb(60): Recv Msg Topic:$log/operation/result/4BZCJGQ1U1/dev1, payload:{"clientToken":"4BZCJGQ1U1-1","log_level":0,"result":0,"timestamp":1558007029,"type":"get_log_level"}

WRN|2019-05-16 19:43:49|log_mqtt.c|_log_level_sub_cb(68): Upload log level change to: 0
DBG|2019-05-16 19:43:49|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$template/operation/result/4BZCJGQ1U1/dev1|packet_id=53425|pUserdata=(null)
DBG|2019-05-16 19:43:49|shadow_client.c|_shadow_event_handler(63): shadow subscribe success, packet-id=53425
INF|2019-05-16 19:43:49|data_template_sample.c|event_handler(95): subscribe success, packet-id=53425
INF|2019-05-16 19:43:49|shadow_client.c|IOT_Shadow_Construct(173): Sync device data successfully
INF|2019-05-16 19:43:49|data_template_sample.c|main(293): Cloud Device Construct Success
DBG|2019-05-16 19:43:49|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$thing/down/event/4BZCJGQ1U1/dev1|packet_id=53426|pUserdata=(null)
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=time registered.
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=float registered.
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=power_switch registered.
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=color registered.
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=brightness registered.
INF|2019-05-16 19:43:49|data_template_sample.c|_register_data_template_property(246): data template property=name registered.
INF|2019-05-16 19:43:49|data_template_sample.c|main(314): Register data template propertys Success
DBG|2019-05-16 19:43:49|shadow_client.c|IOT_Shadow_Get(384): GET Request Document: {"clientToken":"4BZCJGQ1U1-0"}
DBG|2019-05-16 19:43:49|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$template/operation/4BZCJGQ1U1/dev1|payload={"type":"get", "clientToken":"4BZCJGQ1U1-0"}
DBG|2019-05-16 19:43:49|shadow_client.c|_shadow_event_handler(63): shadow subscribe success, packet-id=53426
INF|2019-05-16 19:43:49|data_template_sample.c|event_handler(95): subscribe success, packet-id=53426
DBG|2019-05-16 19:43:49|shadow_client.c|_update_ack_cb(114): requestAck=0
DBG|2019-05-16 19:43:49|shadow_client.c|_update_ack_cb(117): Received Json Document={"clientToken":"4BZCJGQ1U1-0","payload":{"metadata":{"reported":{"brightness":{"timestamp":1558002454249},"color":{"timestamp":1558002454249},"name":{"timestamp":1558002454249},"power_switch":{"timestamp":1558002454249}}},"state":{"reported":{"brightness":0,"color":0,"name":"dev1","power_switch":0}},"timestamp":1558002454249,"version":34},"result":0,"timestamp":1558007029,"type":"get"}

DBG|2019-05-16 19:43:49|data_template_sample.c|main(364): No delta msg received...
DBG|2019-05-16 19:43:49|data_template_sample.c|main(394): No device data need to be reported...
DBG|2019-05-16 19:43:49|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$thing/up/event/4BZCJGQ1U1/dev1|payload={"method":"events_post", "clientToken":"4BZCJGQ1U1-1", "events":[{"eventId":"all_function", "type":"alert", "timestamp":1558007029000, "params":{"bool":0,"int":1,"str":"","float":0.000000,"enum1":1,"time":0}},{"eventId":"status_report", "type":"info", "timestamp":1558007029000, "params":{"status":0,"message":""}},{"eventId":"hardware_fault", "type":"fault", "timestamp":1558007029000, "params":{"name":"","error_code":0}}]}
INF|2019-05-16 19:43:52|qcloud_iot_event.c|_on_event_reply_callback(96): Receive Message With topicName:$thing/down/event/4BZCJGQ1U1/dev1, payload:{"method":"events_reply","clientToken":"4BZCJGQ1U1-1","code":404,"status":"all_function not defined","data":{}}
DBG|2019-05-16 19:43:52|qcloud_iot_event.c|_on_event_reply_callback(115): eventToken:4BZCJGQ1U1-1 code:404 status:all_function not defined
DBG|2019-05-16 19:43:52|data_template_sample.c|event_post_cb(69): Reply:{"method":"events_reply","clientToken":"4BZCJGQ1U1-1","code":404,"status":"all_function not defined","data":{}}
DBG|2019-05-16 19:43:52|qcloud_iot_event.c|_release_reply_request(78): eventToken[4BZCJGQ1U1-1] released
```

#### 2. 执行 light_data_template_sample 示例程序

智能灯示例是基于数据数据模板的场景示例，基于此示例说明控制台和设备端控制交互及日志查询

```bash
./output/release/bin/light_data_template_sample
INF|2019-05-16 19:50:39|device.c|iot_device_info_set(65): SDK_Ver: 3.0.0, Product_ID: 4BZCJGQ1U1, Device_Name: dev1
DBG|2019-05-16 19:50:39|HAL_TLS_mbedtls.c|HAL_TLS_Connect(204): Connecting to /4BZCJGQ1U1.iotcloud.tencentdevices.com/8883...
DBG|2019-05-16 19:50:39|HAL_TLS_mbedtls.c|HAL_TLS_Connect(209): Setting up the SSL/TLS structure...
DBG|2019-05-16 19:50:39|HAL_TLS_mbedtls.c|HAL_TLS_Connect(251): Performing the SSL/TLS handshake...
INF|2019-05-16 19:50:39|HAL_TLS_mbedtls.c|HAL_TLS_Connect(269): connected with /4BZCJGQ1U1.iotcloud.tencentdevices.com/8883...
INF|2019-05-16 19:50:40|mqtt_client.c|IOT_MQTT_Construct(114): mqtt connect with id: GBh8Q success
DBG|2019-05-16 19:50:40|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$log/operation/result/4BZCJGQ1U1/dev1|packet_id=22804|pUserdata=(null)
DBG|2019-05-16 19:50:41|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$log/operation/result/4BZCJGQ1U1/dev1|packet_id=22805|pUserdata=(null)
DBG|2019-05-16 19:50:41|log_mqtt.c|_log_mqtt_event_handler(97): subscribe success, packet-id=22804
DBG|2019-05-16 19:50:41|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$log/operation/4BZCJGQ1U1/dev1|payload={"type": "get_log_level", "clientToken": "4BZCJGQ1U1-1"}
WRN|2019-05-16 19:50:41|mqtt_client_common.c|_handle_suback_packet(1014): There is a identical topic and related handle in list!
DBG|2019-05-16 19:50:41|shadow_client.c|_shadow_event_handler(63): shadow subscribe success, packet-id=22805
DBG|2019-05-16 19:50:41|log_mqtt.c|_log_level_sub_cb(60): Recv Msg Topic:$log/operation/result/4BZCJGQ1U1/dev1, payload:{"clientToken":"4BZCJGQ1U1-1","log_level":0,"result":0,"timestamp":1558007441,"type":"get_log_level"}

WRN|2019-05-16 19:50:41|log_mqtt.c|_log_level_sub_cb(68): Upload log level change to: 0
DBG|2019-05-16 19:50:41|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$template/operation/result/4BZCJGQ1U1/dev1|packet_id=22806|pUserdata=(null)
DBG|2019-05-16 19:50:41|shadow_client.c|_shadow_event_handler(63): shadow subscribe success, packet-id=22806
INF|2019-05-16 19:50:41|light_data_template_sample.c|event_handler(213): subscribe success, packet-id=22806
INF|2019-05-16 19:50:41|shadow_client.c|IOT_Shadow_Construct(173): Sync device data successfully
INF|2019-05-16 19:50:41|light_data_template_sample.c|main(496): Cloud Device Construct Success
DBG|2019-05-16 19:50:41|mqtt_client_subscribe.c|qcloud_iot_mqtt_subscribe(139): topicName=$thing/down/event/4BZCJGQ1U1/dev1|packet_id=22807|pUserdata=(null)
INF|2019-05-16 19:50:41|light_data_template_sample.c|_register_data_template_property(369): data template property=power_switch registered.
INF|2019-05-16 19:50:41|light_data_template_sample.c|_register_data_template_property(369): data template property=color registered.
INF|2019-05-16 19:50:41|light_data_template_sample.c|_register_data_template_property(369): data template property=brightness registered.
INF|2019-05-16 19:50:41|light_data_template_sample.c|_register_data_template_property(369): data template property=name registered.
INF|2019-05-16 19:50:41|light_data_template_sample.c|main(517): Register data template propertys Success
DBG|2019-05-16 19:50:41|shadow_client.c|IOT_Shadow_Get(384): GET Request Document: {"clientToken":"4BZCJGQ1U1-0"}
DBG|2019-05-16 19:50:41|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$template/operation/4BZCJGQ1U1/dev1|payload={"type":"get", "clientToken":"4BZCJGQ1U1-0"}
DBG|2019-05-16 19:50:41|shadow_client.c|_shadow_event_handler(63): shadow subscribe success, packet-id=22807
INF|2019-05-16 19:50:41|light_data_template_sample.c|event_handler(213): subscribe success, packet-id=22807
DBG|2019-05-16 19:50:41|shadow_client.c|_update_ack_cb(114): requestAck=0
DBG|2019-05-16 19:50:41|shadow_client.c|_update_ack_cb(117): Received Json Document={"clientToken":"4BZCJGQ1U1-0","payload":{"metadata":{"reported":{"brightness":{"timestamp":1558002454249},"color":{"timestamp":1558002454249},"name":{"timestamp":1558002454249},"power_switch":{"timestamp":1558002454249}}},"state":{"reported":{"brightness":0,"color":0,"name":"dev1","power_switch":0}},"timestamp":1558002454249,"version":34},"result":0,"timestamp":1558007441,"type":"get"}

DBG|2019-05-16 19:50:42|light_data_template_sample.c|main(602): cycle report:{"version":34, "state":{"reported":{"power_switch":0,"color":0,"brightness":0.000000,"name":"dev1"}}, "clientToken":"4BZCJGQ1U1-1"}
DBG|2019-05-16 19:50:42|shadow_client.c|IOT_Shadow_Update(318): UPDATE Request Document: {"version":34, "state":{"reported":{"power_switch":0,"color":0,"brightness":0.000000,"name":"dev1"}}, "clientToken":"4BZCJGQ1U1-1"}
DBG|2019-05-16 19:50:42|mqtt_client_publish.c|qcloud_iot_mqtt_publish(337): publish packetID=0|topicName=$template/operation/4BZCJGQ1U1/dev1|payload={"type":"update", "version":34, "state":{"reported":{"power_switch":0,"color":0,"brightness":0.000000,"name":"dev1"}}, "clientToken":"4BZCJGQ1U1-1"}
INF|2019-05-16 19:50:42|light_data_template_sample.c|main(607): shadow update(reported) success
INF|2019-05-16 19:50:43|light_data_template_sample.c|OnShadowUpdateCallback(351): recv shadow update response, response ack: 0
```

#### 3. 设备调试

 1. 进入设备调试：
 ![]( https://main.qcloudimg.com/raw/1f2ac1d6cac186394ac1a1da6c22749c.jpg)

 2. 修改目标数据下发设备
  ![]( https://main.qcloudimg.com/raw/911a09872f03a91d1d530537e51147f1.jpg)

3.设备响应及日志输出
  ![]( https://main.qcloudimg.com/raw/c45d35d9abe116408efc2656b55127b7.jpg)

4. 控制台查看设备当前状态
 ![]( https://main.qcloudimg.com/raw/236b8bf3c88b1c532714b730d0993a79.jpg)

5. 控制台查看设备通信日志
 ![]( https://main.qcloudimg.com/raw/10e911975030f2840b9af03a079aec1d.jpg)

6. 控制台查看设备事件上报
 ![]( https://main.qcloudimg.com/raw/d3878541b502619158ec206fc2ae2391.jpg)

## 四. 可变接入参数配置

可变接入参数配置：SDK 的使用可以根据具体场景需求，配置相应的参数，满足实际业务的运行。可变接入参数包括：

1. MQTT 心跳消息发送周期, 单位: ms
2. MQTT 阻塞调用(包括连接, 订阅, 发布等)的超时时间, 单位:ms。 建议 5000 ms
3. TLS 连接握手超时时间, 单位: ms
4. MQTT 协议发送消息和接受消息的 buffer 大小默认是 512 字节，最大支持 256 KB
5. 重连最大等待时间
修改 qcloud_iot_export.h 文件如下宏定义可以改变对应接入参数的配置。

```c
/* MQTT心跳消息发送周期, 单位:ms */
#define QCLOUD_IOT_MQTT_KEEP_ALIVE_INTERNAL                         (240 * 1000)

/* MQTT 阻塞调用(包括连接, 订阅, 发布等)的超时时间, 单位:ms 建议5000ms */
#define QCLOUD_IOT_MQTT_COMMAND_TIMEOUT                             (5000)

/* TLS连接握手超时时间, 单位:ms */
#define QCLOUD_IOT_TLS_HANDSHAKE_TIMEOUT                            (5000)

/* MQTT消息发送buffer大小, 支持最大256*1024 */
#define QCLOUD_IOT_MQTT_TX_BUF_LEN                                  (512)

/* MQTT消息接收buffer大小, 支持最大256*1024 */
#define QCLOUD_IOT_MQTT_RX_BUF_LEN                                  (512)

/* 重连最大等待时间 */
#define MAX_RECONNECT_WAIT_INTERVAL                                 (60000)
```

## 关于 SDK 的更多使用方式及接口了解, 请访问[官方 WiKi](https://github.com/tencentyun/qcloud-iot-sdk-embedded-c/wiki)
