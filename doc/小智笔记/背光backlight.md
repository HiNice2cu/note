# 背光backlight

## backlight.h

#### 定义函数与变量

```c++
public:
	void brightness_timer_startup();
    void brightness_timer_close();
    void brightness_trigger();
protected:
	esp_timer_handle_t transition_timer_2_ = nullptr;
	int last_timer_value_ = 0;
```



## backlight.cc

#### 初始化定时器

```c++
Backlight::Backlight() {
	const esp_timer_create_args_t bl_timer_args = {
        .callback = [](void* arg) {
            auto self = static_cast<Backlight*>(arg);
           
            self->SetBrightness(0,0);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "backlight_timer_2",
        .skip_unhandled_events = true,
    };
    ESP_ERROR_CHECK(esp_timer_create(&bl_timer_args, &transition_timer_2_));
}
```

#### 开启背光定时器,关闭背光定时器,开启背光(设置亮度)

```c++
void Backlight::brightness_timer_startup(){
        esp_timer_start_periodic(transition_timer_2_, 5000 * 1000);
}
void Backlight::brightness_timer_close(){
        esp_timer_stop(transition_timer_2_);
}

void Backlight::brightness_trigger(){
        SetBrightness(last_timer_value_);
}
```

#### 记录上次亮度(使得再次亮屏时亮度不会改变)

```c++
void Backlight::RestoreBrightness(){
	int saved_brightness = settings.GetInt("brightness", 75);
    last_timer_value_ = saved_brightness;
}

void Backlight::SetBrightness(uint8_t brightness, bool permanent){
	...
	//permanent == true 表示“这是用户希望长期保留的亮度设置”；permanent == false 表示“临时/过渡性的改变”（比如自动熄屏、动画过渡）。
	if (permanent) {
        ...
        last_timer_value_ = brightness;
    }
    ...
}

```

## application.cc

状态在空闲状态下需要开启背光定时器(到时间自动熄屏),其他状态要关闭

每一个状态都需要开启背光

```c++
void Application::SetDeviceState(DeviceState state){
	auto backlight = board.GetBacklight();
	...
}

```

# 拓展:触摸亮屏

## lichuang_dev_board.cc

#### 初始化触摸定时器+添加触摸回调函数:

```c++
esp_timer_handle_t touch_timer_2_ = nullptr; 

void InitializeTouch(){
	esp_lcd_touch_config_t tp_cfg ={
	...
	.process_coordinates = touch_process_coordinates,
	}
	
	const esp_timer_create_args_t touch_timer_args = {
        .callback = [](void* arg) {
            ESP_LOGI(TAG, "Touch timer callback");
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "touch_timer",
        .skip_unhandled_events = true,
    };
    ESP_ERROR_CHECK(esp_timer_create(&touch_timer_args, &touch_timer_2_));
}
```

#### 触摸回调函数的实现:

该函数在首次触摸时切换聊天状态，随后触摸在计时器有效期内只会重置计时器而不再次切换，从而实现“进入交互→超时退出”的交互窗口。

```c++
void touch_process_coordinates(esp_lcd_touch_handle_t tp, uint16_t *x, uint16_t *y, uint16_t *strength, uint8_t *point_num, uint8_t max_point_num){
    if(esp_timer_is_active(touch_timer_2_)){
        esp_timer_stop(touch_timer_2_);
        esp_timer_start_once(touch_timer_2_, 60000);
    }
    else{
        auto& app = Application::GetInstance();
        app.ToggleChatState();
        esp_timer_start_once(touch_timer_2_, 60000);
        ESP_LOGI(TAG, "Touch detected, toggling chat state");
    }
}
```

```c++
void Application::ToggleChatState() {
    auto backlight = Board::GetInstance().GetBacklight();
    if(backlight==nullptr){
        return;
    }
    if(backlight->brightness() == 0){
        backlight->brightness_trigger();
        backlight->brightness_timer_startup();
        return;
    }
    ...
}
```

