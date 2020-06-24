---
layout: post
title: VScode-Python 代码静态检查工具pylint及代码格式化工具yapf的配置使用
subtitle: VScode开发Python程序
tags: [Tools]
---
<!-- ## VScode开发Python程序：代码静态检查工具pylint及代码格式化工具yapf的配置使用 -->

### 准备工作

1. 安装Python和pip工具，使用pip安装pylint和yapf：

```shell
pip install pylint yapf
```

2. 下载安装vscode：https://code.visualstudio.com/
3. 打开vscode，点击左侧`扩展`图标按钮，点击`更多`选择`显示常用的扩展` ，选择并安装插件`Python`（Microsoft官方发布），安装完成后点击`重新加载`即可重启vscode并激活`Python`插件

### 代码静态检查工具：pylint

#### 默认设置

在vscode中依次点击`文件 -> 首选项 -> 设置`打开设置文件`settings.json`编辑器标签标题显示（`User Settings`），vscode会自动分左右两栏显示，其中左栏是 `默认用户设置` （已锁定为只读） ，右栏是`用户设置`，用户写在右栏的自定义设置会覆盖掉左栏相应的默认设置。

在`settings.json`页面上方搜索栏内输入`python.linting`即可快速定位与`Python`代码静态检查工具相关的设置内容。激活 Python 插件后，默认设 置已包含以下内容：

```json
// 默认对Python文件进行静态检查
"python.linting.enabled": true,

// 默认在Python文件保存时进行静态检查
"python.linting.lintOnSave": true,

// 默认使用pylint对Python文件进行静态检查
"python.linting.pylintEnabled": true,
```

一般情况下直接采用以上默认设置即可启用pylint。当Python文件保存时，vscode会调用pylint执行静态检查，如检查发现问题则会以错误、警告或信息等形式显示在vscode下方的问题页面，同时在文本编辑器中以波浪线形式标示问题代码。

#### 自定义设置：自定义pylint规则选项

以上默认设置的一个问题在于禁用了大部分pylint检查项目，导致pylint检查显示出的问题较少，对于某些简单的Python文件，pylint检查甚至不会显示出任何问题，仿佛pylint没有被实际调用一样。这是由于`Python`插件在 `2018.1.0 (Jan. 2018)` [版本更新](https://devblogs.microsoft.com/python/python-in-visual-studio-code-jan-2018-release/)中引入了一项默认设置：

```json
// 静态检查时是否使用pylint的最小规则集（minimal set of rules）
"python.linting.pylintUseMinimalCheckers": true
```

这一默认设置等价于以下pylint选项：

```
--disable=all  --enable=F,E,unreachable,duplicate-key,unnecessary-semicolon,global-variable-not-assigned,unused-variable,binary-op-exception,bad-format-string,anomalous-backslash-in-string,bad-open-mode
```


不难看出，这样的默认设置直接禁用了所有的规范（Convertion，`C` ）和重构（Refactor，`R` ）类规则，只保留了致命错误（Fatal，`F` ）、错误（Error，`E` ）和少数几个警告（Warning，`W` ）类规则，因此pylint检查显示出的问题数量大幅减少。

自定义pylint规则选项：

- 如果希望直接应用所有pylint检查规则，则可以简单将 `"python.linting.pylintUseMinimalCheckers"` 的值修改为 `false`。
- 如果只希望应用部分pylint检查规则或者需要使用pylint某些选项，则可在`settings.json`中找到以下默认设置：

```json
// pylint选项，每个选项都是数组中的一个字符串元素
"python.linting.pylintArgs": []
```

在数组 `[]` 中加入所需的pylint选项即可，例如

```json
// pylint选项，以逗号分隔的字符串元素
"python.linting.pylintArgs": [
    "--disable=W0402,R0201,C0304", 
    "--extension-pkg-whitelist=numpy"
]
```

此时 `"python.linting.pylintUseMinimalCheckers"` 的值将自动被覆盖为 `false`，不用再手动修改。

#### 其它代码静态检查工具：flake8、mypy、pydocstyle、pep8、prospector、pylama

除pylint外，vscode的`Python`插件还支持flake8、mypy、pydocstyle、pep8、prospector、pylama等代码静态检查工具，但默认情况下不使用这些工具，仅使用pylint：

```json
// 默认不使用flake8对Python文件进行静态检查
"python.linting.flake8Enabled": false,

// 默认不使用mypy对Python文件进行静态检查
"python.linting.mypyEnabled": false,

// 默认不使用pep8对Python文件进行静态检查
"python.linting.pep8Enabled": false,

// 默认不使用prospector对Python文件进行静态检查
"python.linting.prospectorEnabled": false,

// 默认不使用pydocstyle对Python文件进行静态检查
"python.linting.pydocstyleEnabled": false,

// 默认不使用pylama对Python文件进行静态检查
"python.linting.pylamaEnabled": false,

// 默认使用pylint对Python文件进行静态检查
"python.linting.pylintEnabled": true
```

如需使用某工具，只需要将上方对应的 `"python.linting.xxxEnabled"` 改为 `true` 即可（需要先用 pip 安装对应工具）。vscode可以同时调用多种工具进行代码静态检查，其输出互相独立，因此同一问题可能会被不同工具重复输出多次。

以上工具规则选项的自定义方法与pylint类似，找到并修改以下设置即可：

```json
// xxx选项，以逗号分隔的字符串元素
"python.linting.xxxArgs": []
```

### 代码格式化工具：yapf

在`settings.json`页面上方搜索栏内输入`python.formatting`即可快速定位与Python代码格式化工具相关的设置内容。激活`Python`插件后，默认设置已包含以下内容：

```json
// Python代码格式化工具，可选'autopep8'、'black'或'yapf'.
"python.formatting.provider": "autopep8",
```

`Python` 插件默认的代码格式化工具是`autopep8`，也可以使用`black`或`yapf`。[推荐](https://blog.csdn.net/c1990h/article/details/80119873)使用[Google开发](https://github.com/google/yapf)的`yapf`：

- 首先，安装yapf：`pip install yapf`
- 其次，将上面 `"python.formatting.provider"` 的值改为 `"yapf"`
- 最后，如需自定义yapf选项，则可找到 `""python.formatting.yapfArgs": []"` 并在数组 `[]` 中加入所需选项，例如

```json
// yapf选项，以逗号分隔的字符串元素
"python.formatting.yapfArgs": [
   "--style", "{based_on_style: chromium, indent_width: 4}"
]
```


以上安装配置完成后，在Python代码编辑界面按 `F1` 或 `Ctrl + Shift + P` 打开命令面板，输入 `Format Document` 即可调用yapf进行格式化，也可以在代码编辑页面使用默认快捷键直接调用yapf：`Ctrl + Shift + I`（[Linux系统](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-linux.pdf)）、`Shift + Alt + F` （[Windows系统](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)）或 `⇧(Shift) + ⌥(Option) + F`（[Mac系统](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf)）。

以上安装配置完成后，在Python代码编辑界面按 F1 或 Ctrl + Shift + P 打开命令面板，输入 Format Document 即可调用yapf进行格式化，也可以在代码编辑页面使用默认快捷键直接调用yapf：Ctrl + Shift + I（Linux系统）、Shift + Alt + F （Windows系统）或 ⇧(Shift) + ⌥(Option) + F（Mac系统）。

除以上手动调用yapf以外，yapf也可以像pylint那样在文件保存甚至是代码段粘贴时自动调用，与之相关的设置是

```JSON
// 控制编辑器是否应自动设置粘贴内容的格式。格式化程序必须可用并且能设置文档中某一范围的格式。
"editor.formatOnPaste": false,

// 保存时设置文件的格式。格式化程序必须可用，不能自动保存文件，并且不能关闭编辑器。
"editor.formatOnSave": false,

// 在保存时格式化操作的超时时间。为 formatOnSave 命令指定时间限制 (单位: 毫秒)。运行超过设定时间的命令将被取消。
"editor.formatOnSaveTimeout": 750,
```


可以看出这些设置属于`编辑器editor`，不是 `Python` 插件的专属设置，因此这些设置会影响所有配置有格式化工具的语言文件。`"editor.formatOnPaste"` 和 `"editor.formatOnSave"` 默认均为 `false` ，主要是为了避免频繁自动调用格式化工具对编辑工作产生干扰，如确实需要可以将其改为 `true`。

可以看出这些设置属于 editor ，不是 Python 专属设置，因此这些设置会影响所有配置有格式化工具的语言文件。"editor.formatOnPaste" 和 "editor.formatOnSave" 默认均为 false ，主要是为了避免频繁自动调用格式化工具对编辑工作产生干扰，如确实需要可以将其改为 true。

### 常见报错

vscode报错：`Linter pylint is not installed` 或者 `Path to the pylint linter is invalid`

- 首先，确认pylint已正确安装，可打开命令行/终端测试 `pip show pylint` 能否正确打印模块信息。
- 其次，在vscode中打开设置文件 `settings.json` ，找到以下默认设置：

```json
// pylint的默认调用路径
"python.linting.pylintPath": "pylint",
```


其中 `"python.linting.pylintPath"` 给定pylint的调用路径，正常情况下此路径应该已包含在系统环境变量中，因此采用以上默认设置即可直接调用	（可打开命令行/终端测试 `pylint --version` 能否正确打印版本信息）；如无法直接调用pylint，则应将此项设置修改为pylint的正确调用路径。

- 最后，如果电脑上同时安装有Python2和Python3，则需要注意区分和引导vscode采用哪个Python版本。对于同时安装有Python2和Python3的Linux系统，若只需开发Python3程序，并只对Python3进行代码静态检查（请确认已正确安装适配Python3版本的pylint模块），则可打开设置文件`settings.json`，找到以下默认设置：

```json
// 默认Python路径
"python.pythonPath": "python",
```


将 `"python.pythonPath"` 改为Python3的路径：

```json
// Linux系统的Python3路径
"python.pythonPath": "/usr/bin/python3",
```


对于Windows系统，以上Python3路径应设置为：

```json
// Windows系统的Python3路径
"python.pythonPath": "x:\\xxx\\Python3x\\python.exe",
```

vscode报错：`Formatter yapf is not installed`
与上面 `Linter pylint is not installed` 报错类似，先确认是否已正确安装 `yapf`，再找到设置语句 `"python.formatting.yapfPath": "yapf"` 将其修改为正确调用路径，最后检查多版本共存情况下的Python路径。

本文转载自[CSDN](https://blog.csdn.net/sunxb10/article/details/80984243)

