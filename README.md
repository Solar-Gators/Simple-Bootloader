# STM32U575 Minimal Bootloader

A lightweight bootloader for the STM32U575 that resides at the start of internal flash and transfers execution to an application stored at a fixed offset.

## Current Scope

- Single application image
- No A/B slots or rollback
- No CRC or signature verification
- No CAN-based update

## Memory Map

```text
0x08000000  Bootloader (64 KB)
0x08010000  Application
0x081FFFFF  End of flash
```

## How It Works

On reset, the MCU begins executing the bootloader at `0x08000000`. The bootloader reads the first two words at `APP_START_ADDR` and checks whether they form a valid application vector table. If valid, it transfers control to the application.

## Vector Table

Every Cortex-M image begins with a vector table at its base address:

```text
APP_START_ADDR + 0x00  Initial MSP
APP_START_ADDR + 0x04  Reset_Handler
APP_START_ADDR + 0x08  NMI_Handler
APP_START_ADDR + 0x0C  HardFault_Handler
...
```

The bootloader uses the first two entries to launch the application:

- **Word 0** — initial stack pointer (MSP)
- **Word 1** — application `Reset_Handler` address

The jump sequence:

1. Disable interrupts
2. Clear bootloader interrupt state
3. Set `SCB->VTOR = APP_START_ADDR`
4. Load the application MSP
5. Branch to the application `Reset_Handler`

## Linker Configuration

### Bootloader

```text
ROM origin = 0x08000000
ROM length = 64K
```

### Application

```text
ROM origin = 0x08010000
ROM length = remaining flash
```

## Future Improvements

- Bootloader entry condition (e.g. GPIO pin or flag)
- CAN-based firmware update
- Image metadata / header format
- CRC or hash verification
- Dual-slot updates with rollback support

