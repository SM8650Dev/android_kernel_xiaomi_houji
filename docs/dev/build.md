# Houji kernel compilation

## Verified source configuration

The MiCode shennong-u-oss release is based on Linux 6.1.25. With the fixes
through 56b6e8d9ecb5, the source builds using the Android tree's Clang r547379
and kernel-build-tools, with gki_defconfig, vendor/pineapple_GKI.config and
vendor/houji_GKI.config merged in that order.

CONFIG_WERROR=y, CONFIG_CFI_CLANG=y and CONFIG_MODVERSIONS=y remain enabled.
No missing-symbol override, prebuilt kernel or stock binary module is used.
Each compiler or diagnostic defect is committed separately by driver; the
QRTR padding use-after-free, LED ID argument and MTD reason-enum corrections
are separate fixes. The duplicated top-level DTS recipes are also removed.

## Reproduce from the Android tree root

The matching official DTS checkout must be present at
kernel/xiaomi/houji-devicetrees, as declared in lineage.dependencies.

```sh
export PATH="$PWD/prebuilts/clang/host/linux-x86/clang-r547379/bin:$PWD/prebuilts/kernel-build-tools/linux-x86/bin:$PATH"
kernel_src="$PWD/kernel/xiaomi/houji"
kernel_out="$PWD/out/houji-kernel-check"
make -C "$kernel_src" O="$kernel_out" ARCH=arm64 LLVM=1 LLVM_IAS=1 gki_defconfig
KCONFIG_CONFIG="$kernel_out/.config" "$kernel_src/scripts/kconfig/merge_config.sh" \
    -m -O "$kernel_out" "$kernel_out/.config" \
    "$kernel_src/arch/arm64/configs/vendor/pineapple_GKI.config" \
    "$kernel_src/arch/arm64/configs/vendor/houji_GKI.config"
make -C "$kernel_src" O="$kernel_out" ARCH=arm64 LLVM=1 LLVM_IAS=1 olddefconfig
make -C "$kernel_src" O="$kernel_out" ARCH=arm64 LLVM=1 LLVM_IAS=1 \
    DTC_FLAGS=-@ -j36 Image modules dtbs
make -C "$kernel_src" O="$kernel_out" ARCH=arm64 LLVM=1 LLVM_IAS=1 \
    INSTALL_MOD_PATH="$PWD/out/houji-kernel-modules" INSTALL_MOD_STRIP=1 \
    -j12 modules_install
```

No unit tests are required for this bringup. The validation consists of the
actual compiler, linker, modpost, module installation and device-tree tools.

## Android product host-tool builds

Lineage passes a glibc 2.17 sysroot through HOSTCFLAGS and HOSTLDFLAGS for
this 6.1 kernel. resolve_btfids must pass its host CFLAGS as EXTRA_CFLAGS to
libsubcmd, just as it does for libbpf. libsubcmd initializes its own CFLAGS
and otherwise discards the sysroot inherited through the environment.

Without that forwarding, a newer host can compile libsubcmd against
/usr/include/stdlib.h and emit __isoc23_strtol/strtoul/strtoull references,
then fail to link them against the older sysroot libc. Keep the headers and
libraries on the same sysroot; no BTF disablement or libc symbol shim is
needed. This affects the Android product build even when the standalone
host build above succeeds against the native host libc.

## Observed results

On the bringup server at /SSD/cm23.2:

- Full Image/modules build succeeds, including modpost and BTF generation.
- Image: out/houji-kernel-check/arch/arm64/boot/Image, 33,823,232 bytes.
- Build release: 6.1.25-g56b6e8d9ecb5.
- Image SHA-256: b4aca7aaafa2681417b2f2f4387fadf0a367368adf5d16a2dbede62145aaffb2.
- modules_install succeeds and installs 335 source-built modules.
- drivers/misc/hwid/hwid.ko is an AArch64 ELF module with all nine expected
  exported APIs; it is included in the installed module set.
- The full dtbs target succeeds. The official sources still emit DTC warnings;
  this result is not a claim that DTS bindings are warning-free.
- vendor/qcom/pineapple.dtb, pineapple-v2.dtb and houji-sm8650-overlay.dtbo
  compile with symbols enabled. fdtoverlay applies the houji overlay to each
  base successfully.

Logs are under out/: houji-kernel-build.log, houji-kernel-dtbs.log,
houji-kernel-dtbs-all.log and houji-kernel-modules-install.log.
The temporary verification outputs and compiler-fix scripts are not source
artifacts and are not committed.

## Remaining system integration

The current stock-derived load lists name 164 distinct modules that are not
emitted by this configuration. Per-list counts overlap: boot 19, recovery 61,
system 2, vendor 143. The local audit is out/houji-kernel-module-gaps.json.
These counts compare filenames; they do not prove that every associated
function is absent. Some functions may be built in or selected by another
configuration, while others require external sources and porting.

Camera, WLAN and display DTS checkouts declared as dependencies remain porting
inputs; this validation does not build or integrate the complete external
camera, display, audio, GPU and connectivity module stack. Keep missing-module
checks enabled and audit each stock load entry before changing the lists.

Stock OS3.0.305.0.WNCMIXM uses a 6.1.157 kernel. Building this 6.1.25 release
and applying its own overlays does not establish compatibility with that newer
vendor ABI, firmware or hardware configuration. Product kernel packaging,
complete ROM construction and real-device boot remain unverified. These output
files alone are not a validated flashable release.
