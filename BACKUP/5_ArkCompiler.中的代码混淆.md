# [ArkCompiler 中的代码混淆](https://github.com/changxvv/Blog/issues/5)

## 现有的混淆
在 DevEco 现有的最新版本里，采用的代码混淆是使用的是开源的 [uglify](https://github.com/mishoo/UglifyJS)。

```json5
{
	"apiType": "stageMode",
	"buildOption": {
	    "artifactType": "obfuscation" // 在项目设置文件中开启混淆
	}
}
```

在 DevEco 中调用的代码位于 https://gitee.com/openharmony/developtools_ace_ets2bundle/blob/master/compiler/uglify-source.js。

## Arkguard
Arkguard 是华为新开发的针对 Javascript 和 Typescript 的源码混淆工具。（我猜名称应该是模仿 Java 的开源混淆工具 ProGuard。）

在文档中，Arkguard 只能用于 Stage 模型 (不支持 FA 模型)，并且 Arkguard 只提供名称混淆的能力（理由是因为其它混淆能力会劣化性能）。

混淆的名称有：
- 参数名和局部变量名
- 顶层作用域的名称
- 属性名称

后两者默认关闭，因为默认打开可能会导致运行时错误。

项目源代码结构：

```text
src
├── ArkObfuscator.ts          // Entry，对文件进行代码混淆
├── IObfuscator.ts            // 定义混淆接口
├── cli                       // 调用ArkObfuscator，创建了混淆命令SecHarmony
├── common                    // 提取目标文件中的API信息
├── configs                   // 混淆的参数设置
├── generator                 // 主文件NameFactory，生成不同类型的混淆名称
├── transformers              // 被ArkObfuscator调用，进行代码转换和混淆
│   ├── TransformPlugin.ts    // 接口定义
│   ├── TransformerManager.ts // 根据混淆参数创建针对性的TransformerFactory进行代码混淆
│   ├── layout                // 处理代码布局，有三个TransformerFactory，分别用于禁用控制台调用、禁用Hilog调用和简化代码
│   └── rename                // 名称混淆，有三个TransformerFactory，分别用于重命名标识符、重命名属性、展开属性简写
└── utils
```

## Reference
- Arkguard 仓库及文档：https://gitee.com/openharmony/arkcompiler_ets_frontend/tree/master/arkguard