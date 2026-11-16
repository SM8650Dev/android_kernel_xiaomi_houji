# External driver source provenance

The MiCode shennong release separates kernel core, WLAN, camera and display.
Some necessary Qualcomm driver repositories have no MiCode shennong branch.
The following additional porting inputs are pinned to the public Qualcomm
`LA.VENDOR.14.3.0.r1-05600-lanai.0.xml` manifest (SM8650/pineapple family):

| Repository under `https://git.codelinaro.org/clo/la/` | Upstream revision | Local directory |
| --- | --- | --- |
| `platform/vendor/qcom/opensource/securemsm-kernel` | `99e7a9ca5236209af5afadda64be30d472550c68` | `kernel/xiaomi/houji-modules/securemsm-kernel` |
| `platform/vendor/opensource/dataipa` | `723906905cbf3e7a59ec8924e9d4bbe81e72097c` | `kernel/xiaomi/houji-modules/dataipa` |

These are explicit porting baselines, not a claim that Qualcomm's vendor
manifest is identical to Xiaomi's `KERNEL.PLATFORM.3.0.r1-03200-kernel.0-1`
release or the Android 16 stock modules. Source builds, symbol checks and
runtime validation are still required.

Install `device/xiaomi/houji/docs/dev/houji.xml` as the local manifest to
provide the `houji-clo` remote used by `lineage.dependencies`. No change to
the Lineage main manifest or roomservice implementation is required.

Local fixes are committed on `houji-port` in each added repository. The
manifest pins upstream inputs; it does not contain unpublished local fixes.
Before sharing a build checkout, publish those commits to maintained forks
and update the manifest/dependency revisions to those forks together.

## RMNET porting inputs

The source-build adapters additionally use the following pinned Qualcomm
repositories under the same CodeLinaro remote. These supply RMNET core,
control, memory and extension drivers omitted from the MiCode core tree.

| Repository | Upstream revision | Local directory |
| --- | --- | --- |
| `platform/vendor/qcom/opensource/datarmnet` | `0f7747af0bd8e8f0603c726ffa0e330f927e1602` | `kernel/xiaomi/houji-modules/datarmnet` |
| `platform/vendor/qcom/opensource/datarmnet-ext` | `24380abaae2422e1cf0b2b22a85263459f2fedc3` | `kernel/xiaomi/houji-modules/datarmnet-ext` |

Local fixes remain on the porting branches and must be published with the
other driver fixes before these upstream input pins can reproduce a build.

## Multimedia and synchronization porting inputs

The next source adapters use the public CodeLinaro GitHub mirror,
https://github.com/CodeLinaro-mirror/. The upstream release branches have
continued changing through 2026, so these inputs are pinned to historical
commits from the Android 14 / Pineapple development period rather than their
current heads. They are porting baselines, not exact matches to the MIXM ROM.

| Mirror repository | Upstream revision | Commit date |
| --- | --- | --- |
| la_platform_vendor_opensource_mm-drivers | 129a3797e8dd1163aa49809b26355c8bb0573584 | 2023-06-27 |
| la_platform_vendor_opensource_mmrm-driver | 0ecad902add56e93e083c1a4acab37b05b14c508 | 2023-01-13 |
| la_platform_vendor_opensource_synx-kernel | d5bed2898aa514562a4fb6a5a0ae2732833a54f9 | 2023-12-20 |

mm-drivers supplies actual hardware fence, software sync fence and external
display implementations; mmrm-driver supplies the clock resource manager.
synx-kernel supplies Synx and IPCLite. Adapters select Pineapple and pass real
upstream Module.symvers files. Display continues to use the MiCode shennong
implementation and securemsm's actual HDCP interface.

The houji-clo-mirror remote is defined in the device's docs/dev/houji.xml
local manifest. Dependency revisions above identify upstream inputs; the
local strict-compiler and diagnostic fixes on houji-port are not published.
As with the earlier networking drivers, publish the local port branches to
maintained forks and update both manifest and dependency pins before claiming
that a fresh upstream-only sync reproduces these builds.

Selected stock miev and NFC fallbacks and their validation are described in
device/xiaomi/houji/docs/dev/prebuilt-modules.md. They do not change the
source-built kernel Image or permit arbitrary stock modules.

## Camera and DSP source integration

Camera and cameralog continue to use MiCode's
vendor_qcom_opensource_camera-kernel shennong-u-oss source already listed
in lineage.dependencies. The camera adapter selects the supplied Pineapple
configuration, Synx vendor implementation, MMRM, secure camera and SMMU proxy.
It uses actual upstream symbol tables and generates the original build header.

FastRPC and CDSP loader use the GitHub mirror
CodeLinaro-mirror/la_platform_vendor_qcom_opensource_dsp-kernel at
a976ff28ccb56d1b227a857a7cf5f92cd60985f2 (2023-12-18).
This historical Pineapple-capable input supplies frpc-adsprpc and cdsp-loader.
It is a porting baseline, not proof of equivalence to MIXM HAL/firmware.
The adapter selects CONFIG_ARCH_PINEAPPLE and the supplied DSP profile header.
Trace include paths and strict diagnostic fixes are separate local port commits.

The device adapters and verification boundaries are described in
device/xiaomi/houji/docs/dev/camera-dsp-modules.md.
The mirror pin identifies the upstream input; local camera and DSP fixes
must be published to maintained forks, with manifest/dependency revisions
updated, before a new checkout can reproduce the patched build.

## Houji camera device trees

MiCode's camera-devicetree repository has no shennong-u-oss branch. Its
aurora-u-oss revision c07bd5b2c17c003829b8b04d776ee28d66bbdad1 contains the
actual houji-sm8650-camera-sensor.dtsi as well as common Pineapple and v2
camera descriptions. The device-tree integration uses those houji files,
not aurora sensor wiring. The camera driver's dt-bindings supply the matching
camera constants; the sensor description explicitly includes RPMH level
definitions.

The core DT repository links this external source at qcom/camera and includes
common camera nodes in the Pineapple/PineappleP bases and houji sensor nodes
in the houji overlay. Both product overlays apply successfully. Across 47
active camera nodes, 917 compared properties match the corresponding stock
MIXM DT variants after provider phandle normalization; v1/v2 differ only in
the expected IPE clock-rate table when compared across variants. This checks
DT descriptions, not camera HAL or hardware operation.

The dependency pin records the public MiCode input. The local header fix and
core DT integration must also be published to forks and pinned before a new
sync reproduces the patched build.
