# STM32 Feetech Servo Driver

A lightweight C library designed for STM32 microcontrollers (using STM32CubeIDE and HAL) to control Feetech Serial Bus Smart Servos (specifically the [**STS3215**](https://www.feetechrc.com/74v-19-kgcm-plastic-case-metal-tooth-magnetic-code-double-axis-ttl-series-steering-gear.html)) Magnetic Encoding series via RS485.

![Feetech](https://img.shields.io/badge/Feetech-Bus%20Servo-ee2524?)
![STM32](https://img.shields.io/badge/STM32-HAL%20C-03234b?logo=stmicroelectronics&logoColor=white)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-lightgrey.svg)](https://opensource.org/licenses/Apache-2.0)
![CI: Linters](https://github.com/Mowibox/stm32-sts3215-lib/actions/workflows/lint.yml/badge.svg)
![CI: Unit Tests](https://github.com/Mowibox/stm32-sts3215-lib/actions/workflows/tests.yml/badge.svg)
![Issues](https://img.shields.io/github/issues/Mowibox/stm32-sts3215-lib)
![Stars](https://img.shields.io/github/stars/Mowibox/stm32-sts3215-lib?style=social)

<p align="center">
  <img alt="STS3215 Feetech Servo" src="sts3215.png"/>
</p>

Designed to be easily dropped into any STM32CubeMX generated project.

## Table of contents

| Section | Description |
| ------- | ----------- |
| [Features](#features) | Key functionalities of the library |
| [Author](#author) | Main contributors' information |
| [Documentation](#documentation) | Links to protocol manual and STS3215 memory table registry |
| [Usage & Integration](#usage--integration) | Instructions for integrating the `inc/` and `src/` files into a project. |
| [Testing](#testing) | Running the host-side unit tests locally and in CI |
| [Contributions](#contributions) | How to contribute to the repository |
| [License](#license) | Licensing information |

## Features

- **RS485 hardware DE management** - Driver Enable pin controlled directly by the STM32 USART peripheral in RS485 mode (no GPIO toggling, no timing risk)
- **Non-blocking DMA transfers** - TX and RX both run on DMA with IDLE line detection, freeing the CPU and RTOS scheduler during bus transactions
- **Layered architecture** - Protocol logic (`sts3215_protocol.c`) is fully decoupled from the HAL layer (`sts3215_hal.c`); the protocol layer has zero STM32 dependency and can be unit-tested on a host PC with any C99 compiler
- **Complete register coverage** - All STS3215 EEPROM and SRAM registers documented with addresses, sizes, default values, units, and access rights (`sts3215_regs.h`)
- **Full instruction set** - Builders for every protocol instruction: `PING`, `READ`, `WRITE`, `REG_WRITE`, `ACTION`, `SYNC_READ`, `SYNC_WRITE`, `RESET`, and `RESET_TURNS`
- **Atomic motion commands** - Position, speed, and acceleration packed into a single write frame (registers `0x29`→`0x2F`) to prevent partial-update race conditions
- **Synchronised multi-servo control** - `REG_WRITE` + broadcast `ACTION` pattern for frame-accurate simultaneous motion across any number of servos on the bus
- **EEPROM write protection** - Explicit `UnlockEEPROM` / `LockEEPROM` helpers enforce the mandatory write-lock sequence before any persistent configuration change
- **Unit conversion helpers** - Steps <-> degrees <-> radians, raw current -> milliamps, raw voltage -> volts, all as inline-friendly functions.

## Author

| |
| :---: |
| <img src="https://github.com/Mowibox.png" width="100"> |
| [@Mowibox](https://mowibox.github.io)<br>Ousmane THIONGANE |

## Documentation

- [Communication Protocol Manual](./docs/protocol_manual.md) (Feetech Official Protocol Manual for Smart Servos).
- [STS3215 Memory Table](./docs/sts3215_memory_table.md) (Register addresses, default values, and operational constraints).

## Usage & Integration

### 1 - Hardware Requirements

| Component | Specification |
| --------- | ------------- |
| MCU | STM32G431KBT6 (or any STM32 with USART + DMA + RS485 hardware mode) |
| Servo | Feetech STS3215 |
| Interface | RS485 transceiver with DE pin |

---

### 2 - STM32CubeIDE .ioc Configuration

#### 2.1 Pinout

**Pinout used in this documentation:**

In the **Pinout & Configuration** tab, assign:

```text
PA1 → USART2_DE  
PA2 → USART2_TX
PA3 → USART2_RX
```

> [!NOTE]
> You can assign and use any other USART that has a driver enable pin, provided you make the necessary changes to the files afterward.

#### 2.2 USART2 - Mode

Go to the **Connectivity** section and configure the chosen USART:

```text
  Mode: Asynchronous
  Hardware Flow Control (RS485): ☑ Enabled
```

#### 2.3 USART2 - Parameter Settings

Click on the **Parameter Settings** tab of your USART and configure it as shown below:

| Parameter | Value |
| --------- | ----- |
| Baud Rate | `1000000 Bits/s` |
| Word Length | `8 Bits` |
| Parity | `None` |
| Stop Bits | `1` |
| Data Direction | `Receive and Transmit` |
| Over Sampling | `16 Samples` |
| Single Wire (Half Duplex) | `Disable` |
| DE Polarity | `High` |
| DE Assertion Time | `8 Sample Time Unit` |
| DE Deassertion Time | `8 Sample Time Unit` |

#### 2.4 USART2 - DMA Settings

Go to **System Core > DMA** tab to add two DMA channels:

| DMA Request | Direction | Priority | Mode | Mem Increment |
| ----------- | --------- | -------- | ---- | ------------- |
| `USART2_TX` | Memory To Peripheral | High | Normal | Enabled |
| `USART2_RX` | Peripheral To Memory | High | Normal | Enabled |

#### 2.5 NVIC

In the **NVIC** tab, enable the following interrupts and set them to the **same preemption priority** (e.g., `5`):

```text
☑ USART2 global interrupt - Preemption Priority 5
☑ DMA1 Channel[X] global interrupt - Preemption Priority 5
☑ DMA1 Channel[Y] global interrupt - Preemption Priority 5
```

Where [X] and [Y] are the channels you have defined for your DMA (TX/RX).

> [!WARNING]
> Do not forget to **regenerate code** before proceeding!

---

### 3 - File Placement

Copy the library files into your CubeIDE project as follows:

```text
YourProject/
└── Core/
    ├── Inc/
    │   ├── main.h 
    │   ├── sts3215_regs.h ← 
    │   ├── sts3215_protocol.h ←
    │   └── sts3215_hal.h ←
    └── Src/
        ├── main.c 
        ├── sts3215_protocol.c ←
        └── sts3215_hal.c ←
```

---

### 4 - Includes

Add the following to your `main.c`:

```c
/* USER CODE BEGIN Includes */
#include "sts3215_regs.h"
#include "sts3215_protocol.h"
#include "sts3215_hal.h"
/* USER CODE END Includes */
```

---

### 5 - Minimal Example

The following example initializes the driver, sends a **PING** instruction to servo ID 1, then moves it back and forth between two positions every 2 seconds.

```c
/* USER CODE BEGIN Includes */
#include "sts3215_regs.h"
#include "sts3215_protocol.h"
#include "sts3215_hal.h"
/* USER CODE END Includes */

/* USER CODE BEGIN PV */
static STS3215_HAL_Handle_t hservo;
static uint8_t tx_frame[STS3215_TX_BUF_SIZE];
/* USER CODE END PV */

/* USER CODE BEGIN 0 */
static void on_reply(const STS3215_Reply_t *reply, uint8_t idx, STS3215_Status_t status, void *ctx)
{
    (void)idx;
    (void)ctx;

    g_reply_id       = reply->id;
    g_reply_error    = reply->error;
    g_reply_status   = status;
    g_reply_data_len = reply->data_len;

    uint8_t copy_len = (reply->data_len <= STS3215_REPLY_DATA_MAX) ? reply->data_len : STS3215_REPLY_DATA_MAX;
    for (uint8_t i = 0; i < copy_len; i++) {
        g_reply_data[i] = reply->data[i];
    }

    g_reply_received = 1;
}

static void on_error(STS3215_HAL_Error_t err, void *ctx)
{
    (void)ctx;
}
/* USER CODE END 0 */

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_DMA_Init();
    MX_USART2_UART_Init();

    /* USER CODE BEGIN 2 */

    /*  Initialise the driver */
    STS3215_HAL_Init(&hservo, &huart2, 10U, on_reply, on_error, NULL);
    STS3215_HAL_RegisterInstance(&hservo);

    /* Ping servo ID 1 */
    int16_t len = STS3215_BuildPing(tx_frame, sizeof(tx_frame), 1U);
    if (len > 0) {
        STS3215_HAL_SendFrame(&hservo, tx_frame, (uint16_t)len, false, 1U);
    }

    STS3215_MotionCmd_t cmd = {
        .acceleration  = 50,
        .target_pos    = 1024,
        .running_time  = 0,     /* unused in position mode */
        .running_speed = 1000,  
    };

    uint32_t last_move_ms = HAL_GetTick();
    uint8_t  toggle       = 0U;

    /* USER CODE END 2 */

    while (1)
    {
        /* USER CODE BEGIN WHILE */
        STS3215_HAL_Process(&hservo);

        bool bus_is_free = STS3215_HAL_IsIdle(&hservo);
        uint32_t elapsed_ms = HAL_GetTick() - last_move_ms;
        bool time_has_elapsed = (elapsed_ms >= 2000U);

        if (bus_is_free && time_has_elapsed)
        {
            last_move_ms = HAL_GetTick();

            /* Alternate the target position on each iteration:
             *   toggle = 0 → move to position 1024 (≈ 90°)
             *   toggle = 1 → move to position 3072 (≈ 270°) */
            if (toggle == 0U)
            {
                cmd.target_pos = 1024;
                toggle = 1;
            }
            else
            {
                cmd.target_pos = 3072;
                toggle = 0;
            }

            len = STS3215_BuildMotionCmd(tx_frame, sizeof(tx_frame), 1U, &cmd);

            if (len > 0)
            {
                STS3215_HAL_SendFrame(&hservo, tx_frame, (uint16_t)len, false, 1U);
            }
        }

        /* USER CODE END WHILE */
    }
}
```

More detailed examples are available in the [`examples/`](./examples) folder.

## Testing

The protocol layer (`sts3215_protocol.c`) has zero STM32 dependency, so it can be
unit-tested on the host instead of on target hardware:

```sh
cd tests
make test
```

See [`tests/`](./tests) for more details.

## Contributions

Contributions are always welcome!

- **Report Issues:** Found a bug or have a feature request? Create a new issue [here.](https://github.com/Mowibox/stm32-sts3215-lib/issues/new/choose)
- **Fix Bugs & Add Features:** Find out where you can lend a hand by checking out [existing issues.](https://github.com/Mowibox/stm32-sts3215-lib/issues)

## License

This project is licensed under the Apache 2.0 License. See the [LICENSE](https://github.com/Mowibox/stm32-sts3215-lib/blob/main/LICENSE) file for more details.
