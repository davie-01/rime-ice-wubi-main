# rime-ice-wubi

把 **雾凇拼音** 与 **极点五笔86** 整合成一套可以自由切换的 RIME 配置，界面与提示风格统一采用雾凇。

原来两套分别部署时会互相覆盖（先部署一套，再复制另一套部署后前一套失效），本仓库解决了这个冲突，
并对两边做了一些适配。

## 方案列表

| 顺序 | schema_id | 显示名 |
|---|---|---|
| 1 | `rime_ice` | 雾凇拼音（全拼） |
| 2 | `double_pinyin_flypy` | 小鹤双拼（支持全拼混输） |
| 3 | `wubi86_jidian` | 极点五笔86 |
| 4 | `numbers` | 大写数字 |

方案选单 `F4` / `Control+\``。开关项折叠成一行，选中展开，等于「先选方案、再做二次配置」。

## 主要特性

- **五笔**：`z` + 拼音反查，候选后面显示五笔编码
- **小鹤双拼**：全拼和双拼都能输入，候选后面显示小鹤编码 —— 过渡期不用翻键位表
- **简入繁出**：由五笔方案的「简体→繁体」开关控制，状态可记忆
- **动态调频**：拼音与五笔均已开启用户词典
- 编码提示使用绿色 `0x5AC461`，取自极点五笔原皮肤

## 部署

1. 备份并清空 `~/Library/Rime/`（保留 `*.userdb/`、`installation.yaml` 可延续词频）
2. 把本仓库内容（`LICENSE`、`licenses/`、`README.md`、两个 .md 文档除外）复制进去
3. 删除旧的 `default.custom.yaml`、`squirrel.custom.yaml`、`weasel.custom.yaml` —— 它们优先级更高，留着会覆盖本配置
4. 鼠须管 →「重新部署」

Windows（小狼毫）同理，配置目录为 `%APPDATA%\Rime`。

## 版本基线

`upstream-baseline` 标签指向未经修改的上游文件。

```sh
git diff upstream-baseline HEAD    # 本方案相对上游的全部改动
```

## 来源与许可

本仓库是两个上游项目的衍生作品，未包含任何原创词库。

| 上游项目 | 许可证 | 本仓库中的文件 |
|---|---|---|
| [iDvel/rime-ice](https://github.com/iDvel/rime-ice) | GPL-3.0 | `rime_ice.*`、`double_pinyin_flypy.*`、`melt_eng.*`、`radical_pinyin.*`、`default.yaml`、`squirrel.yaml`、`weasel.yaml`、`symbols_*.yaml`、`cn_dicts/`、`en_dicts/`、`opencc/`、`lua/`、`custom_phrase.txt` |
| [KyleBing/rime-wubi86-jidian](https://github.com/KyleBing/rime-wubi86-jidian) | Apache-2.0 | `wubi86_jidian*.yaml`、`pinyin_simp.*`、`numbers.schema.yaml`、`wubi86_jidian.ico` |

许可证全文：

- [`LICENSE`](LICENSE) —— GPL-3.0
- [`licenses/GPL-3.0-rime-ice.txt`](licenses/GPL-3.0-rime-ice.txt)
- [`licenses/Apache-2.0-rime-wubi86-jidian.txt`](licenses/Apache-2.0-rime-wubi86-jidian.txt)

由于包含 GPL-3.0 的内容，本仓库整体以 **GPL-3.0** 分发。Apache-2.0 部分保留其原有声明。

### 已作的修改

Apache-2.0 第 4(b) 条要求标明对原文件的修改，以下为本仓库对极点五笔文件的改动：

- `wubi86_jidian.schema.yaml` —— 快捷键改为与雾凇一致；`zh_trad` 去掉 `reset` 以支持状态记忆；启用用户词典动态调频；启用 `pinyin_simp` 依赖
- `wubi86_jidian_trad.schema.yaml` —— 同上
- `wubi86_jidian.dict.yaml` —— 移除 iOS Hamster 空词库的 import
- `pinyin_simp.schema.yaml` —— 移除缺失的 `stroke` 依赖，仅作五笔反查码表使用
- 未纳入：`default.custom.yaml`、`squirrel.custom.yaml`、`weasel.custom.yaml`、`wubi86_jidian_pinyin` 系列方案

对雾凇拼音文件（GPL-3.0）的改动：

- `default.yaml` —— 方案列表改为上述四项；`switcher/save_options` 增加 `zh_trad`、移除 `full_shape`
- `squirrel.yaml` —— 注释（编码提示）颜色改为 `0x5AC461`；增加 `app_options`；候选横排
- `double_pinyin_flypy.schema.yaml` —— `speller/algebra` 由 `xform` 改为 `derive` 以支持全拼混输；
  `translator/comment_format` 改为显示小鹤双拼编码；相应关闭 `corrector.lua`
- `custom_phrase_double.txt` —— 新增（上游需使用者自行创建）
- 未纳入：`t9` 及其余双拼方案、`others/`、`recipe.yaml`、部分 `en_dicts/cn_en_*.txt`
