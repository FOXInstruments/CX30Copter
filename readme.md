# CX30 Quadrocopter custom firmware

C lang Quadrocopter custom firmware project<br>
Control loops using High Order Sliding modes<br>
Build on IAR Workbench or PlatformIO and STM8S_StdPeriph_Lib v2.3.1

# Hardware specification
Controller STM8S005K6T6C<br>
HSI f_cpu=16MHz<br>
Acceleration and Gyro chip MPU6052<br>
RF Transceiver BK2425

## CPU Pin configuration:
PA1 - LED control. Active level High. After init flashes (period 300ms) while payload from remote control (BK2425) will be received. After it is on always<br>
PB2 - Analog input ADC. Battery voltage from divider with ratio 42/(42+49)<br>
PB4 - I2C_SCL connected to MPU6052<br>
PB5 - I2C_SDA connected to MPU6052<br>
PC1 - PWM out for Right front motor (16KHz)<br>
PC2 - PWM out for Right back motor (16KHz)<br>
PC3 - LED control. Active level High. After init it is off. When payload (BK2425) is received it turns on. When more than 10 requests have no payload than it starts flash<br>
PC5 - SPI_SCK connected to BK2425<br>
PC5 - SPI_MOSI connected to BK2425<br>
PC5 - SPI_MOSO connected to BK2425<br>
PE5 - SPI_SS connected to BK2425<br>
PD0 - LED control. Active high level. Battery voltage > 3.2V - LED On. Battery voltage <= 3.2V - LED Flashes with period 500ms. Range 3.0V to 4.2V<br>
PD2 - PWM out for Left back motor (16KHz)<br>
PD3 - PWM out for Left front motor (16KHz)<br>
PD5, PD6 - UART telemetry data flow


## SPI BK2425
STM8 is master<br>
Clock 4MHz<br>
File with init sequence of chip BK2425: BK_Init.txt<br>
Data from remote control.<br>
SPI_CMD 0x61 (Read RX payload) 8 bytes:<br>
0x80 default value. Left stick Y position (0x0 - 0xFF) - vertical speed control<br>
0x80 default value. Left stick X position (0x0 - 0xFF) - afterburner control<br>
0x40 default value. Left bottom rocker. Left button click -1, Right click +1 (0x0E - 0x72), Long Beep when cross 0x40. Value to adjust quadcopter pitch orientation<br>
0x80 default value. Right stick Y position (0x60 - 0xA0). Quadrocoper pitch control<br>
0x80 default value. Right stick X position (0x60 - 0xA0). Quadrocoper roll control<br>
0x40 default value. Reserved<br>
0x40 default value. Right bottom rocker. Left button click -1, Right click +1 (0x0E - 0x72), Long Beep when cross 0x40. Value to adjust quadcopter roll orientation<br>
0x00 default value. 0x40 - Left upper 2nd button<br>
Pool rate: 500Hz when in Flight mode (payload received) and 20Hz after init and if there is no payload more than 10 requests

## I2C MPU6052
Clock 200KHz<br>
File with init sequence of chip MPU6052: MPU_Init.txt<br>
Request data from MPU:
1. 3 axis of Accel
2. Temperature
3. 3 axis of Gyro
Pool rate: 500Hz when in Flight mode (payload recived) and after init no Pool and if there is no payload from BK2425 more than 10 requests than stop MPU pool

## Motors
PWM frequency 16kHz<br>
Normal flight duty cycle from 20% - 85%<br>
afterburner duty cycle from 85% - 95%<br>

# Firmware specification
Use fixedpoint arithmetic. Sliding mode control loops.

## Control loops
1. Pitch control. Right stick X controls pitch angel (maximum angels +-30 deg).
2. Roll control. Right stick Y controls roll angel (maximum angels +-30 deg). When stick is in dead zone +-3 from default value than target Roll = 0 and target Pitch = 0
3. Sliding mode vertical speed control loop. Left stick uses to control it. Dead zone +-10 from default value. When Y pos in dead zone than keep vertival speed equal 0 (keep altitude)

## Calculate state of quadrocopter
Roll, Pitch, Yaw - angels<br>
Wroll, Wpitch, Wyaw - angular speed<br>
x, y, z - center mass position (initial condition (0, 0, 0) takeoff point)<br>
Vx, Vy, Vz - center mass speed<br>
Ax, Ay, Az - center mass acceleration

## WiFi camera module
Link 65Mbs<br>
I = 250mA<br>
IP gateway = 172.16.10.1

## UART
Baudrate 19200<br>
Add module to send state of quadrocoper, bk_payload, voltage on UART2 with frequency 1hz.<br>
Speeds 19200 and 56700 set through #define<br>
Use interrupt to free CPU
