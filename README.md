ESP-NOW using the PlatformIO ESP-IDF framework
==============================================

This page first describes installing Visual Studio Code and the PlatformIO plugin. Then, getting a blink example going using the Arduino framework and then the ESP-IDF framework. And finally, getting an ESP-NOW example running.

PlatformIO setup
----------------

Guided by Random Nerd Tutorial's [Getting Started with VS Code and PlatformIO IDE for ESP32 and ESP8266 (Windows, Mac OS X, Linux Ubuntu)](https://randomnerdtutorials.com/vs-code-platformio-ide-esp32-esp8266-arduino/), I did the following...

Downloaded the latest `.deb` from the VS Code [downloads page](https://code.visualstudio.com/Download), then...

```
$ cd ~/Downloads
$ sudo apt install ./code_1.89.1-1715060508_amd64.deb
```

VS Code was then available via the [Super key](https://help.ubuntu.com/stable/ubuntu-help/keyboard-key-super.html).

The `python3` and `python3-distutils` packages were already installed and the `/usr/bin/pytho3` version was 3.10.12 so, no additional Python related steps were required.

I started VS Code and:

* Clicked the _Extensions_ icon (bottom icon in group of four icons at the top of the left-hand bar).
* Searched for "plaformio IDE" and installed it.
* Exited and restarted VS Code.
* On restart, I clicked the PlatformIO "bug/alien head" icon (now, just below _Extensions_) and then its home icon (a little house that appears at the left of the bottom bar).

I also installed the Vim extension as I'm a Vim person.

The Random Nerd's tutorial then starts discussing the PlatformIO tasks (that appear to the right of the home icon), but these only appear if you create a new project or open an existing one so, I created a new one as descibed in the tutorial's next section...

Absolutely nothing happened when I pressed the _Create New Project_ button in the _Project Tasks_ section. But pressing _New Project_ in the main window worked as expected.

I used:

* Name: blink-c3-led
* Board: Seeed Studio XIAO ESP32C3
* Framework: Arduino
* Location: &lt;parent directory&gt;

Rather than let it use the default parent directory (which is `~/Documents/PlatformIO/Projects`), I chose a specific directory where I keep all my coding projects. You just specify the parent directory (PlatformIO then creates a subdirectory with the given project name).

Note: I'm sure _Arduino_ didn't appear as a dropdown option initially but after a bit of changing boards etc. it eventually appeared.

It took a minute or two before the usual dialog appeared about trusting code in the `blink-c3-led` directory that I'd just created.

I added the following line to `platformio.ini`:

```
monitor_speed = 115200
```

I actually used a SuperMini board (like this [one](https://www.aliexpress.com/item/1005005877531694.html) on AliExpress) that, unlike the Xiao C3, has a builtin LED connected to pin 8.

So, I could run this sketch:

```
#include <Arduino.h>

int led = 8;

void setup() {
  Serial.begin(115200);
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);
  Serial.println("LED value is now HIGH");
  delay(1000);
  digitalWrite(led, LOW);
  Serial.println("LED value is now LOW");
  delay(1000);
}
```

I replaced the contents of the `main.c` file with the above, clicked compile (the tick icon to the right of the home icon in the bottom bar) and then upload (next icon over).

As seems usual, this failed with:

```
OSError: [Errno 19] No such device
*** [upload] Error 1
```

The solution is to put the board into upload mode - hold down the BOOT button on the board, then press and release the RESET button and finally release the BOOT button.

Once this is done, click upload again - this time it should work.

Note: upload also does a compile so, you don't have to actively do compile as a separate step.

And then press RESET to get the board back into normal mode and it will start running the uploaded program.

Now, press the serial-monitor icon (it looks like a plug) and watch it print out:

```
LED value is now HIGH
LED value is now LOW
LED value is now HIGH
LED value is now LOW
...
```

Libraries
---------

To add a library, go to the _PIO Home_ tab, select the libraries icon, search for e.g. "neopixel", click "Adafruit NeoPixel" and then click the _Add to Project_ button. The library is then added to the `platformio.ini` file.

Creating an ESP-IDF project
---------------------------

Creating an ESP-IDF project is much the same as an Arduino one, just select _Espidf_ as the _Framework_.

As above, I added `monitor_speed = 115200` to the `platformio.ini`.

And I replaced the contents of the auto-generated `main.c` file with:

```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"
#include "sdkconfig.h"

static const char *TAG = "blink";

#define BLINK_GPIO 8
#define BLINK_PERIOD_MS 1000

static uint8_t s_led_state = 0;

void app_main(void)
{
    ESP_LOGI(TAG, "Example configured to blink GPIO LED!");
    gpio_reset_pin(BLINK_GPIO);
    gpio_set_direction(BLINK_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        ESP_LOGI(TAG, "Turning the LED %s!", s_led_state == true ? "ON" : "OFF");
        gpio_set_level(BLINK_GPIO, s_led_state);
        s_led_state = !s_led_state;
        vTaskDelay(BLINK_PERIOD_MS / portTICK_PERIOD_MS);
    }
}
```

The `ESP_LOGI` lines output log message with color escape codes (see [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)) - to get the PlatformIO serial monitor to properly interpret these, add the following to `platformio.ini`:

```
monitor_raw = yes
```

Initially, on trying to compile, it failed, complaining that the project was not a git repo:

```
fatal: not a git repository (or any of the parent directories): .git
```

I'm not sure why this is necessary but it was easy enough to resolve:

```
$ cd .../esp-idf-blink-c3-led
$ git init
$ git commit -m 'Initial import.'
```

When using the ESP-IDF framework, it seemed to handle uploading better, without always having to put the board into upload mode first.

ESP-IDF C++
-----------

I renamed `main.c` to `main.cpp` and updated the contents to:

```
#include "freertos/FreeRTOS.h"
#include "driver/gpio.h"
#include "esp_log.h"
#include <cstdint>

static const char *TAG = "blink";

const gpio_num_t BLINK_GPIO = GPIO_NUM_8;
const TickType_t BLINK_PERIOD_MS = 1000;

static std::uint8_t s_led_state = 0;

extern "C" void app_main(void)
{
    ESP_LOGI(TAG, "C++2 Example configured to blink GPIO LED!");
    gpio_reset_pin(BLINK_GPIO);
    gpio_set_direction(BLINK_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        ESP_LOGI(TAG, "C++2 Turning the LED %s!", s_led_state == true ? "ON" : "OFF");
        gpio_set_level(BLINK_GPIO, s_led_state);
        s_led_state = !s_led_state;
        vTaskDelay(BLINK_PERIOD_MS / portTICK_PERIOD_MS);
    }
}
```

The most important thing is the `extern "C"` for `app_main`.

No other changes were required - nothing needed to be told about the name change and the CMake setup compiled up `main.cpp` without issue.

ESP-NOW
-------

PlatformIO installs ESP-IDF to `~/.platformio/packages/framework-espidf`.

Look at the `version.txt` file there:

```
$ cd ~/.platformio/packages/framework-espidf
$ cat version.txt 
5.2.1
```

The directory contains the same as the corresponding tag in the ESP-IDF repo. In this case <https://github.com/espressif/esp-idf/tree/v5.2.1>

There's also an `examples/wifi`:

```
$ cd ~/.platformio/packages/framework-espidf/examples/wifi/espnow
$ find . -type f | sort
./CMakeLists.txt
./main/CMakeLists.txt
./main/espnow_example.h
./main/espnow_example_main.c
./main/Kconfig.projbuild
./README.md
```

I created a new ESP-IDF PlatformIO project called "esp-now-get-started-c3" and replaced the auto-generated `main.c` with the result of merging the example's `espnow_example.h` into the `espnow_example_main.c` file and, rather than the `CONFIG_...` values (derived from `Kconfig.projbuild`), I hardcoded these values:

```
#define ESPNOW_WIFI_MODE WIFI_MODE_STA
#define ESPNOW_WIFI_IF   ESP_IF_WIFI_STA

static const char *ESPNOW_PMK = "pmk1234567890123";
static const char *ESPNOW_LMK = "lmk1234567890123";
static const int ESPNOW_CHANNEL = 1;
static const int ESPNOW_SEND_COUNT = 100;
static const int ESPNOW_SEND_DELAY = 1000;
static const int ESPNOW_SEND_LEN = 10;
#undef ESPNOW_ENABLE_LONG_RANGE
#undef ESPNOW_ENABLE_POWER_SAVE
```

And trimmed off the `CONFIG_` from the places where these values are needed.

The result can be found in [`main.c`](main.c).

I then compiled and uploaded the program to two different C3 boards one after the other. After programming the second one and switching to the serial monitor, I saw:

```
Reconnecting to /dev/ttyACM0 ........    Connected!
I (5512) espnow_example: Start sending broadcast data
I (6512) espnow_example: send data to ff:ff:ff:ff:ff:ff
I (7512) espnow_example: send data to ff:ff:ff:ff:ff:ff
...
```

On plugging back in the first board as well, the two discover each other and the second board starts outputting:

```
I (78512) espnow_example: Receive 0th broadcast data from: 64:e8:33:88:a6:20, len: 10
I (79512) espnow_example: send data to ff:ff:ff:ff:ff:ff
I (79512) espnow_example: Receive 1th broadcast data from: 64:e8:33:88:a6:20, len: 10
I (79512) espnow_example: Start sending unicast data
I (79512) espnow_example: send data to 64:e8:33:88:a6:20
I (80522) espnow_example: send data to 64:e8:33:88:a6:20
I (80522) espnow_example: Receive 2th broadcast data from: 64:e8:33:88:a6:20, len: 10
I (81522) espnow_example: send data to 64:e8:33:88:a6:20
I (82522) espnow_example: send data to 64:e8:33:88:a6:20
```

You can watch what the other board is outputting with screen:

```
I (7943) espnow_example: Receive 1th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (8933) espnow_example: Receive 2th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (9933) espnow_example: Receive 3th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (10933) espnow_example: Receive 4th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (11933) espnow_example: Receive 5th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (12933) espnow_example: Receive 6th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (13933) espnow_example: Receive 7th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (14933) espnow_example: Receive 8th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (15933) espnow_example: Receive 9th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (16933) espnow_example: Receive 10th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
I (17933) espnow_example: Receive 11th unicast data from: 40:4c:ca:fa:f6:a8, len: 10
```

Unfortantely, the logger is just outputting a `CR` character so, actually it displays a bit more weirdly that that (it doesn't return to the start of the line on each new line).

It seems `screen` can't handle mapping `CR` to `CRLF` (see the answers to this StackExchange [question](https://unix.stackexchange.com/q/238774/111626)). But sophisticated terminal emulators can handle this (including `esp_idf_monitor`).

Once 100 unicast messages have been sent the program shutsdown and you have to press RESET on both boards to get things going again.
