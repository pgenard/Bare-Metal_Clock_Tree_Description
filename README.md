# Bare-Metal Clock Tree Description on Emulated i.MX8MP with QEMU.

## About

**Bare-Metal Clock Tree Description** demonstrates bare-metal software development on an **ARMv8-A** processor with a cooperative scheduler. The project configures the **Generic Interrupt Controller** (**GICv3**), **NXP i.MX8 UART** to receive interrupts, the **i.MX8MP General Purpose Timer** (**GPT**) and **i.MX8MP Clock Tree Description**.

A simple command-line interface illustrates how UART interrupts, exception handling, timer configuration and low-level hardware access work together in a minimal bare-metal environment.

### Features
- Bare-metal ARMv8-A startup code,
- EL2 exception vector table,
- GICv3 initialization and interrupt routing,
- Interrupt-driven UART receive,
- Circular receive buffer,
- Polling-based UART transmit,
- Simple serial command-line interface,
- Memory inspection with debug command,
- 16 MHz GPT clock source configuration,
- GPT timer delay,
- Software IRQ handler,
- Uptime command,
- Cooperative scheduler,
- Clock tree description.

### Hints

Our QEMU emulated platform page is available here: [i.MX8MP](https://www.qemu.org/docs/master/system/arm/imx8m.html)

Our DDR base is mapped at *0x40000000* address.

Our UART base register is mapped at *0x30860000* address.

### Toolchain

The project uses the ARM GNU bare-metal toolchain: [ARM Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)

## Compiling

### Makefile
```bash
$ make
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock scheduler.c -o scheduler.s
./aarch64-none-elf-as -mcpu=cortex-a53 scheduler.s -o scheduler.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock systime.c -o systime.s
./aarch64-none-elf-as -mcpu=cortex-a53 systime.s -o systime.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock irq.c -o irq.s
./aarch64-none-elf-as -mcpu=cortex-a53 irq.s -o irq.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock gpt.c -o gpt.s
./aarch64-none-elf-as -mcpu=cortex-a53 gpt.s -o gpt.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock uart_imx8.c -o uart_imx8.s
./aarch64-none-elf-as -mcpu=cortex-a53 uart_imx8.s -o uart_imx8.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock string.c -o string.s
./aarch64-none-elf-as -mcpu=cortex-a53 string.s -o string.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock_root.c -o clock/imx8mp_clock_root.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock_root.s -o clock/imx8mp_clock_root.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock.c -o clock/imx8mp_clock.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock.s -o clock/imx8mp_clock.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock_pll.c -o clock/imx8mp_clock_pll.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock_pll.s -o clock/imx8mp_clock_pll.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock_pll_desc.c -o clock/imx8mp_clock_pll_desc.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock_pll_desc.s -o clock/imx8mp_clock_pll_desc.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock_parent.c -o clock/imx8mp_clock_parent.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock_parent.s -o clock/imx8mp_clock_parent.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock clock/imx8mp_clock_diag.c -o clock/imx8mp_clock_diag.s
./aarch64-none-elf-as -mcpu=cortex-a53 clock/imx8mp_clock_diag.s -o clock/imx8mp_clock_diag.o
./aarch64-none-elf-gcc -S -mcpu=cortex-a53 -ffreestanding -I. -Iclock main.c -o main.s
./aarch64-none-elf-as -mcpu=cortex-a53 main.s -o main.o
./aarch64-none-elf-as -mcpu=cortex-a53 startup.s -o startup.o
./aarch64-none-elf-as -mcpu=cortex-a53 vectors.s -o vectors.o
./aarch64-none-elf-as -mcpu=cortex-a53 exceptions.s -o exceptions.o
./aarch64-none-elf-ld -T linker.ld -o hello.elf scheduler.o systime.o irq.o gpt.o uart_imx8.o string.o clock/imx8mp_clock_root.o clock/imx8mp_clock.o clock/imx8mp_clock_pll.o clock/imx8mp_clock_pll_desc.o clock/imx8mp_clock_parent.o clock/imx8mp_clock_diag.o main.o startup.o vectors.o exceptions.o
./aarch64-none-elf-ld: warning: hello.elf has a LOAD segment with RWX permissions
./aarch64-none-elf-objcopy -O binary hello.elf hello.bin
```

## Executing our Serial Shell
```bash
$ ./qemu/build/qemu-system-aarch64 -M imx8mp-evk -m 512M -kernel ../Bare-Metal/Clock_Tree_7/hello.bin -display none -serial stdio -monitor telnet:127.0.0.1:1234,server,nowait
[i.MX8MP Board Emulated by QEMU]
Command> clock
[i.MX8MP Clock Tree Description]
================================
[PLL Configuration]
--------------------------------------------------------------------------
PLL          REF       CONFIG     EFFECTIVE   LOCK   BYPASS   ENABLED
--------------------------------------------------------------------------
AUDIO_PLL1     24M     650 MHz     24 MHz     YES      NO       NO
AUDIO_PLL2     24M     650 MHz     24 MHz     YES      NO       NO
VIDEO_PLL1     24M     650 MHz     24 MHz     YES      NO       NO
DRAM_PLL     24M     600 MHz     24 MHz     YES      NO       NO
GPU_PLL     24M     800 MHz     24 MHz     YES      NO       NO
VPU_PLL     24M     600 MHz     24 MHz     YES      NO       NO
ARM_PLL     24M     1000 MHz     24 MHz     YES      NO       NO
SYS_PLL1     24M     800 MHz     24 MHz     YES      NO       NO
SYS_PLL2     24M     1000 MHz     24 MHz     YES      NO       NO
SYS_PLL3     24M     1000 MHz     24 MHz     YES      NO       NO
--------------------------------------------------------------------------
[CCM Clock Roots]
------------------------------------------------
ROOT       MUX   PRE   POST   FREQUENCY   STATE
------------------------------------------------
UART1     0     1     1     24 MHz     DISABLED
GPT1     0     1     1     24 MHz     DISABLED
------------------------------------------------
Command>
```

## Debugging
```bash
$ telnet localhost 1234
```

## Author
**Pierre GENARD**
