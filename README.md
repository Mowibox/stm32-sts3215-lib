# STM32 Feetech Servo Driver

A lightweight C library designed for STM32 microcontrollers (using STM32CubeIDE and HAL) to control Feetech Serial Bus Smart Servos (specifically the [**STS3215**](https://www.feetechrc.com/74v-19-kgcm-plastic-case-metal-tooth-magnetic-code-double-axis-ttl-series-steering-gear.html)) magnetic encoding series via RS485.

![Feetech](https://img.shields.io/badge/Feetech-Bus%20Servo-ee2524?)
![STM32](https://img.shields.io/badge/STM32-HAL%20C-03234b?logo=stmicroelectronics&logoColor=white)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-lightgrey.svg)](https://opensource.org/licenses/Apache-2.0)
![Issues](https://img.shields.io/github/issues/Mowibox/stm32-sts3215-lib)
![Stars](https://img.shields.io/github/stars/Mowibox/stm32-sts3215-lib?style=social)

Designed to be easily dropped into any STM32CubeMX generated project.

## Table of contents

| Section | Description |
| ------- | ----------- |
| [Features](#features) | Key functionalities of the library |
| [Author](#author) | Main contributors information |
| [Documentation](#documentation) | Links to protocol manual and STS3215 memory table registry |
| [Usage & Integration](#usage--integration) | Instructions for integrating the `inc/` and `src/` files into a project. |
| [Contributions](#contributions) | How to contribute to the repository |
| [License](#license) | Licensing information |

## Features

## Author

| |
| :---: |
| <img src="https://github.com/Mowibox.png" width="100"> |
| [@Mowibox](https://mowibox.github.io)<br>Ousmane THIONGANE |

## Documentation

* [Communication Protocol Manual](./docs/protocol_manual.md) (Corrected definitions for Sync Read, Checksums, and limits).
* [STS3215 Memory Table](./docs/sts3215_memory_table.md) (Register addresses, default values, and operational constraints).

## Usage & Integration

Usart 
* Mode Asyncronous
* Baudrate 1000000bps
* Assertion Time : 8 Sample time unit
* Deassertion Time : 8 Sample time unit

In DMA section of usart add two channel for rx and tx and change priority to high

NVIC section 
DMA1 channelX interrupt : enable 
DMA1 channelY interrupt : enable 
USART2 global interrupt : enable 
with all of them with the same preemption priority

## Contributions

Contributions are always welcome!

* **Report Issues:** Found a bug or have a feature request? Create a new issue [here.](https://github.com/Mowibox/stm32-sts3215-lib/issues/new/choose)
* **Fix Bugs & Add Features:** Find out where you can lend a hand by checking out [existing issues.](https://github.com/Mowibox/stm32-sts3215-lib/issues)

## License

This project is licensed under the Apache 2.0 License. See the [LICENSE](https://github.com/Mowibox/stm32-sts3215-lib/blob/main/LICENSE) file for more details.
