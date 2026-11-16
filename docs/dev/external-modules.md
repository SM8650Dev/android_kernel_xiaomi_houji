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
