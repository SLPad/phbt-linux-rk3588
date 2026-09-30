SLPad RK3588 Phase 0
====================

Baseline: rockchip-linux/kernel develop-6.12,
470f9dccbdc42e7b8a824d0a5c5640a10e9457d2 (6.12.69).
Board: user-confirmed SLPad v1.2 / RK3588GamePad V115.

Board DTS and RK806 definitions originate from complete additions in the
locally supplied 2024-06-24 Android 5.10 BSP, reviewed against V115 wiring.
No incomplete third-party diff was used to invent a missing driver. Original
BSP patch and schematic are local references, not build inputs.

rocknix-joypad retains ROCKNIX copyright/GPL-2.0-or-later; leds-sk9822 retains
Nicolas Belin copyright/GPL-2.0. Both were complete BSP additions. Polling,
GPIO, removal and SPI error handling use 6.12 interfaces. Focaltech reuses the
complete existing BSP driver with GPIO lookup, IRQ flags, touchscreen
coordinate transforms and kernel-read interfaces. Disabled factory tests are
excluded from composite Kbuild objects.

Build arm64 defconfig plus slpad_base.config, optionally adding
slpad_development.config. Build Image, rockchip/slpad_rk3588.dtb and modules.
All configs, board files, drivers and bindings live in this fork.

Panthor replaces vendor Mali. Debian firmware-misc-nonfree provides
arm/mali/arch10.8/mali_csffw.bin; the runtime package lock pins its version,
and the output firmware manifest records the hash/license path. AP6398S/SV
requires exact board NVRAM and Bluetooth HCD identification. Speaker amp
identity/control is insufficiently established and its GPIO is omitted.
OLED sequence is preserved; panel/touch firmware and suspend require hardware
testing. Generic baseline Meson HDMI and MAX96712 drivers have incompatible
BSP interfaces and are disabled for this board. DCPHY requires the V4L2 media
controller API even without a camera. Standard thermal maps replace the
vendor-only fan temperature list. No new U-Boot tree is created.

Compilation and targeted bindings are host-side checks only. Power rails,
charging, wireless, touch/joypad calibration, audio, fan, suspend, bootloader
compatibility and GPU execution must be verified on hardware.

GPU core/SRAM share RK806 DCDC_REG1. A standard single-regulator OPP table
replaces vendor hardware masks and dual-regulator arrays. Frequencies are
initially limited to 300-800 MHz, using the BSP nominal voltages; runtime
thermal/voltage/performance characterization remains pending.
