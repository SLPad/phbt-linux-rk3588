SLPad core DTS audit against Rockchip develop-6.12
================================================

Verified on 2026-09-30 using git ls-remote against
https://github.com/rockchip-linux/kernel.git refs/heads/develop-6.12.
Official HEAD is 470f9dccbdc42e7b8a824d0a5c5640a10e9457d2, identical to the
pinned baseline. Thus no speculative SoC register/clock/interrupt backport is
needed. rk3588.dtsi and rk3588s.dtsi are used unchanged at that commit.

Reference files at the pinned official commit:
- rk3588-rk806-single.dtsi (single-PMIC regulator definitions)
- rk3588-evb.dtsi (UART/MMC/USB core setup)
- rk3588-evb1-lp4.dtsi (USB/DP orientation and mode-switch graph)
- rk3588s-rk806-dual.dtsi (PMIC control pin examples)

Layers:
slpad-rk3588-core.dtsi -> official SoC/PMIC includes and V115 core bus wiring;
slpad-console.dtsi -> panel, inputs, LEDs, sensors, sound, battery, wireless;
slpad_rk3588.dtsi -> combines core first, then console peripherals.

PMIC/power: replace the copied 5.10 regulator table with an include of the
current official rk3588-rk806-single.dtsi. Inherit current regulator limits,
CPU-little 800mV initialization, 837.5mV NLDO3, and S0/S3 suspend defaults.
Keep V115-specific external RK860-2/RK860-3 regulators on I2C0 and NPU RK860-2
on I2C1, and board PMIC control pins; these are actual schematic/BSP wiring,
not EVB assumptions. CONFIG_REGULATOR_RK860X is explicitly built in, fixing a
missing driver for CPU-big/NPU supplies. ROCKCHIP_THERMAL is also built in.
Suspend/voltage behavior still requires real-board validation.

UART: explicitly select UART2 M0, matching V115 GPIO0_B5/B6 and serial2.
Storage: keep V115 8-bit 1.8V HS400 eMMC, 4-bit removable SD, and SDIO Wi-Fi;
inherit controller clocks/resets/interrupts from current official SoC files.
No EVB peripheral GPIO or dual-PMIC definitions are copied onto V115.
USB/Type-C: keep DWC3 role-switch graph and the single wired Type-C0 port;
add current USBDP mode-switch, remove unused svid, and retain orientation
and DP altmode endpoints. Disable EHCI/OHCI controllers whose HOST PHYs are
disabled/unwired. Fix the role endpoint unit address. Keep the V115 FUSB302,
VBUS GPIO and SBU pins rather than replacing them with an EVB port layout.
Display: current DSI/DCPHY/HDPTX/VOP definitions with V115 OLED sequence.
Allocate the 128MiB display CMA pool dynamically, rather than retaining a
fixed physical HDMI-RX reservation on a board with HDMI RX disabled.
GPU: standard Panthor CSF binding and one-rail 300-1000MHz OPP table; shared
GPU core/SRAM RK806 supply, no vendor supported-hw masks.
Network: no wired Ethernet PHY in the V115 schematic. GMAC1 pins overlap
console buttons, so both GMAC controllers stay disabled. Maintenance uses
USB network adapters/gadget provisioning and SDIO AP6398S/SV Wi-Fi.

Target DTB compilation and focused schemas are checked. The official BSP
also retains vendor RK806 properties/node names outside generic upstream
schemas; this audit is not a claim that the entire BSP dtbs_check passes.

The Panthor OPP table is copied from Linux v6.12 rk3588-opp.dtsi.
It does not reproduce vendor Mali PVTM/leakage/bin or low-temperature
voltage adjustment. Compilation does not establish electrical compatibility.
DP0 sound requires ROCKCHIP_HDMI and ROCKCHIP_SPDIF; Android audio routing
and EDID/ELD must be investigated on the original running BSP.
