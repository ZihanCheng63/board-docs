---
sys: baremetal
sys_ver: null
sys_var: null
status: basics
last_update: 2026-05-31
model: Nuclei RV-STAR
profile: Hello World

---

# RuyiSDK 基础示例

可直接在开发板上进行编译和运行的示例，适合初学者快速上手。

安装依赖包

```

sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential

```

安装 ruyi 包管理器

```

wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.47.0/ruyi-0.47.0.riscv64

chmod +x ruyi-0.47.0.riscv64

sudo cp -v ruyi-0.47.0.riscv64 /usr/local/bin/ruyi

```

安装工具链

```

ruyi update

ruyi install gnu-plct llvm-plct

```

克隆 Nuclei SDK

```

git clone https://github.com/Nuclei-Software/nuclei-sdk.git

cd nuclei-sdk

```

下载并配置 Nuclei 工具链和 OpenOCD

```

wget https://download.nucleisys.com/upload/files/toolchain/gcc/nuclei_riscv_newlibc_prebuilt_linux64_nuclei-2024.tar.bz2
tar -xjvf nuclei_riscv_newlibc_prebuilt_linux64_nuclei-2024.tar.bz2

wget https://download.nucleisys.com/upload/files/toolchain/openocd/nuclei-openocd-2024.02.28-linux-x64.tgz
tar -xzvf nuclei-openocd-2024.02.28-linux-x64.tgz

export PATH=~/nuclei-sdk/gcc/bin:$PATH
export PATH=~/nuclei-sdk/Nuclei/openocd/2024.02.28/bin:$PATH

sed -i 's/ -Wl,--no-warn-rwx-segments//' ~/nuclei-sdk/Build/toolchain/nuclei_gnu.mk

```

在终端1启动 OpenOCD

```

sudo openocd -f ~/nuclei-sdk/SoC/gd32vf103/Board/gd32vf103v_rvstar/openocd_gd32vf103.cfg

```

## 说明

RV-STAR 是裸机开发板，需要使用 Nuclei 官方提供的专用工具链（包含针对 GD32VF103 芯片的启动文件和链接脚本）。RuyiSDK 作为统一的工具链管理平台，整合了该专用工具链。

## Hello World (GCC版)

编译Hello World

```
cd ~/nuclei-sdk/application/baremetal/helloworld

cat > main.c << 'EOF'
#include <stdio.h>

int main(void) {
    printf("Hello RISC-V from RV-STAR!\n");
    printf("Testing GCC toolchain with Nuclei SDK.\n");
    return 0;
}
EOF

make SOC=gd32vf103 BOARD=gd32vf103v_rvstar TOOLCHAIN=nuclei_gnu clean
make SOC=gd32vf103 BOARD=gd32vf103v_rvstar TOOLCHAIN=nuclei_gnu all

```

在终端2执行烧录

```

sudo chmod 666 /dev/ttyUSB1

riscv64-unknown-elf-gdb helloworld.elf -ex "target extended-remote localhost:3333" -ex "monitor reset halt" -ex "load" -ex "monitor resume" -ex "quit"

minicom -D /dev/ttyUSB1 -b 115200

```

正常情况下，终端会看到类似如下输出：

```

Nuclei SDK Build Time: May Nuclei SDK Build Time: May 31 202Nuclei SDK Build Time: May0
�Nuclei SDK Build Time: May 31 2026, 17:59:30
Download Mode: FLASHXIP
CPU Frequency 108000000 Hz
Hello RISC-V from RV-STAR!
Testing GCC toolchain with Nuclei SDK.

```

## Hello World (LLVM版)

编译Hello World

```
cd ~/nuclei-sdk/application/baremetal/helloworld

cat > main.c << 'EOF'
#include <stdio.h>

int main(void) {
    printf("Hello RISC-V from RV-STAR!\n");
    printf("Testing LLVM/Clang toolchain with Nuclei SDK.\n");
    return 0;
}
EOF

make SOC=gd32vf103 BOARD=gd32vf103v_rvstar TOOLCHAIN=nuclei_llvm clean
make SOC=gd32vf103 BOARD=gd32vf103v_rvstar TOOLCHAIN=nuclei_llvm all

```

在终端2执行烧录

```

sudo chmod 666 /dev/ttyUSB1

riscv64-unknown-elf-gdb helloworld.elf -ex "target extended-remote localhost:3333" -ex "monitor reset halt" -ex "load" -ex "monitor resume" -ex "quit"

minicom -D /dev/ttyUSB1 -b 115200

```

正常情况下，终端会看到类似如下输出：

```

Nuclei SDK Build Time: May 31 2026, 18:01:49
Download Mode:Nuclei SDK Build Time: May 31 2026, 18:01:Nuclei SDK Build Time: May 31 7
Download Mode�Nuclei SDK Build Time: May 31 2026, 18:04:17
Download Mode: FLASHXIP
CPU Frequency 108000000 Hz
Hello RISC-V from RV-STAR!
Testing LLVM/Clang toolchain with Nuclei SDK.

```
