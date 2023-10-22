# [ArkCompiler 中的 IR](https://github.com/changxvv/Blog/issues/6)

看了一圈 ArkCompiler 相关仓库的文档，其中 runtime_core 仓库的文档是最全的，介绍了 ArkCompiler 的整体设计。

ArkCompiler 中的字节码文件后缀为 abc，整个编译器运行时有一个名字叫做 Panda，所以说 Arkcompiler 的字节码又被成为 Panda 字节码。

文档文件树如下：

```text
├── compiler
│   └── docs                                 // 讲了各种各样在ir阶段的optimizations
└── docs
    ├── 2022-08-18-isa-changelog.md
    ├── PBC2IR.md                            // Panda Bytecode转IR的对照表
    ├── aot.md                               // 介绍arkcompiler的Ahead Of Time Compilation
    ├── assembly_format.md                   // Panda字节码的格式
    ├── bc_verification                      // 介绍字节码验证器
    ├── cfi_directives.md                    // cfi指令用法
    ├── code_metainfo.md                     // 介绍编译后的代码元信息
    ├── coding-style.md
    ├── debugger-vscode-communication.md
    ├── design-of-interpreter.md             // 解释器设计
    ├── file_format.md                       // 二进制文件格式
    ├── glossary.md
    ├── ir_format.md                         // 介绍ir的设计和格式
    ├── irtoc.md                             // 将手工创建的IR编译为目标代码
    ├── memory-management-SW-requirements.md
    ├── memory-management.md                 // 内存管理
    ├── on-stack-replacement.md              // 介绍OSR
    ├── panda-runtime.md                     // 字节码文档目录
    ├── rationale-for-bytecode.md            // 字节码设计原理
    ├── runtime-class.md
    ├── runtime-compiled_code-interaction.md // 编译后代码和运行时的交互
    ├── runtime-debug-api.md
    └── tracing.md
```

这里重点说一下 ir_format。里面提到了 ArkCompiler 使用了 Reverse Post Order (RPO)、Domtree、Linear Order 以及多种 Analysis 技术。并且提到了 Panda IR 处理的两个步骤：

1. Panda bytecode 被转换为 *high-level* 的指令，进行部分架构独立的优化；
2. 指令被进一步分为几个 *low-level* 指令（接近汇编），再进行优化。

第一个步骤是为了快速生成 IR，第二个步骤目标是 ARM64，然后进行 ARM 的特殊优化。文档也提到了支持 Panda IR 和 LLVM IR 之间的转换，这样就能允许进行 LLVM 的特殊优化。

其次，在文档中，ArkCompiler 选择了 CFG (Control Flow Graph) with SSA (Static Single Assignment) 的架构而非 Sea-of-Nodes，原因是前者更简单通用并且更轻量。而 IR 的指令则使用类继承的方式来设计实现（`Inst` 是基类）。

寄存器分配方面，目前使用的是消耗较小的 *Linear Scan* 算法，之后会考虑转向 *Graph-coloring* 算法，因为它能生成更高效的代码。

IR 转机器码方面，目前 ArkCompiler 使用的是标准 vixl 库，算是 ARM 的技术。之后可能会考虑自己实现这部分。

值得注意的是，最近 runtime_core 仓库中增加了一个 static_core 模块，里面对现有的 core 做了改动，文档也更丰富了。比如 compiler 部分新增加了几十种在字节码阶段的优化方法的文档（如 branch_elimination、loop_unrolling）；其次，在主文档内还添加了 Deoptimization、Cross-Value 等新功能的文档。

## Reference
- runtime_core 仓库：https://gitee.com/openharmony/arkcompiler_runtime_core