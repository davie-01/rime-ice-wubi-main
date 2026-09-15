# 雾凇拼音 + 极点五笔 整合方案

整合自 `rime-ice-main`（雾凇拼音）与 `rime-wubi86-jidian-master`（极点五笔86）。
界面、符号、标点、快捷键等**提示与风格统一采用雾凇拼音**，五笔仅保留自身必需的功能键位。

## 一、方案列表（default.yaml）

| 顺序 | schema_id | 显示名 | 来源 |
|---|---|---|---|
| 1 | rime_ice | 雾凇拼音 | 雾凇 |
| 2 | double_pinyin_flypy | 小鹤双拼 | 雾凇 |
| 3 | wubi86_jidian | 极点五笔86 | 五笔 |
| 4 | numbers | 大写数字 | 五笔 |

方案选单：`F4` / `Control+\`` （雾凇风格，caption「方案选单」）。

### 两层结构（fold_options: true）

RIME 的方案选单里，开关项永远只属于**当前方案**；`fold_options: true` 把它们折叠成一行，
选中该行才展开成逐项选择，等于"选好方案后再做二次配置"：

```
第一层：1. 雾凇拼音  2. 小鹤双拼  3. 极点五笔86  4. 大写数字
        5. 中 / 繁 / 全 / 扩 / ．，      ← 当前方案（五笔）的开关，折叠成一行
                                          选中它 ↓
第二层：中→En   简体→繁体   半角→全角   常用→扩展   。，→．，
```

切到雾凇拼音时，这一行自动变成雾凇自己的开关（中 / ￥ / 简 / 😄 / 半 / 词）。

### 极点五笔的五个开关

| 开关 | option | 作用 |
|---|---|---|
| 中 → En | `ascii_mode` | 中英文切换，切到 En 直接上屏字母 |
| 简体 → 繁体 | `zh_trad` | 简入繁出，OpenCC `s2hk.json`（香港繁体） |
| 半角 → 全角 | `full_shape` | 全角时字母数字标点变全宽字符 |
| 常用 → 扩展 | `extended_charset` | 字符集过滤，打不出生僻字时切「扩展」 |
| 。， → ．， | `ascii_punct` | 中文标点 / 英文标点 |

原「简入繁出」方案与「简体→繁体」开关是同一件事（同一套码表，只是 `zh_trad` 默认值不同），
已取消独立方案，改由开关控制；`zh_trad` 去掉 `reset` 并加入 `switcher/save_options`，切过去会被记住。
`default.yaml` 里注释掉的一行取消注释即可恢复成独立方案。

## 二、为什么原来两套互相覆盖

| 冲突文件 | 说明 | 本方案处理 |
|---|---|---|
| `default.custom.yaml`（五笔） | 用 patch 整体改写 `schema_list`，后部署的一方把对方方案列表全部顶掉；同时改写 `punctuator` / `key_binder` / `recognizer`，破坏雾凇的符号、v 模式、反查等 | **删除**，方案列表直接写进雾凇的 `default.yaml` |
| `squirrel.custom.yaml`（五笔） | 覆盖 `squirrel.yaml` 的皮肤与候选样式 | **删除**，统一用雾凇 `squirrel.yaml` |
| `weasel.custom.yaml`（五笔） | 同上（Windows 端） | **删除**，统一用雾凇 `weasel.yaml` |

> ⚠️ 部署前请把 `~/Library/Rime/` 下旧的 `default.custom.yaml`、`squirrel.custom.yaml`、`weasel.custom.yaml` 一并删除，否则仍会被覆盖。

## 三、保留的功能

### 极点五笔
- `wubi86_jidian`（极点五笔86）：四码上屏、四码唯一自动上屏、字符集过滤、词组扩展库。
- 简入繁出：由「简体→繁体」开关控制（`wubi86_jidian_trad.schema.yaml` 文件保留但不入列表）。
- `numbers`（大写数字）：按 `1234567890` 出「壹贰叁…」，Shift+数字出「一二三…」，
  `s`=拾 `b`=佰 `q`=仟 `w`=万 `y`=元 `j`=角 `f`=分；二三候选沿用 `;` `'`。
- **z 键拼音反查显示五笔编码**：`reverse_lookup` 挂 `pinyin_simp` 码表，
  `recognizer/patterns/reverse_lookup: "^z[a-z]*'?$"`，候选后以 comment 形式显示五笔码
  （如 `znihao` → `你好 wqvb`）。依赖 `pinyin_simp.schema.yaml` + `pinyin_simp.dict.yaml`，
  该方案不进方案列表，仅作反查码表使用。
- 二、三候选键 `;` `'`（五笔手感，保留在方案自身的 key_binder 内，不影响拼音方案）。

### 雾凇拼音
- `rime_ice`（全拼）与 `double_pinyin_flypy`（小鹤双拼）。
- **小鹤双拼支持全拼混输**：`speller/algebra` 里原本的 `xform`（把全拼改写成双拼，全拼随之失效）
  全部改为 `derive`，保留全拼拼写的同时增加双拼拼写，**两种输入方式都能用**。
  配合下面的编码提示，打全拼时候选后面就显示对应的小鹤码，过渡期查码不用翻键位表。
  代价：拼写集合变大（每音节 1~4 种），部署稍慢，混输时分词歧义和重码会增加。
  想退回纯双拼，把这些 `derive/` 改回 `xform/` 即可（`xlit` 和 `erase` 那两行不动）。
- **小鹤双拼编码提示（默认全程显示）**：每个候选项后面直接显示该字/词的小鹤双拼编码，
  如 `你好 nihc`、`中国 vsgo`、`我们都是中国人 womfdzuivsgorf`，无需任何前缀。
  实现：`translator/spelling_hints: 12` 生成全拼注音，再由 `translator/comment_format`
  的 28 条规则按音节边界换算成小鹤键位，最后去掉音节间分隔。
  因小鹤每个音节固定两键，拼接后的编码与实际按键完全一致（西安 = `xiaj`，天安门 = `tmajmf`）。
  ⚠️ 代价：雾凇原有的**错音错字提示**（corrector.lua）与编码提示共用 comment 位置，
  已在 `engine/filters` 中注释掉，二者只能选其一。
  想给编码加括号显示成 `［nihc］`，取消 `comment_format` 末尾两行注释即可。
- 方案选单第 2 行的开关项（中/￥/简/😄/半/词）完整保留：
  `ascii_mode` `ascii_punct` `traditionalization` `emoji` `full_shape` `search_single_char`。
- 英文混输（melt_eng）、部件拆字辅码（radical_pinyin）、v 模式符号、emoji、
  日期时间农历、数字金额大写、计算器、Unicode、长词优先、错音错字提示等全部 lua 功能。

## 四、相对原包的改动

1. `default.yaml`：`schema_list` 改为上述 4 项；`config_version` 改为 `2026-02-06-merged`。
2. `double_pinyin_flypy.schema.yaml`：`translator/comment_format` 改为全拼→小鹤键位换算规则，
   `spelling_hints` 由 8 提至 12，并注释掉 `lua_filter@*corrector`（见上）。
3. `wubi86_jidian.schema.yaml` / `wubi86_jidian_trad.schema.yaml`：
   `key_binder` 改为 `import_preset: default` + 自有 bindings（`;` `'` 二三候选、
   `- =` / `[ ]` / `Tab` 翻页、`Ctrl+Shift+3` 中英标点、`Ctrl+Shift+4` 简繁），
   与雾凇快捷键位一致；反查提示改为 `〔拼音〕`。
4. `wubi86_jidian.schema.yaml`：`dependencies` 启用 `pinyin_simp`，保证反查码表被构建。
5. `default.yaml`：加入 `numbers` 方案；`switcher/save_options` 增加 `zh_trad`、去掉 `full_shape`
   （`numbers` 把 `full_shape` 复用为「壹贰叁/一二三」，全局记忆会把其他方案带成全角）。
6. `wubi86_jidian.schema.yaml`：`zh_trad` 去掉 `reset`，使简繁状态可被记忆。
7. `pinyin_simp.schema.yaml`：去掉缺失的 `stroke` 笔画反查依赖，去掉多余的
   `reverse_lookup_translator` 与 `/fh` 符号 patterns，仅保留作为反查码表的能力。
8. `wubi86_jidian.dict.yaml`：删去 `wubi86_jidian_user_hamster` 的 import（Hamster 是 iOS 仓输入法的空词库，Mac/Win 用不到，文件已删）。
9. `double_pinyin_flypy.schema.yaml`：`speller/algebra` 的 `xform` 全部改为 `derive`，支持全拼混输。
   （`rime_ice.schema.yaml` 保持上游原样，错音错字提示 corrector.lua 未动。）
10. `squirrel.yaml`：注释颜色改为极点五笔皮肤的绿色 `0x5AC461`；`app_options` 增加
   `com.apple.ScreenContinuity`（iPhone 镜像）设 `ascii_mode: true`，规避 rime/squirrel#1137
   ——该窗口下鼠须管收到的 keyDown 不带 Command 标志，会吞掉 ⌘V/⌘A/⌘C 等快捷键。
11. 新增空的 `custom_phrase_double.txt`（小鹤双拼自定义短语文件，原包需手动创建）。

## 五、已删除（未纳入整合）的内容

- 雾凇：`t9`、`double_pinyin`(自然码)、`_abc`、`_mspy`、`_sogou`、`_ziguang`、`_jiajia` 方案，
  `others/`（文档与示例）、`build/`、`recipe.yaml`、`README.md`、`AGENTS.md`、`LICENSE`、`.github/`，
  以及未使用的 `en_dicts/cn_en_*.txt`（仅保留全拼与小鹤两份）。
- 五笔：`wubi86_jidian_pinyin`、`wubi86_jidian_trad_pinyin`（五笔拼音混输，免 z 前缀 + 自动造词，
  属于引擎级差异，开关切不了，只能作为独立方案，本次未纳入）、`pinyin_simp` 入方案列表的用法，
  `default.custom.yaml`、`squirrel.custom.yaml`、`weasel.custom.yaml`、
  `imgs/`、`plum/`、`仓键盘布局/`、`README.md`、`LICENSE`、`.github/`。

## 六、可选调整

- 恢复雾凇的**错音错字提示**：取消 `double_pinyin_flypy.schema.yaml` 中
  `lua_filter@*corrector` 的注释，并把 `translator/comment_format` 换回原来的 `［］` 两行。
- 五笔**输入时显示编码提示**：删除 `wubi86_jidian*.schema.yaml` 中
  `translator/comment_format` 下的 `- xform/.+//` 一行即可。
- 五笔简繁目标字形：`tradition/opencc_config` 现为 `s2hk.json`，可改 `s2t.json` / `s2tw.json`。
- 只用小鹤双拼时，`melt_eng.schema.yaml` 与 `radical_pinyin.schema.yaml` 的
  `speller/algebra` 需把 `__include: algebra_rime_ice` 换成 `algebra_double_pinyin_flypy`。
  （当前保留全拼写法，因为全拼是首选方案。）

## 七、部署

1. 备份并清空 `~/Library/Rime/`（保留 `*.userdb/`、`installation.yaml`、`user.yaml` 可保留词频）。
2. 把本文件夹内**所有内容**（不含本说明）复制进 `~/Library/Rime/`。
3. 鼠须管 →「重新部署」。
