# Qualcomm 音频私有 UAPI 来源

## 固定来源

- 仓库：https://github.com/MiCode/vendor_qcom_opensource_audio-kernel
- 分支：`aurora-u-oss`。
- 提交：`d2ddef03a379bc82c7bcd08481e29e08bbff80eb`。
- 小米发布说明：Xiaomi 14 Ultra Android U，基于
  `LA.VENDOR.14.3.0.r1-06800-lanai.0-1`。
- 该版本包含 `build/pineapple.bzl`，提供 SM8650/Pineapple 音频实现。
  MiCode 未提供 `shennong-u-oss` 音频分支。

## 导入范围

仅导入原仓库 `include/uapi/audio/` 下的以下三个头文件，保持原始字节不变，
存放于本仓库同名目录。它们是当前 primary-hal 音效和音频路径显式引用的
接口定义；未导入其他机型的声卡配置、codec 驱动或音频模块。

| 相对路径 | 原始及导入文件 SHA-256 |
| --- | --- |
| `sound/audio_effects.h` | `19d543ca81e8fd53af9e847ae1f02ef0d532ffbbf96b5d28ec46fc53455e73ef` |
| `sound/devdep_params.h` | `b958798735adc0959c0e6f9ce147ae70ea70da484cc910f1fe5331e4d5bbb3c2` |
| `linux/msm_audio.h` | `2a4734576a3697e1540e59c409c1d5dde615fbcbf428a40b12c01a1ad7c97c66` |

标准 `headers_install` 将它们导出至 `usr/include/audio/`；Lineage 的
`generated_kernel_includes` 已导出这一搜索路径，供 `audio_kernel_headers`
和 `qti_audio_kernel_uapi` 的用户使用。原始 `devdep_params.h` 中的 `__user`
注解由标准导出脚本清理，不在导入文件中手工改写。

这些头文件用于编译用户空间，不替换同版原厂 Image 和音频模块，也不表示
已完成源码音频驱动移植或真机音效验证。将来引入完整匹配的音频源码时，
需核对本组定义及以上哈希，再统一维护来源。
