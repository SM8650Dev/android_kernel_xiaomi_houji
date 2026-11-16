# Xiaomi hardware identification

The shennong-u-oss release references drivers/misc/hwid but omits its files.
The release also ignores this directory in .gitignore; remove that exclusion
so the restored source is included in future clones.
Restore the GPL-2.0 implementation from:

- Repository: https://github.com/Allay-OS/android_kernel_xiaomi_sm8650
- Source commit: e420e8b2b2cd0b185b331e6a1e916e7ea8d530e3
- Imported files: drivers/misc/hwid/{Kconfig,Makefile,hwid.c,hwid.h}

The donor targets zorn. Its SM8650 product-name mapping is extended to the
projects observed in Xiaomi 14 Global OS3.0.305.0.WNCMIXM's hwid.ko:
1 shennong, 2 houji, 3 manet, 4 aurora, 5 suiren, 6 ruyi, 7 goku, 13 zorn.
Unknown SoCs/projects return unknown rather than a fabricated device identity.
The N3=2 value matters to the USB controller's houji-specific workaround;
some public kernels use a different numbering scheme and cannot be copied as-is.

The driver preserves four read-only module parameters: project, hwid_value,
project_adc, build_adc. Bootloader/module-loader values supply the identity;
no houji hardware ID or country is hardcoded as a default.

The stock module's nine exported functions, hardware-version masks and four
/sys/hwid attributes were checked against its symbol table, relocations and
ARM64 instructions. ADC attributes preserve the stock signed-decimal format;
project and hwid attributes use hexadecimal. The header retains legacy country
spellings used by other Xiaomi modules. Sysfs init failures release their kobject.

This restores the missing driver source and build integration. It does not prove
that the full 6.1.25 kernel is compatible with newer 6.1.157 vendor firmware;
external modules and real-device behavior still need validation.

## Build validation

With the actual gki_defconfig + pineapple_GKI.config + houji_GKI.config and
Clang r547379, configuration generation and the drivers/misc/hwid/ Kbuild
target pass. CONFIG_MI_HARDWARE_ID=m and CONFIG_QCOM_SOCINFO=m remain enabled.
The compiled hwid.o exports the same nine function names as the stock module.
The USB DWC3 controller and eUSB2 repeater objects that include hwid.h also compile.

The existing format errors in WALT, Gunyah, MHI, irqchip, QRTR and other
Qualcomm drivers have been fixed in separate commits. The full Image/modules
build and modules_install now pass with CONFIG_WERROR=y, producing an AArch64
hwid.ko with all nine expected exports. The wcd939x-i2c consumer also compiles.
See build.md for the configuration, artifacts and remaining bringup limits.

The module has not been loaded on a device. Matching export names alone does
not establish symbol-version or binary-module compatibility with stock firmware.
