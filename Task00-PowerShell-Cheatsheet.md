# Task00 环境配置笔记：PowerShell 与 Python 环境速查

> 学习时间：2026-11-16 ~ 2026-11-17
> 记录原则：只记我真正敲过、真正踩过的，不抄文档。

---

## 零、先分清三个「黑框」

新手最容易晕的地方：**同样是黑底窗口，但你在里面能敲的东西完全不同。**

| 你看到的提示符 | 你在哪 | 能敲什么 | 例子 |
|---|---|---|---|
| `PS C:\Users\李雨洁>` | **PowerShell**（终端） | 系统命令、程序名 | `python --version`、`cd`、`pip install` |
| `C:\Users\李雨洁>` | **cmd**（老命令行） | 同上，语法略不同 | `dir`、`venv\Scripts\activate.bat` |
| `>>>` | **Python REPL** | **只能敲 Python 代码** | `print(1+1)`、`exit()` |

**判断规则**：只有 `PS ...>` 或 `C:\...>` 开头的地方，才能敲系统命令。

在 `>>>` 里敲 `python --version` 会得到 `SyntaxError`，因为 REPL 读的是 **Python 代码**，不是命令。

---

## 一、终端到底在干什么：查表器，不是聊天 AI

```
你敲: python --version
        ↓
① PowerShell 按空格把这一行拆成 ["python", "--version"]
② 拿第一个词 "python" 去 PATH 这张表里查有没有同名程序
③ 找到 → 启动它，把剩下的 "--version" 原样递过去
④ python.exe 自己认识 --version，自己打印版本号，然后退出
```

**关键认知**：`Python 3.14.5` 这行字**不是 PowerShell 说的，是 python.exe 自己说的**。PowerShell 全程不知道 Python 装在哪、是什么版本。

所以：

- `python` 能用 → 因为 PATH 里真有 `python.exe`
- `hello world` 不能用 → 因为表里没有叫 `hello` 的程序

> **终端不懂英语，一个词都不懂。它只认「已安装的程序名」这张表。**

验证命令：

```powershell
where.exe python      # 看 python 到底解析到哪个文件
py -0p                # 列出 py 启动器注册的所有 Python 版本
```

---

## 二、导航：搞清楚「我在哪」

| 命令 | 作用 | 说明 |
|---|---|---|
| `pwd` | 打印当前路径 | 终端的 GPS |
| `cd 路径` | 换目录 | Change Directory |
| `ls` / `dir` | 列出当前目录内容 | PowerShell 里两者等价（别名） |
| `ls -Force` | 列出**包括隐藏项** | 带点的文件（如 `.env`、`.venv`）默认看不到 |
| `Test-Path 路径` | 判断路径存不存在 | 返回 `True` / `False` |

### 路径符号

| 写法 | 含义 |
|---|---|
| `C:\Users\李雨洁\xxx` | **绝对路径**，任何位置都指同一个地方 |
| `hello-agents` | **相对路径**，从当前位置往下找 |
| `..` | 上一级目录（`cd ..` 后退一层） |
| `~` | 用户主目录（`C:\Users\李雨洁`） |

**为什么必须 `cd` 进去才能跑脚本**：因为 `python FirstAgentTest.py` 里的文件名是**相对路径**，Python 只在**当前文件夹**里找它。

### `cd` 的一个常见误解

`mkdir D:\study` 只是「**建了一个文件夹**」，**不等于走进去**。提示符没变 = 你还在原地。想进去还得 `cd`。

---

## 三、建文件 / 建目录 / 搬东西

| 目的 | PowerShell | Linux/macOS | 备注 |
|---|---|---|---|
| 建目录 | `mkdir D:\study` | `mkdir` | 有回显，会打印结果 |
| 建空文件 | **`New-Item .env`** | `touch .env` | **Windows 没有 `touch`**（实测确认） |
| 移动/改名 | `Move-Item "源" "目标"` | `mv` | **无输出 = 成功** |
| 删除文件 | `Remove-Item 文件名` | `rm` | — |
| 打开编辑 | `notepad .env` | — | 记事本，零依赖 |

### ⚠️ 建 `.env` 时的坑

**不要用「右键 → 新建 → 文本文档」改名**。Windows 默认隐藏已知扩展名，改完实际叫 `.env.txt`，Python 读不到。

用命令建最稳：

```powershell
New-Item .env
Get-ChildItem -Force -Name     # 确认名字真的是 .env，不是 .env.txt
```

### ⚠️ 记事本保存的坑

- **千万别用「文件 → 另存为」**，默认会存成 `.txt`，`.env` 变 `.env.txt`，前功尽弃
- 直接 `Ctrl + S`
- 点 ✕ 关窗时弹「是否保存」→ **必须点「保存」**，点「不保存」= 白干

---

## 四、Python 环境：venv 是怎么回事

### 建环境

```powershell
python -m venv venv
```

**两个 `venv` 不是一回事**：

```
python -m venv venv
       ↑        ↑
    模块名    文件夹名（随便取）
```

- 第一个 `venv`：Python **自带的模块**（虚拟环境工具），固定不能改
- 第二个 `venv`：**要创建的文件夹叫什么**，可以叫 `my_env`、`.venv` 都行

**现象**：静默 10~30 秒，屏幕什么都不出。**这是正常的**，Python 正在拷一份解释器和标准库，别按 `Ctrl+C`。

### 激活

| 你在哪个终端 | 命令 |
|---|---|
| **PowerShell** | `.\venv\Scripts\Activate.ps1` |
| cmd | `venv\Scripts\activate.bat` |
| Git Bash | `source venv/Scripts/activate` |

**实测结论**：在 PowerShell 里，文档写的 `venv\Scripts\activate` **其实也能跑**（PowerShell 会自动解析成 `Activate.ps1`）。但建议显式写 `Activate.ps1`，因为 `Scripts` 目录下躺着**四个同名文件**：

```
activate         ← Git Bash / Linux 用
activate.bat     ← cmd 用
Activate.ps1     ← PowerShell 用  ← 你真正要的
activate.fish    ← fish shell 用
```

**排错时这个区别很要命**——不写扩展名，你不知道它挑的是哪个。

### 激活到底做了什么（三件事）

```
激活前：python → D:\...\Python314\python.exe       VIRTUAL_ENV → (空)
激活后：python → D:\study\hello-agents\venv\Scripts\python.exe
        VIRTUAL_ENV → D:\study\hello-agents\venv
        PATH 第一项 → D:\study\hello-agents\venv\Scripts
```

1. 给 PATH **最前面**插一条 `...\venv\Scripts`
2. 设环境变量 `VIRTUAL_ENV`
3. 改提示符显示逻辑 → 于是你看到 `(venv)` 前缀

**它不安装任何东西、不改系统设置**，只在当前窗口的内存里改了两个变量。所以：

- 每次开新窗口都要**重新激活**
- 关掉窗口 = 自动退出，不用手动还原
- 两个窗口可以一个激活、一个不激活，互不影响

**成功判据只有一个**：

```
(venv) PS D:\study\hello-agents>
  ↑
这个前缀出现 = 成功
```

---

## 五、装依赖：pip 的四个坑

### 标准写法

```powershell
python -m pip install "requests>=2.31.0" -i https://mirrors.aliyun.com/pypi/simple
```

### 坑 1：必须写 `python -m pip`，不能写裸 `pip`

| 写法 | 含义 | 结果 |
|---|---|---|
| `pip install` | 去 PATH 里找一个叫 `pip.exe` 的程序 | ⚠️ **可能装错环境** |
| `python -m pip install` | **先用这个 python，再调它自己的 pip** | ✅ 不可能装错 |

我用 `python -m pip` 还有一个**这台机器特有的理由**：项目目录搬家后，`venv\Scripts\pip.exe` **失效了**（详见第九节）。

实测对比：

```
venv\Scripts\pip.exe -V          → ❌ 无任何输出（静默失败）
venv\Scripts\python.exe -m pip -V → ✅ pip 26.1.1
```

### 坑 2：包名必须加引号

不加引号时，PowerShell 会把 `>` 当成**重定向符**。实测过这个实验：

```powershell
python -c "print('模拟安装')" >=2.31.0
```

跑完目录里多出一个文件：

```
Name       Length
=2.31.0       36      ← 那句 print 的输出被写进了这个文件
```

套到 `pip install requests>=2.31.0` 上：

| 你以为 | 实际 |
|---|---|
| 安装 ≥2.31.0 版本的 requests | 安装**任意最新版**（版本约束被吃掉） |
| 屏幕上有下载进度 | **屏幕上什么都没有**，全写进了 `=2.31.0` 文件 |

**最阴的地方：它不报错。**

### 坑 3：镜像源要按机器实际情况选

| 源 | 地址 | 我这台机器的实测 |
|---|---|---|
| 清华 | `pypi.tuna.tsinghua.edu.cn` | ❌ **HTTP 403**（被本地代理拦） |
| 阿里云 | `mirrors.aliyun.com/pypi/simple` | ✅ 可用 |
| 官方 | `pypi.org` | ✅ 可用但慢 |

`-i` 是 `--index-url` 的缩写，意思是「去哪下载」。不写就用官方源（服务器在美国，慢且易断）。

### 坑 4：四条分开敲 vs 合成一条

功能**完全等价**。合成一条的好处只是少启动几次 pip。**第一次配环境建议分开敲**，能看清每个包装没装上。

```powershell
python -m pip install "requests>=2.31.0" -i https://mirrors.aliyun.com/pypi/simple
python -m pip install "tavily-python>=0.3.0" -i https://mirrors.aliyun.com/pypi/simple
python -m pip install "openai>=1.0.0" -i https://mirrors.aliyun.com/pypi/simple
python -m pip install "python-dotenv>=1.0.0" -i https://mirrors.aliyun.com/pypi/simple
```

---

## 六、运行与验证

### 跑脚本

```powershell
python FirstAgentTest.py
```

`python` 后面跟什么，决定它干什么：

| 命令 | Python 收到 | 行为 |
|---|---|---|
| `python` | 无参数 | 启动**交互式解释器**，出 `>>>` 等你输入 |
| `python --version` | `--version` | 打印版本号，**立刻退出** |
| `python FirstAgentTest.py` | 文件名 | **打开文件，从头执行到底**，执行完退出 |
| `python -c "代码"` | `-c` + 代码 | 执行这一小段代码，不存文件 |

### 跑完保留记录（重要）

终端窗口一关，屏幕内容**就永远没了**——Windows 不把控制台内容写到磁盘。

三种留证据的办法：

```powershell
# ① 输出同时写进文件（推荐，屏幕上照常显示）
python FirstAgentTest.py *> run.log

# ② 复制文字：在终端里选中要复制的行 → 右键就复制了

# ③ 录整个会话（要严谨复盘时用）
Start-Transcript -Path .\session.log
# ... 干活 ...
Stop-Transcript
```

### 环境体检（装完依赖后必做）

```powershell
python -c "import openai, requests, tavily, dotenv; print('环境正常')"
```

**为什么 `pip list` 说装了，还要这一步**：`pip list` 读的是 pip 的**安装登记表**，`import` 才是**真的去加载**。这两件事会不一致（装到一半中断、依赖的 DLL 缺失、版本冲突）。

**登记 ≠ 可用，这条命令验的是可用。** 它还顺带验证了「激活成功」——如果没激活，用的就是系统 Python，一个包都没有，立刻报 `ModuleNotFoundError`。

---

## 七、环境变量与 .env

### 为什么用 `.env` 而不是把 Key 写进代码

| | 写死在代码里 | 用 `.env` |
|---|---|---|
| 改配置 | 要改代码 | 只改 `.env` |
| 分享代码 | **Key 一起泄露** | 安全，`.env` 不进仓库 |
| 换供应商 | 回去改代码 | **改三行，代码零改动** |

这条我亲身验证过：AIHubmix 额度用完后换 ModelScope，**代码一行没改**，只改了 `.env` 三行。

### `.env` 格式的五条死规矩

| 规矩 | 正确 | 错误 |
|---|---|---|
| 等号两边**不能有空格** | `MODEL_NAME=abc` | `MODEL_NAME = abc` |
| 值**不加引号** | `API_KEY=sk-xx` | `API_KEY="sk-xx"` |
| 用**英文**标点 | `=` | `＝`（全角） |
| 一行一个 | 四行分开 | `A=1;B=2` |
| BASE_URL 尾部干净 | `https://aihubmix.com/v1` | `.../v1/` 或 `.../v1/chat/completions` |

### 代码里怎么读

```python
import os
from dotenv import load_dotenv
load_dotenv()          # 把 .env 每一行塞进环境变量

API_KEY  = os.getenv("OPENAI_API_KEY")
BASE_URL = os.getenv("OPENAI_BASE_URL")
MODEL_ID = os.getenv("MODEL_NAME")
```

### ⚠️ 我踩过的致命错误：`os.getenv()` 括号里填的是「标签名」，不是「值」

```python
os.getenv("sk-4fmCeD6WyGzw...")          # ❌ 把真 Key 当成了变量名
os.getenv("https://aihubmix.com/v1")      # ❌ 网址当成了变量名
os.getenv("OPENAI_API_KEY")               # ✅ 这才是对的
```

**比喻**：`os.getenv("X")` 是「去抽屉里找**贴着 X 标签**的那一格，把里面的东西拿出来」。括号里写的是**标签**，不是东西本身。

`.env` 里 `OPENAI_API_KEY=sk-xxx`：

- 左半边 = **标签名**
- 右半边 = **值**

代码里只写标签名。

### 手动验证 .env 能不能被读到

```powershell
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('URL:', os.getenv('OPENAI_BASE_URL')); print('MODEL:', os.getenv('MODEL_NAME'))"
```

三项全是 `None` → `.env` 没被找到（位置或文件名问题）。

---

## 八、报错对照表

### 输出级别

| 看到什么 | 含义 | 动作 |
|---|---|---|
| `WARNING` 黄字 | 提醒，程序自己绕过去了 | **可忽略** |
| `ERROR` 红字 | 真错了 | 停下 |
| `Traceback` + `File ...` | 程序崩了 | 停下 |

例：`WARNING: Cache entry deserialization failed, entry ignored` = pip 本地缓存坏了，跳过重下。**不是故障。**

### 常见错误 → 根因

| 报错 | 根因 | 怎么改 |
|---|---|---|
| `无法将"xxx"项识别为 cmdlet、函数…` | PATH 表里没有这个程序 | 检查拼写 / 是否装了 |
| `ModuleNotFoundError: No module named 'xxx'` | 包装到别的 Python 里了，或没激活 venv | 确认提示符有 `(venv)` |
| `SyntaxError: invalid syntax` | 在 REPL 里敲了系统命令，或代码本身语法错 | 看是不是在 `>>>` 里敲的 |
| `can't open file 'xxx.py': [Errno 2]` | 当前目录里没有这个文件 | `cd` 到脚本所在目录 |
| `401` / `Incorrect API key` | Key 错 / 没复制全 / 已失效 | 重新复制 Key |
| `404` / `model_not_found` | 模型 ID 错或已退役 | 去平台确认真实 ID |
| `Connection error` / 超时 | 网络到不了服务端 | 换服务商 / 检查代理 |
| `禁止运行脚本` | PowerShell 执行策略 | `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` |

### 一个反直觉的现象：`pip.exe` 静默失败

`venv\Scripts\pip.exe -V` **什么都不输出，也不报错**，直接退出。这比报错更阴险——你会以为装上了，其实啥也没发生。

原因见下一节。

---

## 九、这台机器的特殊坑（别人踩不到，我记下来）

### 坑 1：本地代理会拦国内镜像

环境里有代理 `127.0.0.1:31565`，它把清华 PyPI 镜像拦成 **403**。

**解决**：一律用阿里云源。注意 —— **清环境变量没用**，因为 Python 会读 Windows 注册表里的系统代理设置。

### 坑 2：venv 搬家后 `pip.exe` 坏掉

我把项目从 C 盘搬到 `D:\study\hello-agents`，`pip.exe` 就失效了。

**原因**：`pip.exe` 是个「包装壳」，内部**硬编码了创建时的绝对路径**。那个路径搬走后不存在了 → 它启动失败，**还不报错**。

而 `python.exe` 活着，因为它靠 `pyvenv.cfg` 里的 `home` 找底座（指向 `D:\...\Python314`，还在），找自己的库用 exe 的相对位置算，**跟着搬家自动适应**。

**结论**：这个环境里装包一律用 `python -m pip`，永不失效。这个习惯本身也更稳。

### 坑 3：有四个 Python，别搞混

| 位置 | 版本 | 性质 |
|---|---|---|
| `D:\Users\...\Programs\Python\Python314\` | **3.14.5** | 官方安装包正式安装，带 pip ✅ |
| `C:\Users\...\AppData\Local\Python\pythoncore-3.14-64\` | 3.14.4 | python.org 新安装器的另一份，重复了 |
| `C:\Users\...\AppData\Roaming\uv\python\cpython-3.12.15-…` | 3.12.15 | 被 `uv` 管理的，独立隔离 |
| `C:\Users\...\.workbuddy\binaries\python\versions\3.13.12\` | 3.13.14 | **WorkBuddy 自带便携版** |

**现象**：Start 菜单打开的解释器显示 3.14.4，而 `python --version` 显示 3.14.5。**两个都对，不矛盾**——前者指向 C 盘那份，后者是 D 盘那份排在 PATH 前面。

### 坑 4：venv 里写死了创建时的路径

`venv\pyvenv.cfg` 里有一行：

```
command = ... -m venv C:\Users\李雨洁\WorkBuddy\2026-11-16-18-53-47\hello-agents\venv
```

**所以：一个建好的 venv 搬完家后，部分程序会失效。搬家的正确顺序是先搬目录、再重建 venv。**

---

## 十、完整流程串一遍（从零到跑通）

```powershell
# 1. 进项目目录
cd D:\study\hello-agents

# 2. 激活虚拟环境（看到 (venv) 才算成功）
.\venv\Scripts\Activate.ps1

# 3. 环境体检
python -c "import openai, requests, tavily, dotenv; print('环境正常')"

# 4. 进代码目录
cd code\chapter1

# 5. 跑脚本
python FirstAgentTest.py
```

新建环境的话，中间加两步：

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install "requests>=2.31.0" "tavily-python>=0.3.0" "openai>=1.0.0" "python-dotenv>=1.0.0" -i https://mirrors.aliyun.com/pypi/simple
```

---

## 十一、我真正学到的三件事（不是命令，是思维方式）

### 1. 「没消息就是好消息」

`Move-Item`、`python -m venv`、`pip install`（成功时最后一行除外）都**不输出任何东西**。判断成功的标准永远只有一个：**有没有红字报错**。

### 2. 别人嘴里的「必须这样」，都该自己验一遍

我原以为文档写的 `venv\Scripts\activate` 在 PowerShell 里会报错，实测发现**它能跑**。

从这件事学到：**包括 AI 在内，任何人给你的「必须」，都值得用一条命令验证成本。**

### 3. 缩小故障域

不验环境就直接跑 200 行代码，报错会混着业务逻辑，你会怀疑是自己写错了。

先体检（`import` 一下），出问题就只可能是一个原因：**环境**。排查范围从「不知道哪错了」缩到「就是环境」。

> 就像买冰箱不会直接塞满食材等两天，而是先通电看灯亮不亮、听压缩机响不响。**这就是通电试机。**

---

## 十二、键位速查

| 快捷键 | 作用 |
|---|---|
| `Ctrl + C` | 中断正在运行的程序（别用它关窗口） |
| `Ctrl + S` | 保存（记事本里必按） |
| `Ctrl + Shift + N` | 浏览器无痕窗口（验证链接是否真公开） |
| `Win + X` | 打开终端菜单 |
| `Win + V` | 剪贴板历史 |
| `Tab` | 路径自动补全（中文路径神器） |
| `↑` / `↓` | 翻历史命令 |

---

## 参考

- 环境配置原文档：`Extra-Chapter/Extra07-环境配置.md`
- 项目代码：`code/chapter1/FirstAgentTest.py`
