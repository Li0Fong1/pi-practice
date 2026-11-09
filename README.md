# 实践课1：Pi Agent 安装与基础使用

> 环境：Windows 11 · Node.js v22.22.2 · npm 10.9.7 · Git 2.55.0 · Pi CLI v1.5.0
> 模型：DeepSeek（deepseek-v4-pro / deepseek-flash）

## 一、环境与安装

| 项目 | 结果 |
|---|---|
| 安装方式 | 官方 npm 包 `@earendil-works/pi-coding-agent` 全局安装 |
| 安装命令 | `npm install -g --prefix "%APPDATA%\npm" @earendil-works/pi-coding-agent` |
| 版本验证 | `pi --version` → `1.5.0` |
| 可执行文件 | `C:\Users\Lenovo\AppData\Roaming\npm\pi` |
| 配置目录 | `C:\Users\Lenovo\.pi\agent` |

安装后首次未配置模型时，`pi --list-models` 输出：

```
No models available. Use /login to log into a provider via OAuth or API key.
```

说明客户端已启动，仅缺模型凭证。

练习目录：`pi-test/`（内含 `hello.py`、`seven.png`）。

## 二、配置模型并完成会话

采用 API Key 方式（等价于 `/login` → Sign in with an API key → 选 deepseek），凭证写入 `C:\Users\Lenovo\.pi\agent\auth.json`（API Key 已脱敏，不入库）：

```json
{ "deepseek": { "type": "api_key", "key": "<your-deepseek-key>" } }
```

验证：`pi auth check --provider deepseek` → `ready`；`pi --list-models` 输出：

```
provider  model            context  max-out  thinking  images
deepseek  deepseek-flash   1M       384K     yes       yes
deepseek  deepseek-v4-pro  1M       384K     yes       no
```

会话测试（提示词：`你好，请用一句话自我介绍`）：

> 你好！我是一个智能编程助手，可以帮你阅读代码、运行命令、修改文件，解决各种开发问题。

## 三、切换模型与思考级别

| 阶段 | 模型 | 思考级别 | 一条短问题的回复 |
|---|---|---|---|
| 切换前（默认） | deepseek-v4-pro | medium（默认） | 持续集成（CI）是一种开发实践，要求开发者频繁地把代码合并到共享主干…… |
| 切换到 flash | deepseek-flash | off | 持续集成（CI）是一种开发实践，要求开发者频繁地把代码合并到共享主干，并通过自动化构建和测试来尽早发现集成问题。 |
| 再调思考级别 | deepseek-flash | high | 持续集成（CI）是指开发者频繁地把代码改动合并到主干分支，每次合并都自动触发构建和测试，从而尽早发现并修复集成问题。 |

对应命令：

```bash
pi -p --model deepseek-flash --thinking off  "请用一句话解释持续集成"
pi -p --model deepseek-flash --thinking high "请用一句话解释持续集成"
```

选择依据：日常改写、格式整理用 off/low，延迟低；涉及多文件推理、方案设计时用 high，牺牲等待时间换推理深度。两个模型均支持 thinking，切换模型与调级别互不等价。

## 四、引用代码与粘贴图片

### A. `@` 引用代码文件

`pi-test/hello.py`：

```python
def add(a, b):
    return a + b

print(add(2, 3))
```

提示词：`请解释 @hello.py 的功能，说明运行后输出什么。只解释，不修改文件。`

回复要点：`add(a, b)` 返回两数之和；`print(add(2, 3))` 输出 `5`；文件未被修改。

### B. 小图片识别

素材 `pi-test/seven.png`：200×200，白底 + 深蓝色（约 #0B5FC0）正方形 + 白色数字 7。

提示词：`请描述图片中的颜色、形状和数字。`（模型：deepseek-flash）

回复要点：深蓝色正方形色块，边缘直角无圆角，中央白色无衬线粗体数字 `7`。

## 五、安装并使用插件

| 项目 | 内容 |
|---|---|
| 插件 | `pi-web-access`（npm 包名 `pi-web-access`） |
| 来源 | https://pi.dev/packages |
| 安装命令 | `pi install npm:pi-web-access` |
| 安装位置 | `C:\Users\Lenovo\.pi\agent\npm\node_modules\pi-web-access`（个人级，211 个依赖包） |
| 安装输出 | `Installed npm:pi-web-access` |

重启 Pi 后 `pi list` 显示：

```
User packages:
  npm:pi-web-access
    C:\Users\Lenovo\.pi\agent\npm\node_modules\pi-web-access
```

加载成功后，系统工具列表新增 `web_search`、`source_check`、`fetch_content`、`get_search_content` 四个工具。

最小功能验证：`请获取 https://example.com 的页面标题，只回答标题。`

- 实际调用：`fetch_content({"url":"https://example.com","mode":"raw"})`
- 返回：`Example Domain`

调用记录见 `evidence/pi-web-access-toolcall.jsonl`。

## 六、安装 PPT Skill（选做）

| 项目 | 内容 |
|---|---|
| 来源 | https://github.com/anthropics/skills → `skills/pptx` |
| 源 commit | `2c5958ecac7cee2b583967a2c4981d0f4a75c579` |
| 安装路径 | `D:\projects\study\.pi\skills\pptx` |
| 文件数 | 56 |
| 关键文件 | `SKILL.md`、`LICENSE.txt`、`scripts/thumbnail.py`、`scripts/add_slide.py`、`scripts/clean.py`、`scripts/office/...` |

目录结构：

```
D:\projects\study\
  .pi\
    skills\
      pptx\
        SKILL.md
        LICENSE.txt
        scripts\
          thumbnail.py
          add_slide.py
          clean.py
          office\ ...
  pi-demo.pptx
```

发现与调用验证：在 `D:\projects\study` 执行 `pi` 后输入

```
/skill:pptx 请说明这个 Skill 的名称、用途和所在目录
```

回复确认名称为 `pptx`，用途为创建/编辑/读取 pptx，目录为 `D:\projects\study\.pi\skills\pptx\`。

生成验证（选做）：`pi-demo.pptx`，16:9，2 页；python-pptx 重新打开正常，页数 2，30.8 KB。
第 1 页「Pi Agent 实践课 / 安装与基础使用」，第 2 页「本次完成的任务」含三条要点。

## 七、四层验收对照

| 层次 | 状态 | 依据 |
|---|---|---|
| 安装成功 | ✅ | `pi --version` = 1.5.0；`pi list` 可见插件 |
| 发现成功 | ✅ | 重启后工具列表出现插件 4 个工具；`/skill:pptx` 可用 |
| 调用成功 | ✅ | `fetch_content` 实际抓取 example.com；pptx Skill 正确报出路径 |
| 生成产物成功 | ✅ | `pi-demo.pptx` 2 页可正常打开 |

## 八、提交说明

- 截图 S1–S8 见 `screenshots/`，拍摄清单见 `screenshots/截图清单.md`。
- 仓库不含任何密钥：`auth.json`、`out_*.json` 中的凭证均已排除。
- 说明：S1–S8 为本环境**按真实运行数据渲染**的终端/资源管理器样式图片（版本号、模型回复、`pi list` 插件列表、目录树等均取自实跑结果），非对交互式 TUI 的逐帧屏幕抓取——后者需在本人终端手动完成。如需完全一致的原始截图，按 `截图清单.md` 命令自行补截即可。
