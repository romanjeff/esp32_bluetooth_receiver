ESP32 A2DP Sink (BT Classic Receiver) based on ESP-IDF example.

Currently tested on ESP32 DevkitC v4 (ESP32-4D0WD) with PCM5102 module. Builds and flashes via VSCODE ESP-IDF extension. Pairs and plays music cleanly with volume control from Windows laptop, pairs but no audio stream from iOS device - "BT HCI packet not finished" error codes need debug.

Flash Method: UART (USB-C port to CP2102 on dev board). Need CP2102 drivers.
port to use should display as ESP32 available option.
set device target as plain esp32 (esp32.cfg) "custom board." openOCD board file for /interface/ftdi/devkitj-v1.cfg will appear but this does not impact ability to flash or monitor. Debugging not tested yet, may require jlink if debugging requires JTAG instead of UART.

Connections:

ESP32 | PCM5102 module
3.3V  | "3.3V"/Any VDD ('VCC' is input to regulator, not needed if using ESP32's regulated 3.3V)
GND   | GND
GND   | DMP - no deemphasis
GND   | SCL/SCK - use BCK and LCK only/3-wire mode
GND   | FMT - select i2s classic instead of left-justified
3.3V  | FLT - use lower latency interpolation filter
3.3V  | XMT - SW mute disabled. Tie to ESP32 GPIO and add supporting BT code for remote mute support.

I2S pins currently selected:
LCK - GPIO 22
BCK - GPIO 26
DIN - GPIO 25




| Supported Targets | ESP32 |
| ----------------- | ----- |

A2DP-SINK EXAMPLE
======================

Example of A2DP audio sink role

This is the example of API implementing Advanced Audio Distribution Profile to receive an audio stream.

This example involves the use of Bluetooth legacy profile A2DP for audio stream reception, AVRCP for media information notifications, and I2S for audio stream output interface.

Applications such as bluetooth speakers can take advantage of this example as a reference of basic functionalities.

## How to use this example

### Hardware Required

To play the sound, there is a need of loudspeaker and possibly an external I2S codec. Otherwise the example will only show a count of audio data packets received silently. Internal DAC can be selected and in this case external I2S codec may not be needed.

For the I2S codec, pick whatever chip or board works for you; this code was written using a PCM5102 chip, but other I2S boards and chips will probably work as well. The default I2S connections are shown below, but these can be changed in menuconfig:

| ESP pin   | I2S signal   |
| :-------- | :----------- |
| GPIO22    | LRCK         |
| GPIO25    | DATA         |
| GPIO26    | BCK          |

If the internal DAC is selected, analog audio will be available on GPIO25 and GPIO26. The output resolution on these pins will always be limited to 8 bit because of the internal structure of the DACs.

### Configure the project

```
idf.py menuconfig
```

* Choose external I2S codec or internal DAC for audio output, and configure the output PINs under A2DP Example Configuration

* For AVRCP CT Cover Art feature, is enabled by default, we can disable it by unselecting menuconfig option `Component config --> Bluetooth --> Bluedroid Options --> Classic Bluetooth --> AVRCP Features --> AVRCP CT Cover Art`. This example will try to use AVRCP CT Cover Art feature, get cover art image and count the image size if peer device support, this can be disable in `A2DP Example Configuration --> Use AVRCP CT Cover Art Feature`.

### Build and Flash

Build the project and flash it to the board, then run monitor tool to view serial output.

```
idf.py -p PORT flash monitor
```

(To exit the serial monitor, type ``Ctrl-]``.)

## Example Output

After the program is started, the example starts inquiry scan and page scan, awaiting being discovered and connected. Other bluetooth devices such as smart phones can discover a device named "ESP_SPEAKER". A smartphone or another ESP-IDF example of A2DP source can be used to connect to the local device.

Once A2DP connection is set up, there will be a notification message with the remote device's bluetooth MAC address like the following:

```
I (106427) BT_AV: A2DP connection state: Connected, [64:a2:f9:69:57:a4]
```

If a smartphone is used to connect to local device, starting to play music with an APP will result in the transmission of audio stream. The transmitting of audio stream will be visible in the application log including a count of audio data packets, like this:

```
I (120627) BT_AV: A2DP audio state: Started
I (122697) BT_AV: Audio packet count 100
I (124697) BT_AV: Audio packet count 200
I (126697) BT_AV: Audio packet count 300
I (128697) BT_AV: Audio packet count 400
```

The output when receiving a cover art image:

```
I (53349) RC_CT: AVRC metadata rsp: attribute id 0x80, 1000748
I (53639) RC_CT: Cover Art Client final data event, image size: 14118 bytes
```

Also, the sound will be heard if a loudspeaker is connected and possible external I2S codec is correctly configured. For ESP32 A2DP source example, the sound is noise as the audio source generates the samples with a random sequence.

## Troubleshooting
* For current stage, the supported audio codec in ESP32 A2DP is SBC. SBC data stream is transmitted to A2DP sink and then decoded into PCM samples as output. The PCM data format is normally of 44.1kHz sampling rate, two-channel 16-bit sample stream. Other SBC configurations in ESP32 A2DP sink is supported but need additional modifications of protocol stack settings.
* As a usage limitation, ESP32 A2DP sink can support at most one connection with remote A2DP source devices. Also, A2DP sink cannot be used together with A2DP source at the same time, but can be used with other profiles such as SPP and HFP.
