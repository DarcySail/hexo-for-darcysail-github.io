---
title: How to crack Synergy
copyright: true
date: 2018-05-31 15:23:19
tags: [crack, tips]
comments:
---

Synergy 是一款用于在不同电脑之间共享键盘、鼠标、剪贴板的开源商业软件，支持包括 Linux、Windows、Mac OS 在内的多种主流操作系统，其 Pro 版本甚至支持直接在不同桌面上面拖动文件。

笔者使用的 Linux 系统有免费的基础版可以使用，但想要使用共享剪贴板功能，则需付费购买 Pro 版。这篇文章简单描述了从源代码逆向其付费序列号的过程。由于实在太败人品了……不会放出序列号，读者可以自行尝试编译笔者提供的代码。

<!--more-->

![Synergy 激活窗口](synergy_active_window.png)

观察 Synergy 的激活窗口，注意到存在 "serial key"关键字。
从 Github 将 Synergy 源码仓库 clone 下来之后，使用`ag "serial key"`得到如下结果。从中我们可以定位到 `src/lib/shared/SerialKey.cpp`文件。
```bash
$ ag "serial key"
ChangeLog
7:Bug #5901 - Stored serial key corrupted on macOS
40:Bug #5722 - Malformed serial key in registry will crash GUI on startup
84:Bug #5471 - Serial key textbox on activation screen overflows on Mac
99:Enhancement #4715 - Activation dialog which also accepts a serial key
100:Enhancement #5020 - Recommend using serial key when online activation fails
105:Enhancement #4716 - Allow software to be time limited with serial key

src/lib/shared/SerialKey.cpp
53:        throw std::runtime_error ("Invalid serial key");
```

> https://github.com/symless/synergy-core/blob/master/src/lib/shared/SerialKey.cpp

粗略扫视一眼 SerialKey.cpp 文件，确定如下两个函数可能是对序列号进行解析的关键。

```c
SerialKey::decode(const std::string& serial);
SerialKey::parse(std::string plainSerial);
```

decode 函数并不复杂，就不做分析了。继续跟踪函数逻辑流，发现代码中仅仅对序列号版本（Basic 或者 Pro）以及软件过期时间进行了检验，其余 filed 可以随便乱填。Synergy 毕竟是家小公司，做事可能不太严谨。不过话说回来，既然公司把源码都公布出来，自行编译也实在不是什么难事。所以我做的这事的确是有点败坏人品的，图灵祖师宽恕则个……

下附简单的 Crack Pro 版本序列号的代码。读者可以自行编译，但请勿传播。


```cpp
#include <iostream>
#include <string>
using namespace std;

string decode(const std::string &serial)
{
    static const char *const lut = "0123456789ABCDEF";
    string output;
    int len = serial.length();
    if ((len & 1) != 0u) {
        return output;
    }

    output.reserve(len / 2);
    for (int i = 0; i < len; i += 2) {

        char a = serial[i];
        char b = serial[i + 1];

        const char *p = lower_bound(lut, lut + 16, a);
        const char *q = lower_bound(lut, lut + 16, b);

        if (*q != b || *p != a) {
            return output;
        }

        output.push_back(static_cast<char>(((p - lut) << 4) | (q - lut)));
    }

    return output;
}

string encode(string src)
{
    string result;
    static const char *const lut = "0123456789ABCDEF";
    for (int i = 0; i < src.length(); ++i) {
        char a = src[i];
        char p = (a & 0xf0) >> 4;
        char q = (a & 0x0f);
        //        const char *x = lower_bound(lut, lut + 16, p);
        //        const char *y = lower_bound(lut, lut + 16, q);

        result.push_back(lut[p]);
        result.push_back(lut[q]);
    }
    return result;
}

int main(int argc, char *argv[])
{
    // e.g.: {v1;basic;Bob;1;email;company name;1398297600;1398384000}
    cout << encode("{v1;pro;Bob;1;email;company name;1398297600;1398384000}");
    return 0;
}
```
