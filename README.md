# gesture-iot.github.io

### 1. Video

[Final Demo](https://drive.google.com/file/d/1iwqYzj8SS5lWVos3FztdtkV0NV1YEx-l/view?usp=sharing)

### 2. Images

![d153b2c3c004a44683c97a60ac9878b5](https://github.com/user-attachments/assets/a08ed438-301a-42d6-8829-91285e7f4adf)

<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/5426d0d1-5095-4c59-95f2-5cd6ccac7cc7" />

![31f04a7e3b46b15270a800cd6164f40a](https://github.com/user-attachments/assets/8c326e3c-dfa7-4cc3-9b6f-d6bb104ce2e7)

![微信图片_20251208151054_262_1](https://github.com/user-attachments/assets/3f60e561-e5a1-4f48-906f-32fe289cdb3b)

![微信图片_20251208151035_253_1](https://github.com/user-attachments/assets/c474ccad-e33f-4dc6-b585-6a2d8d9cfa4b)

### 3. Results

Our final solution is a fully working gesture-controlled smart-home system consisting of a wrist-worn controller and an appliance-side gateway. On the wrist, an ATmega328PB reads 6-axis motion data from the LSM6DSO IMU over I²C and classifies four directional gestures (UP, DOWN, LEFT, RIGHT). A flex-sensor front-end (implemented as a voltage divider followed by an LM358 comparator) provides a clean digital signal that allows the ATmega to distinguish between OPEN and CLOSE hand states. Recognized gestures are encoded as compact symbols (e.g., ‘U’, ‘D’, ‘L’, ‘R’, ‘O’, ‘C’) and transmitted over UART to the wrist-side ESP32-S2, which bridges from UART to Wi-Fi.

On the appliance side, two ESP32 terminals receive Wi-Fi packets from the wrist-side controller, but only  after a pointing gesture–based device-selection (“pairing”) step . When the user performs a pointing gesture toward an appliance, the wristband activates its high-power IR LED; the appliance-side IR receiver that detects this pulse reports its device ID back over Wi-Fi, completing a lightweight pairing handshake. Only after this pairing confirmation are subsequent gesture commands routed to that device.

In the final system, the first terminal ESP32 controls  Device 2 (Motor/Fan) ,  Device 1 (LED Strip) , and  Device 4 (LCD-based Air Conditioner UI) . The fan interprets UP/DOWN gestures as PWM speed adjustments and LEFT/RIGHT as mode switching (continuous vs. intermittent). The LED strip supports ON/OFF, brightness UP/DOWN, and animation-mode changes. The AC UI renders temperature, mode, and power state on a TFT display, and responds to gestures mapped to temperature increments and mode transitions. A second terminal ESP32 implements Device 3 (Phone/Music Controller) using USB HID Consumer Control, enabling gesture-driven volume and track control on a paired smartphone.

Across all devices, the full interaction pipeline— pointing gesture → IR-based device selection → pairing acknowledgment → gesture sensing → ATmega classification → UART transfer → Wi-Fi transmission → dual-ESP32 parsing → device-specific actuation —remains robust, with end-to-end latency well under one second. Console logs on both ESP32 terminals confirm stable pairing, correct command routing, and reliable actuation even under rapid and repeated user input.

Hardware-wise, we successfully brought up all critical subsystems: the custom IMU I²C driver, the custom UART driver between the ATmega and ESP32, the flex-sensor comparator circuit, the high-current IR-LED driver and IR receiver for device selection, and the ESP32↔ESP32 Wi-Fi link with PWM-based motor control. The IMU runs at 104 Hz and is down-sampled to a lower rate suitable for gesture recognition, while still meeting our accuracy needs for reliable direction detection. Overall, our prototype met all of the expected design goals for sensing, gesture recognition, wireless communication, IR-based device selection, and end-device control; the only planned features not fully implemented in the final demo were the speaker-based audio feedback and the haptic vibration motor.

#### 3.1 Software Requirements Specification (SRS) Results

Overall, the system met most of the software requirements, including accurate IMU gesture detection, reliable IR-based selection, low-latency ESP32 command processing, and timely LCD UI updates. The IMU pipeline exceeded expectations, achieving stable readings with accuracy better than ±10%, and gesture-processing latency remained well below the 500 ms requirement due to a 5 Hz gesture-output rate and ~100 Hz processing loop. IR detection range significantly outperformed the original software expectation of 3 m, requiring us to update the requirement based on real behavior.

One requirement—haptic vibration feedback—was not validated because the vibration motor hardware did not arrive in time. Aside from this, all other requirements were confirmed through UART logging, timing tests, and real-world device interactions. These results indicate that the software stack was robust and aligned with the functional goals, with requirement changes mainly driven by shifts in hardware availability and observed system performance.

| ID     | Description (Measurable Requirement)                                                                     | Verification Method                                                                 | Validation Outcome                                                                                                                                |
| ------ | -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| SRS-01 | The IMU sensor shall detect wrist rotation and acceleration with an accuracy of ±10%.                   | Use usrt output wrist ratation and compare with real movitation                     | Confirmed, we using the uart to ouput IMU's raw data and the accuracy up to 1%g                                                                   |
| SRS-02 | The IR transmitter shall emit a signal detectable by receivers within 3 m in normal lighting.            | Measure IR detection distance and angle using IR receiver output.                   | Confirmed, the ir receiver can attacked by the IR over the whole lab lenght(about 5m)                                                             |
| SRS-03 | The ESP32 shall process gesture inputs and send control signals with latency <500 ms.                   | Control buzzer to beep when gesture happens, using stopwatch to record respond time | Confirmed, our output frequency of gesture is 5Hz and prcess frquency is about 100Hz, when we check the output on uart, it generate consistently. |
| SRS-04 | The vibration motor shall generate a feedback pulse of 200 ± 50 ms duration after each pairing success. | Measure motor activation time with stopwatch.                                       | Not confirmed, our cvibration motor didn't arrived                                                                                                |
| SRS-05 | The LCD display shall update device and command information within 1s of command recognition.            | Observe update speed using timestamped logs or slow-motion video.                   | Confirmed, we can see from the veidio that when the device ACK. The LCD show up the device name immediately and also the command.                 |
| SRS-06 | The receiver ESP32 shall control all four output devices.                                                | Test each device's response                                                         | Confirmed, when we check all the device, it can be controled by our receiver ESP32                                                                |



#### 3.2 Hardware Requirements Specification (HRS) Results

| ID                                            | Description                                                                                                                                                                          | Verification Method                                                              | Validation outcome                                                                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **HRS-01 (IR Selection and Detection)** | The controller shall transmit modulated IR bursts at 38 kHz. A valid hit shall be recognized and reported to the controller within 50 ms.                                            | Point to different terminals many times to verify the selection.                 | Comfirmed, our IR receiver is designed as only ACK to 38 kHz IR signal and whenever we trigger our IR LED the receiver can ACK to it |
| **HRS-02 (IR Coverage and Range)**      | The IR link shall maintain reliable operation at 3 m ± 45° in front of the receiver under standard indoor lighting.                                                                | Point at the same terminal from different angles many times and show the result. | Not confirmed, we try to point the receiver in almost every angle, and the receiver can ACK in about 5m ±90°                       |
| **HRS-03 (IMU Sampling and Interface)** | The IMU module shall output 3-axis acceleration and angular-velocity data at ≥ 100 Hz, communicating with the controller via I²C (400 kHz).                                        | Use UART to print the raw data from the IMU and check if it is stable.           | Confirmed, we configure the LSM6DSO IMU with an output data rate of 104 Hz                                                           |
| **HRS-04 (PWM Output Hardware)**        | The fan terminal’s ESP32 shall generate motor-control signals with different frequencies and duty cycles.                                                                           | Change the command and observe the output PWM using an oscilloscope.             | Confirmed but not using oscilloscope, we can obverse it we fan change mode.                                                          |
| **HRS-05 (Power and Protection)**       | All boards shall operate from a regulated 5 V ± 5 % supply; the IR-LED driver shall limit continuous current to ≤ 200 mA and include reverse-polarity and over-current protection. | Use a DMM to test all connections, voltage, and current.                         | Not comfirmed, ESP32 are supplied by 3.3V                                                                                            |

Overall, our system met most of the key hardware requirements, including stable IMU sampling, reliable IR-based device selection, and responsive PWM control on the motor terminal. Some requirements changed during development as our understanding of the hardware matured. For example, the IR coverage requirement expanded once we observed significantly greater range and angle tolerance than expected, while the audio subsystem requirement shifted from a speaker–amplifier design to USB HID–based phone control due to hardware delays.


### 4. Conclusion

Through this project, we learned how to integrate sensing, wireless communication, IR-based device pairing, and real-time UI feedback into a single working system, and how tightly each subsystem influences overall responsiveness and reliability. Our IMU pipeline and gesture-recognition logic stabilized early, and the UART-to-Wi-Fi communication between the ATmega328PB and the ESP32-S2 performed better than expected, consistently maintaining low latency during rapid gesture input. We are especially proud that our custom multi-device protocol, pairing design, and TFT UI—along with the motor controller, LED strip driver, and USB-based music controller—came together into a coherent multi-device ecosystem that operated reliably in the final demo.

We also gained experience adapting our design when unexpected constraints arose. Early in the project, we planned to integrate a  **speaker system driven by a dedicated audio amplifier** , but due to delayed hardware delivery, we pivoted to a different solution: using the ESP32-S2’s **USB HID Consumer Control** interface to control music playback on a smartphone. This change required rethinking our command pipeline but ultimately resulted in a more stable and demonstrable audio-control subsystem. Similarly, when the IR LED hardware arrived late, we redesigned significant portions of the pairing workflow and restructured how gesture commands were queued and dispatched. We tuned gesture output to about 5 Hz for improved stability and optimized SPI drawing on the TFT-LCD to maintain UI responsiveness on the appliance-side ESP32.

Several things went well: our IR pairing protocol worked reliably after refinement; our dual-ESP32 device architecture (one controlling the motor, LED strip, and AC-style TFT UI, and another dedicated to phone/music control) proved clean and modular; and the end-to-end interaction pipeline remained responsive even under repeated commands. These accomplishments gave us a deeper understanding of embedded protocol design, multi-device coordination, and timing-sensitive UI rendering.

At the same time, there are areas we would approach differently. Earlier hardware procurement—especially for the IR LEDs, IR receivers, and audio amplifier—would have significantly reduced debugging pressure. More systematic timing tests, automated Wi-Fi throughput measurements, and earlier power-budget analysis would also have helped us catch bottlenecks sooner. We also encountered challenges we did not anticipate, such as Wi-Fi AP instability with multiple ESP32 clients, IR-receiver sensitivity variations across devices, and unexpected delays in SPI-based TFT rendering before optimization.

As next steps, we would like to refine the IR subsystem for better range and noise tolerance, reduce system-wide power consumption, and migrate everything onto a custom PCB for improved robustness. Incorporating a richer TFT interface, restoring the original speaker-and-amplifier audio subsystem, and exploring more advanced gesture-recognition algorithms would move the prototype closer to a polished, consumer-ready wearable.

## References

References

1. STMicroelectronics LSM6DSO Datasheet
2. ESP32-S2 Technical Reference Manual
3. ST7735 / ST7789 TFT LCD Controller Datasheet
4. IR Receiver Module Application Notes

Library

1. Arduino Core for ESP32-S2
2. Adafruit_GFX Library
3. Adafruit_ST7735 / ST7789 Library
4. WiFi.h & WiFiClient.h
