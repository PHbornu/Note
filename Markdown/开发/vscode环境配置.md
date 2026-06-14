# vscode环境配置

VS Code 默认并不自带编译和调试的功能，而是通过插件来扩展这些功能。对于 C++ 开发，通常需要安装 C++ 插件（如 Microsoft 的 C++ 插件）和配置一些文件来指定如何进行编译和调试。tasks.json 和 launch.json 是 VS Code 配置 C++ 项目编译和调试的关键文件。

1、	为什么 VS Code 编译和调试 C++ 项目需要额外配置 tasks.json 和 launch.json 文件？
+ tasks.json：配置编译任务，告诉 VS Code 如何使用编译器（如 g++ 或 clang++）编译源代码。
+ launch.json：配置调试任务，告诉 VS Code 如何启动调试器并调试程序。

2、C++开发多文件编译
找到`tasks.json`中的`args`选项，这个主要是用来配置待编译的文件信息的，
主要就是将：`${file}`替换成`${workspaceFolder}`,最终配置的结果修改如下：
[微软配置文档](https://code.visualstudio.com/docs/cpp/config-mingw#_compiler-path)
```
"args": [
    "-fdiagnostics-color=always",
    "-g",
    "${workspaceFolder}\\*.cpp",
    "-o",
    "${fileDirname}\\${fileBasenameNoExtension}.exe"
],
```


C++
```C++
#include<iostream>
using namespace std;
int main()
{
    string name;

    cout << "输入您的姓名" << endl;
    cin >> name;
    cout << "你好" << name << endl;
}
```


launch.json
```json
{
    "version": "0.2.0",  
    "configurations": [  
        {  
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg",       // 配置类型，这里只能为cppdbg
            "request": "launch",    // 请求配置类型，可以为launch（启动）或attach（附加）  
            "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe",// 将要进行调试的程序的路径  
            "args": [],             // 程序调试时传递给程序的命令行参数，一般设为空即可  
            "stopAtEntry": false,   // 设为true时程序将暂停在程序入口处，一般设置为false  
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录，一般为${workspaceFolder}即代码所在目录  
            "environment": [],  
            "externalConsole": true, // 调试时是否显示控制台窗口，一般设置为true显示控制台  
            "MIMode": "gdb",  
            "miDebuggerPath": "D:\\app\\setup\\mingw\\install\\mingw64\\bin\\gdb.exe", // miDebugger的路径，注意这里要与MinGw的路径对应  
            "preLaunchTask": "gcc", // 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc  
            "setupCommands": [  
                {   
            "description": "Enable pretty-printing for gdb",  
                    "text": "-enable-pretty-printing",  
                    "ignoreFailures": true  
                }  
            ]  
        }  
    ]  
}
```

tasks.json
```json
{
    // 有关 tasks.json 格式的参考文档：https://go.microsoft.com/fwlink/?LinkId=733558 。
    "version": "2.0.0",
    "tasks": [{
        "label": "gcc",
        "type": "shell", // { shell | process }
        // 适用于 Windows 的配置：
        "windows": {
            "command": "gcc",
            "args": [
                "-g",
                "\"${file}\"",
                "-o",
                "\"${fileDirname}\\${fileBasenameNoExtension}.exe\""
                // 设置编译后的可执行文件的字符集为 GB2312：
                // "-fexec-charset", "GB2312"
                // 直接设置命令行字符集为 utf-8：
                // chcp 65001
            ]
        },
        // 定义此任务属于的执行组：
        "group": {
            "kind": "build", // { build | test }
            "isDefault": true // { true | false }
        },
        // 定义如何在用户界面中处理任务输出：
        "presentation": {
            // 控制是否显示运行此任务的面板。默认值为 "always"：
            // - always:    总是在此任务执行时显示终端。
            // - never:     不要在此任务执行时显示终端。
            // - silent:    仅在任务没有关联问题匹配程序且在执行时发生错误时显示终端
            "reveal": "silent",
            // 控制面板是否获取焦点。默认值为 "false"：
            "focus": false,
            // 控制是否将执行的命令显示到面板中。默认值为“true”：
            "echo": false,
            // 控制是否在任务间共享面板。同一个任务使用相同面板还是每次运行时新创建一个面板：
            // - shared:     终端被共享，其他任务运行的输出被添加到同一个终端。
            // - dedicated:  执行同一个任务，则使用同一个终端，执行不同任务，则使用不同终端。
            // - new:        任务的每次执行都使用一个新的终端。
            "panel": "dedicated"
        },
        // 使用问题匹配器处理任务输出：
        "problemMatcher": {
            // 代码内问题的所有者为 cpp 语言服务。
            "owner": "cpp",
            // 定义应如何解释问题面板中报告的文件名
            "fileLocation": [
                "relative",
                "${workspaceFolder}"
            ],
            // 在输出中匹配问题的实际模式。
            "pattern": {
                // The regular expression.
                "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                // 第一个匹配组匹配文件的相对文件名：
                "file": 1,
                // 第二个匹配组匹配问题出现的行：
                "line": 2,
                // 第三个匹配组匹配问题出现的列：
                "column": 3,
                // 第四个匹配组匹配问题的严重性，如果忽略，所有问题都被捕获为错误：
                "severity": 4,
                // 第五个匹配组匹配消息：
                "message": 5
            }
        }
    }]
}
```


setting.json
```json
{
    "files.associations": {
        "tidl_alg_int.h": "c",
        "limits": "c"
    }
}
```


## 问题

### cppdbg 不受支持
#### 原因1
[配置的类型“cppdbg”不受支持 使用vscode编译c++程序时出现这句话，什么意思是，怎么解决？](https://www.zhihu.com/question/266875809/answer/2829448976)
```
原因是，安装在服务器remote的安装的cpptools是windows软件，在linux不支持，你可以看~/.vscode-server/extensions/ms-vscode.cpptools-1.5.1/bin/（或者 .remotedev-server/extensions/ms-vscode.cpptools-1.5.1/bin）下是**.exe可执行文件。所以，首先，卸载现有的romote端的cpptools，删除服务器~/.vscode-server/extensions（或者 ./remotedev-server/extensions/ ）路径下相应的软件，重载vscode。然后，从https://github.com/microsoft/vscode-cpptools/releases?page=5，根据vscode的版本，看git cpptools 的Requirements，寻找合适的版本，然后，下载cpptools-linux.vsix，拷贝到linux服务器的~/.vscode-server/extensions（或者 ./remotedev-server/extensions/ ）路径下，
最后，在windows点击
```

#### 原因2
[配置的类型“cppdbg”不受支持 使用vscode编译c++程序时出现这句话，什么意思是，怎么解决？](https://www.zhihu.com/question/266875809/answer/3547965167)
```
没安装C/C++拓展或者禁用了这个拓展，导致配置的类型“cppdbg”不受支持。安装C/C++拓展或者启用拓展就好了。
```