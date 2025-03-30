# LinuxPDF

这是通过 RISC-V 模拟器在 PDF 文件中运行的 Linux，它基于 [TinyEMU](https://bellard.org/tinyemu/).

你可以点此尝试：[linux.pdf](https://linux.doompdf.dev/linux.pdf)

https://github.com/user-attachments/assets/7e992dd1-41a5-4d32-87cc-878b395e3d92

另请参见：[DoomPDF](https://github.com/ading2210/doompdf)

## 原理解释

本项目和我之前的 [DoomPDF](https://github.com/ading2210/doompdf) 项目实现原理相似.

您可能认为 PDF 文件只由静态文档组成，但令人惊讶的是，PDF文件格式支持 Javascript，它有自己独立的标准库。现代浏览器（Chromium, Firefox）将此作为 PDF 引擎的一部分实现。然而，浏览器中可用的 API 要有限得多。

PDF 中 JavaScript 的完整规范仅由 Adobe Acrobat 实现过，它包含了一些非常不可思议的功能，例如进行[3D渲染](https://opensource.adobe.com/dc-acrobat-sdk-docs/library/jsapiref/JS_API_AcroJS.html#annot3d)、发起[HTTP请求](https://opensource.adobe.com/dc-acrobat-sdk-docs/library/jsapiref/JS_API_AcroJS.html#net-http)以及[检测连接到用户系统的每个显示器](https://opensource.adobe.com/dc-acrobat-sdk-docs/library/jsapiref/JS_API_AcroJS.html#monitor)。然而，在 Chromium 和其他浏览器上，由于显而易见的安全问题，仅实现了一小部分这样的 API。通过这种方式，我们可以进行任何计算，只是 IO 受到一些非常有限的限制。

可以使用一个旧版本的 Emscripten 将 C 代码编译为在 PDF 中运行，该版本针对的是 [asm.js](https://en.wikipedia.org/wiki/Asm.js) 而不是 WebAssembly。通过这种方式，我可以将 TinyEMU RISC-V 模拟器的一个修改版本编译为 asm.js，然后在 PDF 中运行。对于输入和输出，我重新使用了之前为 DoomPDF 编写的显示代码。它通过使用一组独立的文本字段来实现，每个文本字段对应屏幕上的每一行像素，其内容被设置为各种 ASCII 字符。对于输入，有一个虚拟键盘，由许多按钮组成，还有一个文本框，你可以在其中输入内容，以将按键发送到虚拟机。

这里最大的问题是模拟器的性能。例如，Linux 内核在 PDF 中启动大约需要 30 到 60 秒，这比正常情况下慢了 100 多倍。不幸的是，由于 Chrome 的 PDF 引擎所使用的 V8 版本的即时编译器（JIT compiler）被禁用了，因此无法解决这一问题，这严重破坏了其性能。

对于根文件系统，既可以选择 64 位版本，也可以选择 32 位版本。默认情况下，使用的是 32 位的 Buildroot 系统（这是预先构建的，并来自原始 TinyEMU 示例），此外还有一个 64 位的 Alpine Linux 系统。然而，64 位模拟器的速度大约只有 32 位模拟器的一半，因此通常不会使用它。

## 构建说明

克隆此仓库并运行以下命令：
```
python3 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt
./build.sh
```
如果要构建64位版本而不是32位版本，请编辑 `build.sh` 并将 `BITS="32"` 行更改为 `BITS="64"`。

`build.sh` 脚本将自动下载 Emscripten `1.39.20`。你必须在 Linux 上才能构建此项目。

生成的文件将位于 `out/` 目录中。然后你可以运行 `(cd out; python3 -m http.server)` 来在 Web 服务器上提供这些文件。

## 致谢

此项目由 [@ading2210](https://github.com/ading2210/) 制作。

RISC-V模拟器是从 [TinyEMU](https://bellard.org/tinyemu/) 分支而来的，由 [Fabrice Bellard](https://bellard.org/) 编写。

## 开源许可

此仓库在 GNU GPL v3 下授权.

```
ading2210/linuxpdf - Linux running inside a PDF file
Copyright (C) 2025 ading2210

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```
