# Lab 2: Hello World และ Serial Communication

## วัตถุประสงค์
ทำความเข้าใจ Serial Output, Monitor และ Logging ใน ESP-IDF

## เวลาที่ใช้
30 นาที

## อุปกรณ์ที่ต้องใช้
- ESP32 Development Board
- USB Cable

## ขั้นตอนการทดลอง

### Step 1: Flash และ Monitor (10 นาที)

```bash
# 1. Flash โปรแกรมลง ESP32
idf.py flash

# 2. เปิด Serial Monitor
idf.py monitor

# 3. Flash และ Monitor ในคราวเดียว
idf.py flash monitor
```

**ผลลัพธ์ที่ควรเห็น:**
```
Hello, ESP32 World!
ESP32 is running...
ESP32 is running...
ESP32 is running...
```

**การออกจาก Monitor:**
- Linux/macOS: `Ctrl+]`
- Windows: `Ctrl+T` แล้ว `Ctrl+X`

### Step 2: ใช้งาน ESP Logging (15 นาที)

สร้างไฟล์ `main/logging_demo.c`:

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"
#include "nvs_flash.h"

// Define tag for logging
static const char *TAG = "LOGGING_DEMO";

void demonstrate_logging_levels(void)
{
    ESP_LOGE(TAG, "This is an ERROR message - highest priority");
    ESP_LOGW(TAG, "This is a WARNING message");
    ESP_LOGI(TAG, "This is an INFO message - default level");
    ESP_LOGD(TAG, "This is a DEBUG message - needs debug level");
    ESP_LOGV(TAG, "This is a VERBOSE message - needs verbose level");
}

void demonstrate_formatted_logging(void)
{
    int temperature = 25;
    float voltage = 3.3;
    const char* status = "OK";
    
    ESP_LOGI(TAG, "Sensor readings:");
    ESP_LOGI(TAG, "  Temperature: %d°C", temperature);
    ESP_LOGI(TAG, "  Voltage: %.2fV", voltage);
    ESP_LOGI(TAG, "  Status: %s", status);
    
    // Hexadecimal dump
    uint8_t data[] = {0xDE, 0xAD, 0xBE, 0xEF};
    ESP_LOGI(TAG, "Data dump:");
    ESP_LOG_BUFFER_HEX(TAG, data, sizeof(data));
}

void demonstrate_conditional_logging(void)
{
    int error_code = 0;
    
    // Conditional logging
    if (error_code != 0) {
        ESP_LOGE(TAG, "Error occurred: code %d", error_code);
    } else {
        ESP_LOGI(TAG, "System is running normally");
    }
    
    // Using ESP_ERROR_CHECK
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);
    ESP_LOGI(TAG, "NVS initialized successfully");
}

void app_main(void)
{
    // System information
    ESP_LOGI(TAG, "=== ESP32 Hello World Demo ===");
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "Chip Model: %s", CONFIG_IDF_TARGET);
    ESP_LOGI(TAG, "Free Heap: %d bytes", esp_get_free_heap_size());
    ESP_LOGI(TAG, "Min Free Heap: %d bytes", esp_get_minimum_free_heap_size());
    
    // CPU and Flash info
    esp_chip_info_t chip_info;
    esp_chip_info(&chip_info);
    ESP_LOGI(TAG, "Chip cores: %d", chip_info.cores);
    ESP_LOGI(TAG, "Flash size: %dMB %s", 
             spi_flash_get_chip_size() / (1024 * 1024),
             (chip_info.features & CHIP_FEATURE_EMB_FLASH) ? "embedded" : "external");
    
    // Demonstrate different logging levels
    ESP_LOGI(TAG, "\n--- Logging Levels Demo ---");
    demonstrate_logging_levels();
    
    // Demonstrate formatted logging
    ESP_LOGI(TAG, "\n--- Formatted Logging Demo ---");
    demonstrate_formatted_logging();
    
    // Demonstrate conditional logging
    ESP_LOGI(TAG, "\n--- Conditional Logging Demo ---");
    demonstrate_conditional_logging();
    
    // Main loop with counter
    int counter = 0;
    while (1) {
        ESP_LOGI(TAG, "Main loop iteration: %d", counter++);
        
        // Log memory status every 10 iterations
        if (counter % 10 == 0) {
            ESP_LOGI(TAG, "Memory status - Free: %d bytes", esp_get_free_heap_size());
        }
        
        // Simulate different log levels based on counter
        if (counter % 20 == 0) {
            ESP_LOGW(TAG, "Warning: Counter reached %d", counter);
        }
        
        if (counter > 50) {
            ESP_LOGE(TAG, "Error simulation: Counter exceeded 50!");
            counter = 0; // Reset counter
        }
        
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds
    }
}
```

อัปเดต `main/CMakeLists.txt`:
```cmake
idf_component_register(SRCS "logging_demo.c"
                       INCLUDE_DIRS ".")
```

### Step 3: การปรับ Log Level (5 นาที)

```bash
# 1. เปิด configuration menu
idf.py menuconfig

# นำทางไปยัง:
# Component config → Log output → Default log verbosity
# เลือก: Verbose, Debug, Info, Warning, Error, หรือ None

# 2. หรือตั้งค่าผ่าน environment variable
export ESP_LOG_LEVEL=DEBUG
idf.py build flash monitor
```

**การตั้งค่า Log Level ในโค้ด:**
```c
// Set log level for specific tag
esp_log_level_set("LOGGING_DEMO", ESP_LOG_DEBUG);

// Set global log level
esp_log_level_set("*", ESP_LOG_INFO);
```

## แบบฝึกหัด

### Exercise 1: สร้าง Custom Logger

```c
#include "esp_log.h"

#define LOG_COLOR_BLACK   "30"
#define LOG_COLOR_RED     "31"
#define LOG_COLOR_GREEN   "32"
#define LOG_COLOR_BROWN   "33"
#define LOG_COLOR_BLUE    "34"
#define LOG_COLOR_PURPLE  "35"
#define LOG_COLOR_CYAN    "36"

#define LOG_BOLD(COLOR)  "\033[1;" COLOR "m"
#define LOG_RESET_COLOR  "\033[0m"

void custom_log(const char* tag, const char* format, ...)
{
    va_list args;
    va_start(args, format);
    
    printf(LOG_BOLD(LOG_COLOR_CYAN) "[CUSTOM] %s: " LOG_RESET_COLOR, tag);
    vprintf(format, args);
    printf("\n");
    
    va_end(args);
}

// Usage
custom_log("SENSOR", "Temperature: %d°C", 25);
```

### Exercise 2: Performance Monitoring

```c
#include "esp_timer.h"

void performance_demo(void)
{
    ESP_LOGI(TAG, "=== Performance Monitoring ===");
    
    // Measure execution time
    uint64_t start_time = esp_timer_get_time();
    
    // Simulate some work
    for (int i = 0; i < 1000000; i++) {
        volatile int dummy = i * 2;
    }
    
    uint64_t end_time = esp_timer_get_time();
    uint64_t execution_time = end_time - start_time;
    
    ESP_LOGI(TAG, "Execution time: %lld microseconds", execution_time);
    ESP_LOGI(TAG, "Execution time: %.2f milliseconds", execution_time / 1000.0);
}
```

### Exercise 3: Error Handling Demo

```c
void error_handling_demo(void)
{
    ESP_LOGI(TAG, "=== Error Handling Demo ===");
    
    // Simulate various error conditions
    esp_err_t result;
    
    // Success case
    result = ESP_OK;
    if (result == ESP_OK) {
        ESP_LOGI(TAG, "Operation completed successfully");
    }
    
    // Error cases
    result = ESP_ERR_NO_MEM;
    if (result != ESP_OK) {
        ESP_LOGE(TAG, "Error: %s", esp_err_to_name(result));
    }
    
    result = ESP_ERR_INVALID_ARG;
    ESP_ERROR_CHECK_WITHOUT_ABORT(result);
    if (result != ESP_OK) {
        ESP_LOGW(TAG, "Non-fatal error: %s", esp_err_to_name(result));
    }
}
```

## การใช้งาน Serial Monitor ขั้นสูง

### คำสั่งใน Monitor Mode
```bash
# Restart ESP32
Ctrl+T, Ctrl+R

# Exit monitor
Ctrl+T, Ctrl+X

# Help
Ctrl+T, Ctrl+H

# Toggle timestamps
Ctrl+T, Ctrl+T
```

### การบันทึก Log
```bash
# บันทึกลงไฟล์
idf.py monitor > output.log 2>&1

# หรือใช้ tee เพื่อดูและบันทึกพร้อมกัน
idf.py monitor | tee output.log
```

## Checklist การทำงาน

- [ / ] Flash และ Monitor สำเร็จ
- [ / ] เห็น Log messages ต่างๆ
- [ / ] ทดสอบ Log levels ต่างๆ
- [ / ] ใช้ formatted logging ได้
- [ / ] ปรับ Log level ผ่าน menuconfig
- [ / ] ทำแบบฝึกหัดครบ

## คำถามทบทวน

1. ความแตกต่างระหว่าง printf() และ ESP_LOGI() คืออะไร? 📝
printf() คือฟังก์ชันมาตรฐาน C ที่ส่งข้อความออกไปโดยตรง ไม่มีการควบคุม Log Level หรือ Tag

ESP_LOGI() เป็นฟังก์ชันของ ESP-IDF ที่มี Log Level และ Tag สามารถควบคุม (เปิด/ปิด/กรอง) ข้อความที่แสดงผลได้ ทำให้มีประสิทธิภาพและบริบทที่ดีกว่า

2. Log level ไหนที่จะแสดงใน default configuration?
โดยค่าเริ่มต้น (Default) ของ ESP-IDF Log Level ที่จะแสดงคือ Info (I) และ Level ที่สูงกว่า (ร้ายแรงกว่า) ทั้งหมด ได้แก่ Error (E) และ Warning (W)

3. การใช้ ESP_ERROR_CHECK() มีประโยชน์อย่างไร?
มีประโยชน์ในการ ตรวจสอบโค้ด Error ที่ส่งค่ากลับมาจากฟังก์ชันต่าง ๆ หากพบ Error (ค่าที่ไม่ใช่ ESP_OK) ระบบจะ หยุดการทำงาน (abort()) และแสดงข้อมูลข้อผิดพลาด (ไฟล์, บรรทัด) โดยอัตโนมัติ ทำให้โค้ดกระชับและจัดการ Error ได้รวดเร็ว

4. คำสั่งใดในการออกจาก Monitor mode?
คำสั่งในการออกจาก ESP-IDF Monitor mode คือการกดคีย์ผสม Ctrl + ]

5. การตั้งค่า Log level สำหรับ tag เฉพาะทำอย่างไร?
ทำได้ผ่านเมนู menuconfig โดยเข้าไปที่ Component config -> Log output -> Log verbosity per tag จากนั้นจึงระบุ Tag และ Maximum Log Level ที่ต้องการครับ
## บทสรุป

ในแลปนี้คุณได้เรียนรู้:
- การใช้ Serial Monitor
- ระบบ Logging ของ ESP-IDF
- การตั้งค่า Log levels
- การ Format และแสดงข้อมูล
- การจัดการ Errors

**ขั้นตอนต่่อไป:** Lab 3 - สร้าง Task แรกด้วย FreeRTOS