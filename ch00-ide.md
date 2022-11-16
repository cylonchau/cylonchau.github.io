# ch0 ide


## Visual Studio使用

### 离线安装包

在页面 <sup><a href="#4">[4]</a></sup> 下载安装引导命令，下载完成后使用命令（对于C++来说）

```bat
vs_Professional.exe --layout ‪1111 --add Microsoft.VisualStudio.Workload.NativeDesktop --includeRecommended --lang en-US zh-CN
```

随后会触发下载，等待下载完成后，在 `--layout` 指定的目录上点击 ***vs_setup*** 开始离线安装。

> Note: 对于完全脱离C盘安装可以使用下面的脚本，更改变量为要安装的路径

```bat
:: 关闭终端回显
@echo off

SET ROOT_PATH=D:\Program Files\Microsoft Visual Studio
SET X86_PATH=%ROOT_PATH%\Program Files (x86)
SET X86_VS_PATH=%X86_PATH%\Microsoft Visual Studio
SET X86_SDK_PATH=%X86_PATH%\Microsoft SDKs
SET X86_KITS_PATH=%X86_PATH%\Windows Kits
SET X86_AV_PATH=%X86_PATH%\Application Verifier

SET X64_PATH=%ROOT_PATH%\Program Files
rem SET X64_VS_PATH=%X64_PATH%\Microsoft Visual Studio
SET X64_AV_PATH=%X64_PATH%\Application Verifier
SET X64_SQL_PATH=%X64_PATH%\Microsoft SQL Server

SET PD_PATH=%ROOT_PATH%\ProgramData
SET PD_VS_PATH=%PD_PATH%\Microsoft\VisualStudio
SET PD_PC_PATH=%PD_PATH%\Package Cache

@echo =======link directory to %ROOT_PATH%=======:

SET S_X86_SKD_PATH=C:\Program Files (x86)\Microsoft SDKs
SET S_X86_VS_PATH=C:\Program Files (x86)\Microsoft Visual Studio
SET S_X86_KITS_PATH=C:\Program Files (x86)\Windows Kits
SET S_X86_AV_PATH=C:\Program Files (x86)\Application Verifier

SET S_X64_AV_PATH=C:\Program Files\Application Verifier
SET S_X64_SQL_PATH=C:\Program Files\Microsoft SQL Server

SET S_PD_VS_PATH=C:\ProgramData\Microsoft\VisualStudio
SET S_PD_PC_PATH=C:\ProgramData\Package Cache


pause
@echo =======setting visual studio environment=======:


@echo =======check directory exist=======:

if not exist %ROOT_PATH% (
	echo "%ROOT_PATH%目录不存在，已创建该目录！"
	md "%ROOT_PATH%"
)

if not exist %X86_PATH% (
	echo "%X86_PATH%目录不存在，已创建该目录！"
	md "%X86_PATH%"
)

if not exist %X86_VS_PATH% (
	echo "%X86_VS_PATH%目录不存在，已创建该目录！"
	md "%X86_VS_PATH%"
)

if not exist %X86_SDK_PATH% (
	echo "%X86_SDK_PATH%目录不存在，已创建该目录！"
	md "%X86_SDK_PATH%"
)

if not exist %X86_KITS_PATH% (
	echo "%X86_KITS_PATH%目录不存在，已创建该目录！"
	md "%X86_KITS_PATH%"
)

if not exist %X86_AV_PATH% (
	echo "%X86_AV_PATH%目录不存在，已创建该目录！"
	md "%X86_AV_PATH%"
)

if not exist %X64_PATH% (
	echo "%X64_PATH%目录不存在，已创建该目录！"
	md "%X64_PATH%"
)

if not exist %X64_AV_PATH% (
	echo "%X64_AV_PATH%目录不存在，已创建该目录！"
	md "%X64_AV_PATH%"
)

if not exist %X64_SQL_PATH% (
	echo "%X64_SQL_PATH%目录不存在，已创建该目录！"
	md "%X64_SQL_PATH%"
)

if not exist %PD_PATH% (
	echo "%PD_PATH%目录不存在，已创建该目录！"
	md "%PD_PATH%"
)

if not exist %PD_VS_PATH% (
	echo "%PD_VS_PATH%目录不存在，已创建该目录！"
	md "%PD_VS_PATH%"
)

if not exist %PD_PC_PATH% (
	echo "%PD_PC_PATH%目录不存在，已创建该目录！"
	md "%PD_PC_PATH%"
)


@echo =======link directory to %ROOT_PATH%=======:
:: x86 link
mklink /j "%S_X86_SKD_PATH%" "%X86_SDK_PATH%"
mklink /j "%S_X86_VS_PATH%"  "%X86_VS_PATH%"
mklink /j "%S_X86_KITS_PATH%" "%X86_KITS_PATH%"
mklink /j "%S_X86_AV_PATH%"  "%X86_AV_PATH%"

:: x64 link
mklink /j "%S_X64_AV_PATH%" "%X64_AV_PATH%"
mklink /j "%S_X64_SQL_PATH%" "%X64_SQL_PATH%"

:: ProgramData link
mklink /j "%S_PD_VS_PATH%" "%PD_VS_PATH%" 
mklink /j "%S_PD_PC_PATH%" "%PD_PC_PATH%"
pause
```

### VS快捷键

| 快捷键            | 含义           |
| ----------------- | -------------- |
| Ctrl + k,Ctrl + f | 自动格式化代码 |
| Ctrl + k,Ctrl + c | 注释代码       |
| Ctrl + k,Ctrl + u | 取消注释代码   |
| F9                | 设置断点       |
| F5                | 调试运行       |
| Ctrl + F5         | 不调试运行     |
| Ctrl + Shift + b  | 编译，不运行   |
| F10               | next调试       |
| F11               | step调试       |

### 调试

添加行号：工具--》选项 --》文本编辑器--》C/C++ --》行号

调试步骤

- 设置断点。F5启动调试
- 停止（断点处）的位置，是尚未执行的指令。
- 逐语句执行一下条 （F11）：进入函数内部，逐条执行跟踪。
- 逐过程执行一下条 （F10）：不进入函数内部，逐条执行程序。
- 监视：调试 --》窗口 --》监视：输入监视变量名。自动监视变量值的变化。

## VS Code使用

安装扩展 ***C/C++ Extension Pack***，***Code Runner***

调试相关快捷键：

- F5 进入调试
- F9 切换断点
- F10 单步跳过（逐过程执行）
- F11 单步执行（逐语句执行，可进入执行函数体）
- Shift+F5 停止调试
- Ctrl+Shift+F5重启调试
- Ctrl+F5 开始执行，不进入断点
- Ctrl+F9 启用/停止断点
- Ctrl+Shift+F9 删除全部断点
- Ctrl+b 隐藏/打开侧边框
- Ctrl+\` 隐藏/打开terminal
- Ctrl+j 隐藏/打开下边框（plannel）
- Ctrl+Shift+D 打开侧边框 Run and Debug
- Ctrl+Shift+E 打开侧边框 Explorer
- Ctrl+Alt+N run code

配置 ***launch.json*** 根据提示，替换gcc路径即可

```json
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gcc.exe - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\Program Files\mingw64\bin\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: gcc.exe build active file"
        }
    ]
}
```


