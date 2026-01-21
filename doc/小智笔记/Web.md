# Web

## mqtt

例:温度传感器作为客户端连接到 MQTT Broker，并通过发布操作将温度数据发布到一个特定主题（例如 `Temperature`）。MQTT Broker 接收到该消息后会负责将其转发给订阅了相应主题（`Temperature`）的订阅者客户端。

![image-20251116200014788](https://q-1336166460.cos.ap-beijing.myqcloud.com/p/202511162000873.png)

mqtt_self.cc

mqtt_self.h

application.cc

application.h

#### mqtt_self.h

##### 创建类

```c++
#ifndef MQTT_SELF_H
#define MQTT_SELF_H

#include <mqtt.h>
#include <memory>


class mqtt_self
{
private:
    std::unique_ptr<Mqtt> mqtt_;

    /* data */
public:
    mqtt_self(/* args */);
    ~mqtt_self();
};


#endif
```

#### mqtt_self.cc

##### 实现构造函数:

![image-20251116202658417](C:\Users\12804\AppData\Roaming\Typora\typora-user-images\image-20251116202658417.png)

mqtt_->Subscribe("qin/+", 2)



![image-20251116202635577](C:\Users\12804\AppData\Roaming\Typora\typora-user-images\image-20251116202635577.png)

mqtt_->Connect("115.190.41.147", 1883, "client_id", "", "")

```c++
#include "mqtt_self.h"
#include "board.h"
#include <esp_log.h>

#define TAG "MQTT_SELF"

mqtt_self::mqtt_self(/* args */)
{
    auto network=Board::GetInstance().GetNetwork();
    mqtt_=network->CreateMqtt(1);
    mqtt_->SetKeepAlive(90);

    mqtt_->OnDisconnected([this]() {
        ESP_LOGW(TAG, "MQTT disconnected, scheduling reconnect");
    });

    mqtt_->OnConnected([this]() {
        ESP_LOGI(TAG, "MQTT connected");
        if (!mqtt_->Subscribe("qin/+", 2)) {
            ESP_LOGE(TAG, "Failed to subscribe to topic");
        }
    });

    mqtt_->OnMessage([this](const std::string& topic, const std::string& payload) {
        ESP_LOGI(TAG, "Received message on topic %s: %s", topic.c_str(), payload.c_str());
    });

    if (!mqtt_->Connect("115.190.41.147", 1883, "client_id", "", "")) {
        ESP_LOGE(TAG, "Failed to connect to endpoint");
        return;
    }

    ESP_LOGI(TAG, "Connected to endpoint");
}

mqtt_self::~mqtt_self()
{
}

```

#### application.h

```c++
private:
    mqtt_self* mqtt_client_ = nullptr;
```

#### application.cc

```c++
void Application::Start(){
	...
	mqtt_client_ = new mqtt_self();
}
```

## http

#### http_self.h

##### 创建类

```c++
#ifndef HTTP_SELF_H
#define HTTP_SELF_H

class http_self
{
private:
    /* data */
public:
    http_self(/* args */);
    ~http_self();
};
#endif
```

#### http_self.cc

- 网络模块初始化
- HTTP GET 请求
- 状态码检查
- 响应长度获取
- 循环读取数据
- JSON 解析
- 数组元素提取
- 字段验证
- 资源清理
- 析构函数

```c++
#include "http_self.h"
#include "board.h"
#include <esp_log.h>
#include "http.h"
#include "cJSON.h"
#define TAG "HTTP_SELF"

http_self::http_self(/* args */)
{
    auto network=Board::GetInstance().GetNetwork();
    auto http=network->CreateHttp(2);

    if (!http->Open("GET", "https://www.xvsenfeng.top/shops/shoplists/?boardID=24EC4A0AA884")) {
        ESP_LOGE(TAG, "Failed to open HTTP connection");
        return ;
    }

    if (http->GetStatusCode() != 200) {
        ESP_LOGE(TAG, "Failed to get firmware, status code: %d", http->GetStatusCode());
        return ;
    }

    size_t content_length = http->GetBodyLength();
    if (content_length == 0) {
        ESP_LOGE(TAG, "Failed to get content length");
        return ;
    }

    char buffer[600];
    size_t total_read = 0;
    while (true) {
        int ret = http->Read(buffer, sizeof(buffer));
        if (ret < 0) {
            ESP_LOGE(TAG, "Failed to read HTTP data: %s", esp_err_to_name(ret));
            return;
        }
        ESP_LOGI(TAG, "Read %s", buffer);
        total_read += ret;
        buffer[total_read] = 0;
        if(total_read>=content_length){

            break;
        }
    }


    cJSON* root = cJSON_Parse(buffer);
    if (root == nullptr) {
        ESP_LOGE(TAG, "Failed to parse json message %s",buffer);
        return;
    }

    cJSON *item =cJSON_GetArrayItem(root, 1);

    cJSON* type = cJSON_GetObjectItem(item, "id");
    if (!cJSON_IsNumber(type)) {
        ESP_LOGE(TAG, "Message type is invalid");
        cJSON_Delete(root);
        return;
    }

    ESP_LOGI(TAG, "id: %d", type->valueint);
    cJSON_Delete(root);
    http->Close();
}
http_self::~http_self()
{
}
```

#### application.h和application.cc同mqtt

