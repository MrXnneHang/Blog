# C++ 调用Windows API 发送Message

## 前言：

这个追述起来就挺久远了。
大一的时候用类似的思路写了一个QQ聊天机器人。当时参考的视频是:[C++多线程轰炸机，QQ聊天轰炸](https://www.bilibili.com/video/BV1NB4y1e7nb?p=4&vd_source=d7601f0fc447d708fff71aa75186ea10)

相比这次讲的窗口，QQ那边还更复杂一些，需要做伪用户激活，需要窗口置顶才能发送消息。但是正常没有保护的窗口，比如这次的窗口调试助手，那个句柄是一Enum就把儿子孙子都列出来了。甚至标题都是中文没有伪装的。文本框的标题直接是文本内容，真的太热情。直接省掉我用python去建立串口通信的功夫。

但奈何还是太久没摸C++，生疏了一些，实现起来绕了些弯路。

这里直接记录这次的结论：

## 1.用SendMessage + WM_GETTEXT代替GetWindowText

第一次在直接采用GetWindowText的时候，我死活没法截取到文本输入框和文本输出框里面的内容。（而且这个内容用spy++可以获取到，是子窗口的标题）

![Snipaste_2024-07-01_20-55-44](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407012056411.jpeg)

但输出有类名，样式，句柄就是没有标题。

一再怀疑自己写错了什么，因为前一步用SendMessage发送文本到输入框是没有任何报错的。

gpt给出的解释是:

GetWindowText的局限是:当碰到自定义控件或者某些特殊窗口类的时候它无法正常工作，因为它和"EDIT"和"RichEDIT"控件的实现方式不兼容。而EDIT似乎就是文本框。

另外gpt也给我一个模板，下次再碰到就不用太复杂了。

```
#include <iostream>
#include <windows.h>
#include <vector>
#include <string>
#include <fcntl.h> // for _O_U16TEXT
#include <io.h> // for _setmode

// 回调函数用于收集子窗口句柄
BOOL CALLBACK EnumChildWindowsProc(HWND hwnd, LPARAM lParam) {
    auto& windows = *reinterpret_cast<std::vector<HWND>*>(lParam);
    windows.push_back(hwnd);
    return TRUE; // 继续枚举下一个子窗口
}

void PrintWindowInfo(HWND hwnd) {
    wchar_t className[256];
    wchar_t windowText[256];

    // 获取窗口类名
    GetClassName(hwnd, className, sizeof(className) / sizeof(className[0]));

    // 获取窗口样式
    DWORD style = GetWindowLong(hwnd, GWL_STYLE);

    // 获取窗口标题（包括对 EDIT 和 RichEdit 控件的处理）
    int length = GetWindowTextLength(hwnd);
    std::wstring text(length, L'\0');
    GetWindowText(hwnd, &text[0], length + 1);

    // 如果是 EDIT 或 RichEdit 控件，使用 SendMessage 获取文本
    if (className == std::wstring(L"Edit") || 
        className == std::wstring(L"RichEdit20W")) {
        length = SendMessage(hwnd, WM_GETTEXTLENGTH, 0, 0);
        text.resize(length);
        SendMessage(hwnd, WM_GETTEXT, length + 1, (LPARAM)&text[0]);
    }

    // 打印信息
    std::wcout << L"窗口句柄: " << hwnd << std::endl;
    std::wcout << L"类名: " << className << std::endl;
    std::wcout << L"标题: " << text << std::endl;
    std::wcout << L"样式: " << std::hex << style << std::endl;
    std::wcout << L"--------------------------" << std::endl;
}

void GetAndPrintWindowInfo(HWND hwndParent) {
    std::vector<HWND> childWindows;

    // 枚举父窗口下的所有子窗口
    if (!EnumChildWindows(hwndParent, EnumChildWindowsProc, reinterpret_cast<LPARAM>(&childWindows))) {
        std::wcerr << L"EnumChildWindows 失败" << std::endl;
        return;
    }

    std::wcout << L"找到子窗口数量: " << childWindows.size() << std::endl;

    // 打印所有子窗口的信息
    for (HWND hwnd : childWindows) {
        PrintWindowInfo(hwnd);
    }
}

int main() {
    // 设置控制台以支持 Unicode
    _setmode(_fileno(stdout), _O_U16TEXT);

    // 获取父窗口句柄
    HWND hwndParent = FindWindow(NULL, TEXT("父窗口标题")); // 替换为实际的窗口标题
    if (hwndParent == NULL) {
        std::wcerr << L"未找到父窗口" << std::endl;
        return 1;
    } else {
        std::wcout << L"找到父窗口\n";
    }

    GetAndPrintWindowInfo(hwndParent);

    return 0;
}

```

>1. **PrintWindowInfo** 函数获取并打印窗口信息。
>2. 使用 `GetWindowText` 获取标准控件的文本。
>3. 使用 `SendMessage` 和 `WM_GETTEXT` 获取 `EDIT` 和 `RichEdit` 控件的文本。
>4. 枚举子窗口并打印其信息。

## 2.输出C重定向到python终端时候的编码问题:

C++设置: 

1.输出UTF-8编码到终端:

```c++
_setmode(_fileno(stdout), _O_U8TEXT);
```

2.对你要打印的信息使用std::wcout:

```c++
std::wcout << L"窗口句柄: " << hwnd << std::endl;
std::wcout << L"类名: " << className << std::endl;
std::wcout << L"标题: " << text << std::endl;
std::wcout << L"样式: " << std::hex << style << std::endl;
std::wcout << L"--------------------------" << std::endl;
```



事已至此，顺便写一下如何让生成python能够调用的可执行文件和传参。

## 3.生成可传参调用的可执行文件。

### C++:

```c++
int wmain(int argc, wchar_t* argv[]) {



return 0;

}
```

这么写之后你是无法直接执行得到结果的，似乎会说Usage什么的，你可以先调试好再进行一个函数形式更改。

然后我用的是Visual Studio,窗口选项栏有一个生成->生成解决方案。或者ctrl+shift+B就可以生成exe文件在项目X64/Debug下方。

### Python的调用:

```python
def check_output_from_Cpp(send_data, open_chuankou=1, close_chuankou=0):
    result = subprocess.run(
        [r'D:\program\stm32\cpp_hwnd\x64\Debug\stmCpp.exe', send_data, str(open_chuankou), str(close_chuankou)],
        capture_output=True,
        text=True,
        encoding='utf-8'
    )
    return result.stdout

```

subprocess.run这里有一点要注意，似乎GPT给我写的用法经常会不打印输出。然后我复制粘贴这个写法就会把文件日志打印出来，我也没仔细对比差别在哪里。但碰到时这么用就可以了。



相关源代码都已在github存好:

[https://github.com/MrXnneHang/Embedded-System-Design-Course](https://github.com/MrXnneHang/Embedded-System-Design-Course)
