# Zulip 配方（提取中）

```toml
format = "recipkg/1"

[package]
name = "zulip"
version = "0.1.3"
kind = "software"
description = "Zulip 组织化团队通讯产品的重建配方；当前仍在提取。"
entry = "BUILD_PROMPT.md"

[dependencies.quality]
git = "https://github.com/recipkg/recipkg.git"
rev = "41e5b55c6612a9a5a10e499b5ace6a1a3fed0f2b"
subdir = "recipes/software-quality"

[dependencies.design]
git = "https://github.com/recipkg/design-directions.git"
rev = "6fdba350d1effa2d50dc0185095bb91ca7a699ee"
```

来源：[zulip/zulip](https://github.com/zulip/zulip)，观察提交 `dd5adc2de6c43bd7bac7d1a0e6dc18c9364268a7`；上游 README 声明 Apache 2.0。此配方是未完成的提取草案，完整性和重建状态见 `COVERAGE.md`。
