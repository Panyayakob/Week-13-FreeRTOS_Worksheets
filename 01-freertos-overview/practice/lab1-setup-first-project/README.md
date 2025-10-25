# Lab 1: ESP-IDF Setup และโปรเจกต์แรก

## วัตถุประสงค์
ติดตั้งและตรวจสอบการทำงานของ ESP-IDF พร้อมสร้างโปรเจกต์แรก

## เวลาที่ใช้
45 นาที

## ข้อกำหนดเบื้องต้น
- ESP32 Development Board
- USB Cable
- คอมพิวเตอร์ที่ติดตั้ง ESP-IDF แล้ว

## ขั้นตอนการทดลอง

### Step 1: ตรวจสอบการติดตั้ง ESP-IDF (10 นาที)

```bash
# 1. ตรวจสอบ ESP-IDF environment
echo $IDF_PATH

# 2. ตรวจสอบเวอร์ชัน
idf.py --version

# 3. ตรวจสอบ Python และ pip
python --version
pip --version

# 4. ตรวจสอบ toolchain
xtensa-esp32-elf-gcc --version
```

**ผลลัพธ์ที่คาดหวัง:**
```
ESP-IDF v5.x
Python 3.x.x
xtensa-esp32-elf-gcc version x.x.x
```

### Step 2: สร้างโปรเจกต์แรก (15 นาที)

```bash
# 1. สร้างโปรเจกต์ใหม่
idf.py create-project hello_esp32
cd hello_esp32

# 2. ดูโครงสร้างโปรเจกต์
ls -la
```

**โครงสร้างโปรเจกต์:**
```
hello_esp32/
├── CMakeLists.txt          # Build configuration
├── main/
│   ├── CMakeLists.txt      # Main component
│   └── hello_esp32.c       # Main source file
├── README.md               # Project documentation
└── sdkconfig               # Project configuration (after build)
```

### Step 3: ศึกษาไฟล์หลัก (10 นาที)

**ไฟล์ `main/hello_esp32.c`:**
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

void app_main(void)
{
    printf("Hello, ESP32 World!\n");
    
    // Keep the program running
    while(1) {
        printf("ESP32 is running...\n");
        vTaskDelay(pdMS_TO_TICKS(1000)); // 1 second delay
    }
}
```

**ไฟล์ `CMakeLists.txt` (root):**
```cmake
cmake_minimum_required(VERSION 3.16)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(hello_esp32)
```

**ไฟล์ `main/CMakeLists.txt`:**
```cmake
idf_component_register(SRCS "hello_esp32.c"
                       INCLUDE_DIRS ".")
```

### Step 4: Build โปรเจกต์ (10 นาที)

```bash
# 1. กำหนด target chip
idf.py set-target esp32

# 2. Build โปรเจกต์
idf.py build
```

**ข้อความที่ควรเห็น:**
```
Project build complete. To flash, run:
  idf.py flash
or
  idf.py -p (PORT) flash
```

**ไฟล์ที่ถูกสร้าง:**
```
build/
├── bootloader/
├── partition_table/
├── hello_esp32.bin         # Main application binary
├── hello_esp32.elf         # ELF file with debug info
└── flash_args              # Flash arguments
```

## การแก้ไขปัญหาที่อาจเกิดขึ้น

### ปัญหา 1: ESP-IDF not found
```bash
# แก้ไข: Set up environment
source $HOME/esp/esp-idf/export.sh
# หรือ (Windows)
%USERPROFILE%\esp\esp-idf\export.bat
```

### ปัญหา 2: Permission denied
```bash
# Linux/macOS
sudo chmod 666 /dev/ttyUSB0
# หรือเพิ่ม user ใน dialout group
sudo usermod -a -G dialout $USER
```

### ปัญหา 3: Target not set
```bash
idf.py set-target esp32
```

## แบบฝึกหัด

### Exercise 1: แก้ไขข้อความ
แก้ไขไฟล์ `main/hello_esp32.c` ให้แสดงข้อความที่แตกต่าง:

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"

void app_main(void)
{
    printf("=== My First ESP32 Project ===\n");
    printf("Chip model: %s\n", esp_get_idf_version());
    printf("Free heap: %d bytes\n", esp_get_free_heap_size());
    
    int counter = 0;
    while(1) {
        printf("Running for %d seconds\n", counter++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

### Exercise 2: เพิ่ม Build Information
สร้างไฟล์ `main/build_info.h`:

```c
#ifndef BUILD_INFO_H
#define BUILD_INFO_H

#define PROJECT_NAME "Hello ESP32"
#define PROJECT_VERSION "1.0.0"
#define BUILD_DATE __DATE__
#define BUILD_TIME __TIME__

#endif
```

แก้ไข `main/hello_esp32.c`:

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "build_info.h"

void app_main(void)
{
    printf("=== %s v%s ===\n", PROJECT_NAME, PROJECT_VERSION);
    printf("Built on: %s %s\n", BUILD_DATE, BUILD_TIME);
    printf("ESP-IDF Version: %s\n", esp_get_idf_version());
    printf("Chip: %s\n", CONFIG_IDF_TARGET);
    printf("Free heap: %d bytes\n", esp_get_free_heap_size());
    
    while(1) {
        printf("System is running...\n");
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}
```

### Exercise 3: การ Build แบบ Verbose
```bash
# Build with verbose output
idf.py -v build

# Clean และ build ใหม่
idf.py clean
idf.py build
```

## Checklist การทำงาน

- [ / ] ตรวจสอบ ESP-IDF environment สำเร็จ
- [ / ] สร้างโปรเจกต์ใหม่ได้
- [ / ] เข้าใจโครงสร้างโปรเจกต์
- [ / ] Build โปรเจกต์สำเร็จ
- [ / ] แก้ไขโค้ดและ build ใหม่ได้
- [ / ] เข้าใจไฟล์ CMakeLists.txt
- [ / ] ทำแบบฝึกหัดครบ

## คำถามทบทวหน

1. ไฟล์ใดบ้างที่จำเป็นสำหรับโปรเจกต์ ESP-IDF ขั้นต่ำ?
ต้องมี CMakeLists.txt ในรูทโปรเจกต์ และมีโฟลเดอร์ main/ ซึ่งบรรจุไฟล์ main/CMakeLists.txt และไฟล์ซอร์สโค้ด (เช่น main/*.c) ที่มีฟังก์ชัน app_main()

2. ความแตกต่างระหว่าง hello_esp32.bin และ hello_esp32.elf คืออะไร?
.bin คือ ไฟล์เฟิร์มแวร์ (Binary) ที่พร้อมนำไป แฟลช (Flash) ลง ESP32 เพื่อรันจริง

.elf คือ ไฟล์ดีบัก (Symbolic) ที่ใช้สำหรับ การดีบัก ด้วยข้อมูลสัญลักษณ์ (Symbol Table) ครบถ้วน

3. คำสั่ง idf.py set-target ทำอะไร?
ใช้เพื่อ กำหนดเป้าหมายชิป (เช่น ESP32, ESP32-S3) ให้แก่ระบบบิลด์ เพื่อให้เลือกใช้ Toolchain และ Library ที่ถูกต้องสำหรับฮาร์ดแวร์นั้น ๆ

4. โฟลเดอร์ build/ มีไฟล์อะไรบ้าง?
เก็บ ไฟล์ที่ถูกสร้างขึ้นระหว่างการบิลด์ ซึ่งรวมถึงไฟล์เอาต์พุตสุดท้าย (.bin, .elf), ไฟล์ Object (.o), และไฟล์ Configuration ต่าง ๆ ที่ใช้ในการคอมไพล์

5. การใช้ vTaskDelay() แทน delay() มีความสำคัญอย่างไร?
vTaskDelay() จะทำให้ Task คืนการควบคุม CPU ให้ Scheduler ทันที เพื่อให้ Task อื่นสามารถรันได้ (Multitasking) ส่วน delay() แบบปกติจะ ผูกขาด CPU ทำให้ Task อื่นทำงานไม่ได้ ซึ่งขัดต่อหลักการของ RTOS

## บทสรุป

ในแลปนี้คุณได้เรียนรู้:
- การตรวจสอบ ESP-IDF environment
- การสร้างโปรเจกต์ ESP-IDF
- โครงสร้างโปรเจกต์และไฟล์สำคัญ
- กระบวนการ build
- การแก้ไขปัญหาเบื้องต้น

**ขั้นตอนต่อไป:** Lab 2 - Hello World และ Serial Communication