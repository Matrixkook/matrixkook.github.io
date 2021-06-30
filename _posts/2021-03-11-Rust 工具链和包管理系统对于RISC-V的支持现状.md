[toc]



# Rust 工具链和包管理系统对于RISC-V的支持现状



##　测试步骤



| test            |                                       |
| :-------------- | :-----------------------------------: |
| Host            |             ubantu 20.04              |
| Qemu            |                 v5.1                  |
| Base            |  Intel Core i7-7700HQ @ 8x 2.801GHz   |
| Target          | riscv64 / riscv64gc-unknown-linux-gnu |
| Rust toolchain  |         rustc 1.52.0-nightly          |
| riscv toolchain |              up-to-date               |



> Roadmap of testing rust toolchain for RISC-V [note here](https://www.notion.so/dustbin/600be9882640461683e54cd8532863b6?v=1dc81dde95b34685b782188267123c69)





| target                         | std  | host | notes                                  |
| ------------------------------ | ---- | ---- | -------------------------------------- |
| `riscv32i-unknown-none-elf`    | *    |      | Bare RISC-V (RV32I ISA)                |
| `riscv32imac-unknown-none-elf` | *    |      | Bare RISC-V (RV32IMAC ISA)             |
| `riscv32imc-unknown-none-elf`  | *    |      | Bare RISC-V (RV32IMC ISA)              |
| `riscv64gc-unknown-linux-gnu`  | ✓    | ✓    | RISC-V Linux (kernel 4.20, glibc 2.29) |
| `riscv64gc-unknown-none-elf`   | *    |      | Bare RISC-V (RV64IMAFDC ISA)           |
| `riscv64imac-unknown-none-elf` | *    |      | Bare RISC-V (RV64IMAC ISA)             |

- std:
  - ✓ indicates the full standard library is available.
  - \* indicates the target only supports [`no_std`](https://rust-embedded.github.io/book/intro/no-std.html) development.
  - ? indicates the standard library support is unknown or a work-in-progress.
- host: A ✓ indicates that `rustc` and `cargo` can run on the host platform.

TEST ON: 

```
            .-/+oossssoo+/-.               root@Ubuntu-riscv64
        `:+ssssssssssssssssss+:`           -------------------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04 LTS focal riscv64
    .ossssssssssssssssssdMMMNysssso.       Host: riscv-virtio,qemu
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 5.5.0-dirty
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 1 min
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: bash 5.0.16
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Terminal: /dev/ttyS0
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   CPU: (4)
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Memory: 63MiB / 3948MiB
ossyNMMMNyMMhsssssssssssssshmmmhssssssso
+sssshhhyNMMNyssssssssssssyNMMMysssssss+
.ssssssssdMMMNhsssssssssshNMMMdssssssss.
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.
```

All test passed

> [see here for more information](https://doc.rust-lang.org/nightly/rustc/platform-support.html)



### step0 交叉编译程序 至 RISC-V 测试



rust 已经对 RISC-V64 拥有完全支持 
[tested libs](https://www.notion.so/dustbin/step0-risc-v-efb2cec3f8374068b9e2e002c675edee)

####　Error
[nom](https://github.com/Geal/nom)

>nom is a parser combinators library written in Rust. Its goal is to provide tools to build safe parsers without compromising the speed or memory consumption. To that end, it uses extensively Rust's strong typing and memory safety to produce fast and correct parsers, and provides functions, macros and traits to abstract most of the error prone plumbing.
>

原因分析--

--> 		https://github.com/gnzlbg/jemallocator/tree/HEAD/jemalloc-sys

-->		http://jemalloc.net/

-->		支持 jemalloc on risc-v

Some lib can't build on RISC-V

[run on debian](https://buildd.debian.org/status/package.php?a=amd64,armhf,ppc64el,riscv64,sparc64&p=rust-core-arch,rust-caps,rust-http,rust-jemalloc-sys,rust-nix,rust-nodrop-union)

 



###　step1 写CI 自动化测试



[github action](https://docs.github.com/cn/actions)

在 GitHub Actions 的仓库中自动化、自定义和执行软件开发工作流程。 您可以发现、创建和共享操作以执行您喜欢的任何作业（包括 CI/CD），并将操作合并到完全自定义的工作流程中。

Work on

```yml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
env:
  CARGO_TERM_COLOR: always
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup riscv-gnu-toolchain 
        uses: colinaaa/setup-riscv-gnu-toolchain@v1.0.2

      - name: test for-riscv-gnu-toolchain
        run: ls /opt/hostedtoolcache/riscv-gnu-toolchain/2020.08-1/x64/bin/
      
      - name: Docker Setup QEMU
        uses: docker/setup-qemu-action@v1.0.1
        with:
            platforms: riscv64
      
      - name: Rustup add
        run: rustup target add riscv64gc-unknown-linux-gnu
      
      - name: Rustup test
        run:  rustup target list
      
      - name: Rust Build
        run: cargo build --verbose

      - name: Run tests
        run: cargo test --verbose
        
```

​	[like this website](https://github.com/RISC-V-for-Rust-toolchain/Main-CI-Template/runs/2065516741?check_suite_focus=true)



### step2 RISC-V 上标准库的支持情况



[Standard library support for riscv64gc-unknown-linux-gnu](https://github.com/rust-lang/rust/pull/66899/)

####　编译Rust 工具链

| depend        |                version                |
| ------------- | :-----------------------------------: |
| python3       |              up-to-date               |
| ninja         |             newest on apt             |
| cmake         |                3.16.3                 |
| time-to-build |                40mins                 |
| rustc         | 1.51.0-nightly (23adf9fd8 2021-02-05) |



Rust 是⼀个⾃举的编译器，需要通过旧的编译器来构建最新的版本。所以⼀般是分阶段来完成：

1. `Stage0` 阶段。下载最新`beta`版的编译器，这些`x.py`会⾃动完成。你也可以通过修改配置⽂件来使⽤其他版本的Rust。
2. `Stage1` 阶段，使⽤`Stage0`阶段下载的`beta`版编译器来编译从`Git`仓库⾥下载的代码。最终⽣成`Stage1`版编译器。但是为了对其优化，还需要进⾏下⼀阶段。
3. `Stage2`，⽤`Stage1`版编译器继续对源码进⾏编译，以便⽣成Stage2版编译器。

理论上，`Stage1`和`Stage2`编译器在功能上是相同的，但实际上还有些细微的差别。



on host

````shell
test result: ok. 1129 passed; 0 failed; 20 ignored; 0 measured; 0 
filtered out; finished in 87.32s
finished in 104.623 seconds

Build completed successfully in 0:01:47
```

on risc-v

![image-20210312145346995](C:\Users\matri\AppData\Roaming\Typora\typora-user-images\image-20210312145346995.png)

### step3 在 RISC-V上自举

#### rustc



`Stage2`

`riscv64gc-unknown-linux-gnu` 是Stage2 的编译器 也就是说

- Official binary releases are provided for the platform.
- Automated building is set up, but may not be running tests.
- Landing changes to the `rust-lang/rust` repository's master branch is gated on platforms **building**. For some platforms only the standard library is compiled, but for others `rustc` and `cargo` are too

#### other toolchain



| tools |   build   | test | run？（can be cross-compiled) |
| ----- | :--- | :---- | ----- |
| cargo | √ | All passed | √ |
| cargo-miri | X |  | √ |
| fmt | √ | All passed | √ |



###　stepx 嵌入式 rust实验

#### 嵌入式现状

[嵌入式的rust](https://rustmagazine.github.io/rust_magazine_2021/chapter_1/embedded_rust.html#%E5%B5%8C%E5%85%A5%E5%BC%8F%E9%A2%86%E5%9F%9F%E7%9A%84rust%E8%AF%AD%E8%A8%80)

[rcore](https://rcore.gitbook.io/rust-os-docs/)

#### [Longan-nano](https://github.com/riscv-rust/longan-nano)





````rust
// os/src/main.rs
#![no_std]
#![no_main]

mod lang_items;

// os/src/lang_items.rs
use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
````



````rust
Text::new(" Hello Rust! ", Point::new(42, 0))
        .into_styled(style)
        .draw(&mut lcd)
        .unwrap();
````

![img](http://funwithsoftware.org/images/2020-longan-nano-ferris.jpg)





## rustc 对 RISC-V 指令集 的支持程度



###　abi

[doc here](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_target/spec/riscv_base/fn.unsupported_abis.html)

````rust
// All the calling conventions trigger an assertion(Unsupported calling
// convention) in llvm on RISCV
pub fn unsupported_abis() -> Vec<Abi> {
    vec![
        Abi::Cdecl,
        Abi::Stdcall,
        Abi::Fastcall,
        Abi::Vectorcall,
        Abi::Thiscall,
        Abi::Aapcs,
        Abi::Win64,
        Abi::SysV64,
        Abi::PtxKernel,
        Abi::Msp430Interrupt,
        Abi::X86Interrupt,
        Abi::AmdGpuKernel,
    ]
}
````

```rust
use crate::spec::{CodeModel, Target, TargetOptions};

pub fn target() -> Target {
    Target {
        llvm_target: "riscv64-unknown-linux-gnu".to_string(),
        pointer_width: 64,
        data_layout: "e-m:e-p:64:64-i64:64-i128:128-n64-S128".to_string(),
        arch: "riscv64".to_string(),
        options: TargetOptions {
            unsupported_abis: super::riscv_base::unsupported_abis(),
            code_model: Some(CodeModel::Medium),
            cpu: "generic-rv64".to_string(),
            features: "+m,+a,+f,+d,+c".to_string(),
            llvm_abiname: "lp64d".to_string(),
            max_atomic_width: Some(64),
            ..super::linux_gnu_base::opts()
        },
    }
}
```







