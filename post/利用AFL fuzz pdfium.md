# 下载源码

安装depot-tools后下载源码

```bash
# 安装depot-tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"
# 下载pdfium
mkdir repo
cd repo
gclient config --unmanaged https://pdfium.googlesource.com/pdfium.git
gclient sync
cd pdfium
```

# 编译

ubuntu 或者 Debian 系统可直接使用 `./build/install-build-deps.sh`安装依赖（不是的可以加上`--unsupported`试试，或者手动配置依赖），利用`gn args out/afl`生成编译参数文件，参考如下

```ini
# Build arguments go here.
# See "gn args <out_dir> --list" for available build arguments.
use_goma = false # Googlers only. Make sure goma is installed and running first.
is_debug = false  # Enable debugging features.

pdf_use_skia = true # Set true to enable experimental skia backend.
pdf_use_skia_paths = false  # Set true to enable experimental skia backend (paths only).

pdf_enable_xfa = true  # Set false to remove XFA support (implies JS support).
pdf_enable_v8 = true  # Set false to remove Javascript support.
pdf_is_standalone = true  # Set for a non-embedded build.
is_component_build = false # Disable component build (must be false)
v8_static_library = true

clang_use_chrome_plugins = false  # Currently must be false.
use_sysroot = false  # Currently must be false on Linux, but entirely omitted on windows.

use_afl = true
is_asan = true
enable_nacl = true
optimize_for_fuzzing = true
symbol_level=1
```

> pdfium 源码仓库中没有afl-fuzz 的代码，需要自己下载。（默认版本为2.52b，推荐使用2.57b)
>
> `https://chromium.googlesource.com/chromium/src/third_party/+/master/afl/ `

使用 `ninja -C out/afl` 编译

# 开始Fuzz

afl-fuzz 的使用和其他项目一样。初始的种子文件有几个地方可以获取：

- https://pdfium.googlesource.com/pdfium/+/refs/heads/master/testing/resources/
- https://github.com/mozilla/pdf.js/tree/master/test/pdfs

```shell
./afl-fuzz -M 01 -m none -t 6000 -i /home/fuzz/input -o /home/fuzz/out -x /home/fuzz/pdf.dict -- ./pdfium_test @@
```

