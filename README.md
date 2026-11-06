# fcitx5-wetypex (personal fork)

基于 [panxuc/fcitx5-wetypex](https://github.com/panxuc/fcitx5-wetypex) 的个人补丁分支，新增两个功能：

- **中英文之间自动加空格**：输入「我在Linux」上屏为「我在 Linux」
- **切换中英文时显示提示**：按 Shift 切换后短暂显示「中文」或「English」

除这两项外与原版完全一致。原版的微信输入法引擎、云候选、语音、跨设备同步、问 AI 等功能全部保留，配置方式不变。

原版完整文档见 [README.upstream.md](README.upstream.md)。

## 安装

原版未提供预编译包，从源码编译。依赖见下方。

```bash
git clone https://github.com/sijin-xb/fcitx5-wetypex.git
cd fcitx5-wetypex
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DWETYPE_BUNDLED_CURL=OFF
cmake --build build --parallel $(nproc)
sudo cmake --install build
```

`-DWETYPE_BUNDLED_CURL=OFF` 使用系统 libcurl，跳过内置 curl 的静态编译，配置从十几分钟降到几秒。

运行时需要官方微信输入法安装包，见上游的[首次设置](https://github.com/panxuc/fcitx5-wetypex#首次设置)。

### 依赖

```bash
sudo pacman -S --needed cmake clang boost libc++ libime python-fonttools \
  fcitx5 fcitx5-qt qt6-base qt6-svg qt6-webengine layer-shell-qt \
  bubblewrap pipewire ffmpeg libnotify polkit wl-clipboard \
  json-c curl openssl wayland fontconfig python-pillow
```

`boost` 不在原版 PKGBUILD 的依赖里，但 `LibIMEPinyin` 的 CMake config 会 `find_dependency(Boost)`，缺少时配置阶段报错。

只改设置窗口时可单独编译，几秒完成：

```bash
cmake --build build --target fcitx5-wetypex-settings
```

## 新功能

两项开关都在设置窗口的**「输入」**页，位于「模糊拼音」下方。

打开设置窗口：

```bash
fcitx5-wetypex-settings
```

### 中英文自动空格

上屏文本时，在汉字与 ASCII 字母/数字的交界处插入空格：

| 输入 | 输出 |
| --- | --- |
| 我在Linux | 我在 Linux |
| API接口 | API 接口 |
| 我承认MacBook是一个好用的品牌 | 我承认 MacBook 是一个好用的品牌 |

数字后接量词时不加，避免误伤：

| 输入 | 输出 |
| --- | --- |
| 100元 | 100元 |
| 第3章 | 第3章 |
| 5个 | 5个 |

量词表包含 40 个常见字（个/元/章/年/月/日/点/次/位/米/克/级/版/期…）。

判定基于 UTF-8 解码后的码点，CJK 范围包括基本区、扩展 A/B/C/D 和兼容区。已有空格、标点、emoji 不触发。

跨 commit 的末字符记忆保存在每个输入上下文独立的状态里，切换窗口不会互相污染。

### 中英文切换提示

按 Shift 切换后，候选框位置显示「中文」或「English」，700ms 后自动消失。

提示是一个不可选的候选词，点击不触发任何动作。提示期间开始打字会被真实候选立即覆盖，不会卡住输入。

## 配置

两项功能在 `~/.config/fcitx5/wetypex.json` 中对应 `auto_space` 和 `language_hint`，默认均为 `true`。

也可以直接改文件，切换窗口后生效（`reloadSettings()` 在每次激活输入法时调用）：

```json
{
    "auto_space": true,
    "language_hint": true
}
```

这个项目的配置有**两套**：`wetypex.json` 是权威来源，`~/.config/fcitx5/conf/wetypex.conf` 是它的镜像，每次加载会被覆盖。改配置只改 json。

## 已知限制

- 自动空格只处理 ASCII 字母/数字与汉字的交界，不处理全角符号、括号、百分号等情况
- 量词表是静态枚举，未收录的词（如「3匹马」的「匹」）仍会加空格
- 切换提示依赖候选框渲染，在禁用候选框的极简配置下不可见

## License

MIT，与上游一致。原版微信输入法核心的使用条款由腾讯提供，详见上游说明。
