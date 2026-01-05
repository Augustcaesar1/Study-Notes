# Python 包管理笔记：从 Pip 到 uv

## 一、 传统流派：Pip + Venv

### 1. 核心问题：环境隔离

在默认情况下，`pip install` 会将库安装到系统的全局环境中。

- **后果：**
  - **版本冲突：** 项目 A 需要 Django 2.0，项目 B 需要 Django 3.0，全局环境无法同时满足。
  - **依赖混乱：** 难以分清哪些库是哪个项目需要的。

### 2. 解决方案：虚拟环境 (Virtual Environment)

为每个项目创建一个独立的“沙盒”。

- **创建命令：**

  Bash

  ```
  python -m venv .venv
  ```

  > **注：** 推荐使用 `.venv` 作为名字，PyCharm 和 VSCode 等主流 IDE 会自动识别并关联该目录。

- **激活环境：**

  - Linux/Mac: `source .venv/bin/activate`
  - Windows: `.venv\Scripts\activate`
  - **如果 IDE 识别成功，打开终端时会自动激活。**

- 验证路径 (Debug)：

  可以通过打印 sys.path 查看当前 Python 解释器搜索库的路径顺序，确认是否使用了虚拟环境。

  Python

  ```
  import sys, pprint
  pprint.pprint(sys.path)
  ```

### 3. 环境复现与依赖管理

- **导出依赖：**

  Bash

  ```
  pip freeze > requirements.txt
  ```

  输出当前环境中安装的所有库及其确切版本。

- **Pip 模式的缺陷：**

  1. **缺乏层级区分：** `requirements.txt` 是“扁平”的。它混杂了**直接依赖**（你安装的 `flask`）和**间接依赖**（`flask` 依赖的 `werkzeug`）。
  2. **卸载残留：** `pip uninstall package` 只删除指定的库，**不删除**该库引入的间接依赖。随着时间推移，环境中会堆积大量无用的“僵尸依赖”。

------

## 二、 现代流派：uv (The Successor)

`uv` 是一个由 Rust 编写的极速 Python 包管理器，旨在直接解决上述 Pip 的痛点。

### 1. 解决依赖管理痛点

`uv` 引入了现代化的项目元数据管理标准。

- **pyproject.toml (项目配置)：**
  - 只记录**直接依赖**（例如：`dependencies = ["flask"]`）。
  - 这是给人看和维护的文件。
- **uv.lock (锁定文件)：**
  - 自动生成，记录所有直接和间接依赖的确切版本、哈希值。
  - 这是给机器看的，确保在任何机器上 `uv sync` 都能还原出一模一样的环境。

### 2. 解决卸载残留痛点

- **命令：** `uv remove package`
- **机制：** 当你移除一个库时，`uv` 会自动重新计算依赖树，并**自动清理**掉那些不再被任何库需要的间接依赖。始终保持环境清洁。

### 3. 常用命令对照表

| **操作**     | **传统 (Pip/Venv)**               | **现代 (uv)**         | **备注**                       |
| ------------ | --------------------------------- | --------------------- | ------------------------------ |
| **初始化**   | `python -m venv .venv`            | `uv init` / `uv venv` | uv 自动管理 Python 版本        |
| **安装库**   | `pip install numpy`               | `uv add numpy`        | uv 自动写入 `pyproject.toml`   |
| **同步环境** | `pip install -r requirements.txt` | `uv sync`             | uv 依据 lock 文件严格同步      |
| **删除库**   | `pip uninstall numpy`             | `uv remove numpy`     | uv 会自动清理无用依赖          |
| **临时运行** | (无直接对应功能)                  | `uv run script.py`    | 创建临时环境运行脚本，跑完即焚 |

------

## 三、 特殊概念补充

### 1. Editable Install (可编辑安装)

- **场景：** 开发自己的 Python 包（Library）时。

- **命令：**

  Bash

  ```
  pip install -e .
  # 或者
  uv pip install -e .
  ```

- **原理：** 在 `site-packages` 中创建一个**软链接**指向你的源代码目录，而不是复制文件。

- **效果：** 修改源代码后，无需重新安装，变更立即生效。

### 2. 为什么需要 pyproject.toml？

它是 Python 社区定义的新标准（PEP 518/621），旨在取代 `setup.py` 和 `requirements.txt`，统一管理项目的构建工具、元数据和依赖关系。它是使用 `uv`、`poetry` 或 `pdm` 等现代工具的基础。