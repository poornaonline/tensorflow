# Hello World example

This example is designed to demonstrate the absolute basics of using [TensorFlow
Lite for Microcontrollers](https://www.tensorflow.org/lite/microcontrollers).
It includes the full end-to-end workflow of training a model, converting it for
use with TensorFlow Lite, and running inference on a microcontroller.

The sample is built around a model trained to replicate a `sine` function. It
contains implementations for several platforms. In each case, the model is used
to generate a pattern of data that is used to either blink LEDs or control an
animation.

![Animation of example running on STM32F746](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/hello_world/images/STM32F746.gif)

## Table of contents

-   [Understand the model](#understand-the-model)
-   [Deploy to Arduino](#deploy-to-arduino)
-   [Deploy to SparkFun Edge](#deploy-to-sparkfun-edge)
-   [Deploy to STM32F746](#deploy-to-STM32F746)
-   [Run the tests on a development machine](#run-the-tests-on-a-development-machine)

## Understand the model

The sample comes with a pre-trained model. The code used to train and convert
the model is available as a tutorial in [create_sine_model.ipynb](https://colab.research.google.com/github/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/hello_world/create_sine_model.ipynb).

Walk through this tutorial to understand what the model does,
how it works, and how it was converted for use with TensorFlow Lite for
Microcontrollers.

## Deploy to Arduino

The following instructions will help you build and deploy this sample
to [Arduino](https://www.arduino.cc/) devices.

![Animation of example running on Arduino MKRZERO](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/hello_world/images/arduino_mkrzero.gif)

The sample has been tested with the following devices:

- [Arduino Nano 33 BLE Sense](https://store.arduino.cc/usa/nano-33-ble-sense-with-headers)
- [Arduino MKRZERO](https://store.arduino.cc/usa/arduino-mkrzero)

The sample will use PWM to fade an LED on and off according to the model's
output. In the code, the `LED_BUILTIN` constant is used to specify the board's
built-in LED as the one being controlled. However, on some boards, this built-in
LED is not attached to a pin with PWM capabilities. In this case, the LED will
blink instead of fading.

### Install the Arduino_TensorFlowLite library

This example application is included as part of the official TensorFlow Lite
Arduino library. To install it, open the Arduino library manager in
`Tools -> Manage Libraries...` and search for `Arduino_TensorFlowLite`.

### Load and run the example

Once the library has been added, go to `File -> Examples`. You should see an
example near the bottom of the list named `TensorFlowLite:hello_world`. Select
it and click `hello_world` to load the example.

Use the Arduino IDE to build and upload the example. Once it is running,
you should see the built-in LED on your device flashing.

The Arduino Desktop IDE includes a plotter that we can use to display the sine
wave graphically. To view it, go to `Tools -> Serial Plotter`. You will see one
datapoint being logged for each inference cycle, expressed as a number between 0
and 255.

## Deploy to ESP32

The following instructions will help you build and deploy this sample
to [ESP32](https://www.espressif.com/en/products/hardware/esp32/overview)
devices using the [ESP IDF](https://github.com/espressif/esp-idf).

The sample has been tested on ESP-IDF version 4.0 with the following devices:
- [ESP32-DevKitC](http://esp-idf.readthedocs.io/en/latest/get-started/get-started-devkitc.html)
- [ESP-EYE](https://github.com/espressif/esp-who/blob/master/docs/en/get-started/ESP-EYE_Getting_Started_Guide.md)

### Install the ESP IDF

Follow the instructions of the
[ESP-IDF get started guide](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html)
to setup the toolchain and the ESP-IDF itself.

The next steps assume that the
[IDF environment variables are set](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html#step-4-set-up-the-environment-variables) :

 * The `IDF_PATH` environment variable is set
 * `idf.py` and Xtensa-esp32 tools (e.g. `xtensa-esp32-elf-gcc`) are in `$PATH`

### Generate the examples
The example project can be generated with the following command:
```
make -f tensorflow/lite/micro/tools/make/Makefile TARGET=esp generate_hello_world_esp_project
```

### Building the example

Go the the example project directory
```
cd tensorflow/lite/micro/tools/make/gen/esp_xtensa-esp32/prj/hello_world/esp-idf
```

Then build with `idf.py`
```
idf.py build
```

### Load and run the example

To flash (replace `/dev/ttyUSB0` with the device serial port):
```
idf.py --port /dev/ttyUSB0 flash
```

Monitor the serial output:
```
idf.py --port /dev/ttyUSB0 monitor
```

Use `Ctrl+]` to exit.

The previous two commands can be combined:
```
idf.py --port /dev/ttyUSB0 flash monitor
```

## Deploy to SparkFun Edge

The following instructions will help you build and deploy this sample on the
[SparkFun Edge development board](https://sparkfun.com/products/15170).

![Animation of example running on SparkFun Edge](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/hello_world/images/sparkfun_edge.gif)

If you're new to using this board, we recommend walking through the
[AI on a microcontroller with TensorFlow Lite and SparkFun Edge](https://codelabs.developers.google.com/codelabs/sparkfun-tensorflow)
codelab to get an understanding of the workflow.

### Compile the binary

The following command will download the required dependencies and then compile a
binary for the SparkFun Edge:

```
make -f tensorflow/lite/micro/tools/make/Makefile TARGET=sparkfun_edge hello_world_bin
```

The binary will be created in the following location:

```
tensorflow/lite/micro/tools/make/gen/sparkfun_edge_cortex-m4/bin/hello_world.bin
```

### Sign the binary

The binary must be signed with cryptographic keys to be deployed to the device.
We'll now run some commands that will sign our binary so it can be flashed to
the SparkFun Edge. The scripts we are using come from the Ambiq SDK, which is
downloaded when the `Makefile` is run.

Enter the following command to set up some dummy cryptographic keys we can use
for development:

```
cp tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.2.0/tools/apollo3_scripts/keys_info0.py \
tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.2.0/tools/apollo3_scripts/keys_info.py
```

Next, run the following command to create a signed binary:

```
python3 tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.2.0/tools/apollo3_scripts/create_cust_image_blob.py \
--bin tensorflow/lite/micro/tools/make/gen/sparkfun_edge_cortex-m4/bin/hello_world.bin \
--load-address 0xC000 \
--magic-num 0xCB \
-o main_nonsecure_ota \
--version 0x0
```

This will create the file `main_nonsecure_ota.bin`. We'll now run another
command to create a final version of the file that can be used to flash our
device with the bootloader script we will use in the next step:

```
python3 tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.2.0/tools/apollo3_scripts/create_cust_wireupdate_blob.py \
--load-address 0x20000 \
--bin main_nonsecure_ota.bin \
-i 6 \
-o main_nonsecure_wire \
--options 0x1
```

You should now have a file called `main_nonsecure_wire.bin` in the directory
where you ran the commands. This is the file we'll be flashing to the device.

### Flash the binary

Next, attach the board to your computer via a USB-to-serial adapter.

**Note:** If you're using the [SparkFun Serial Basic Breakout](https://www.sparkfun.com/products/15096),
you should [install the latest drivers](https://learn.sparkfun.com/tutorials/sparkfun-serial-basic-ch340c-hookup-guide#drivers-if-you-need-them)
before you continue.

Once connected, assign the USB device name to an environment variable:

```
export DEVICENAME=put your device name here
```

Set another variable with the baud rate:

```
export BAUD_RATE=921600
```

Now, hold the button marked `14` on the device. While still holding the button,
hit the button marked `RST`. Continue holding the button marked `14` while
running the following command:

```
python3 tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.2.0/tools/apollo3_scripts/uart_wired_update.py \
-b ${BAUD_RATE} ${DEVICENAME} \
-r 1 \
-f main_nonsecure_wire.bin \
-i 6
```

You should see a long stream of output as the binary is flashed to the device.
Once you see the following lines, flashing is complete:

```
Sending Reset Command.
Done.
```

If you don't see these lines, flashing may have failed. Try running through the
steps in [Flash the binary](#flash-the-binary) again (you can skip over setting
the environment variables). If you continue to run into problems, follow the
[AI on a microcontroller with TensorFlow Lite and SparkFun Edge](https://codelabs.developers.google.com/codelabs/sparkfun-tensorflow)
codelab, which includes more comprehensive instructions for the flashing
process.

The binary should now be deployed to the device. Hit the button marked `RST` to
reboot the board. You should see the device's four LEDs flashing in sequence.

Debug information is logged by the board while the program is running. To view
it, establish a serial connection to the board using a baud rate of `115200`.
On OSX and Linux, the following command should work:

```
screen ${DEVICENAME} 115200
```

You will see a lot of output flying past! To stop the scrolling, hit `Ctrl+A`,
immediately followed by `Esc`. You can then use the arrow keys to explore the
output, which will contain the results of running inference on various `x`
values:

```
x_value: 1.1843798*2^2, y_value: -1.9542645*2^-1
```

To stop viewing the debug output with `screen`, hit `Ctrl+A`, immediately
followed by the `K` key, then hit the `Y` key.


## Deploy to STM32F746

The following instructions will help you build and deploy the sample to the
[STM32F7 discovery kit](https://os.mbed.com/platforms/ST-Discovery-F746NG/)
using [ARM Mbed](https://github.com/ARMmbed/mbed-cli).

![Animation of example running on STM32F746](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/hello_world/images/STM32F746.gif)

Before we begin, you'll need the following:

- STM32F7 discovery kit board
- Mini-USB cable
- ARM Mbed CLI ([installation instructions](https://os.mbed.com/docs/mbed-os/v5.12/tools/installation-and-setup.html))
- Python 2.7 and pip

Since Mbed requires a special folder structure for projects, we'll first run a
command to generate a subfolder containing the required source files in this
structure:

```
make -f tensorflow/lite/micro/tools/make/Makefile TARGET=mbed TAGS="CMSIS disco_f746ng" generate_hello_world_mbed_project
```

This will result in the creation of a new folder:

```
tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/hello_world/mbed
```

This folder contains all of the example's dependencies structured in the correct
way for Mbed to be able to build it.

Change into the directory and run the following commands, making sure you are
using Python 2.7.15.

First, tell Mbed that the current directory is the root of an Mbed project:

```
mbed config root .
```

Next, tell Mbed to download the dependencies and prepare to build:

```
mbed deploy
```

By default, Mbed will build the project using C++98. However, TensorFlow Lite
requires C++11. Run the following Python snippet to modify the Mbed
configuration files so that it uses C++11:

```
python -c 'import fileinput, glob;
for filename in glob.glob("mbed-os/tools/profiles/*.json"):
  for line in fileinput.input(filename, inplace=True):
    print line.replace("\"-std=gnu++98\"","\"-std=c++11\", \"-fpermissive\"")'

```

Finally, run the following command to compile:

```
mbed compile -m DISCO_F746NG -t GCC_ARM
```

This should result in a binary at the following path:

```
./BUILD/DISCO_F746NG/GCC_ARM/mbed.bin
```

To deploy, plug in your STM board and copy the file to it. On MacOS, you can do
this with the following command:

```
cp ./BUILD/DISCO_F746NG/GCC_ARM/mbed.bin /Volumes/DIS_F746NG/
```

Copying the file will initiate the flashing process. Once this is complete, you
should see an animation on the device's screen.


```
screen /dev/tty.usbmodem14403 9600
```

In addition to this animation, debug information is logged by the board while
the program is running. To view it, establish a serial connection to the board
using a baud rate of `9600`. On OSX and Linux, the following command should
work, replacing `/dev/tty.devicename` with the name of your device as it appears
in `/dev`:

```
screen /dev/tty.devicename 9600
```

You will see a lot of output flying past! To stop the scrolling, hit `Ctrl+A`,
immediately followed by `Esc`. You can then use the arrow keys to explore the
output, which will contain the results of running inference on various `x`
values:

```
x_value: 1.1843798*2^2, y_value: -1.9542645*2^-1
```

To stop viewing the debug output with `screen`, hit `Ctrl+A`, immediately
followed by the `K` key, then hit the `Y` key.

### Run the tests on a development machine

To compile and test this example on a desktop Linux or macOS machine, first
clone the TensorFlow repository from GitHub to a convenient place:

```bash
git clone --depth 1 https://github.com/tensorflow/tensorflow.git
```

Next, `cd` into the source directory from a terminal, and then run the following
command:

```bash
make -f tensorflow/lite/micro/tools/make/Makefile test_hello_world_test
```

This will take a few minutes, and downloads frameworks the code uses. Once the
process has finished, you should see a series of files get compiled, followed by
some logging output from a test, which should conclude with
`~~~ALL TESTS PASSED~~~`.

If you see this, it means that a small program has been built and run that loads
the trained TensorFlow model, runs some example inputs through it, and got the
expected outputs.

To understand how TensorFlow Lite does this, you can look at the source in
[hello_world_test.cc](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/micro/examples/hello_world/hello_world_test.cc).
It's a fairly small amount of code that creates an interpreter, gets a handle to
a model that's been compiled into the program, and then invokes the interpreter
with the model and sample inputs.
