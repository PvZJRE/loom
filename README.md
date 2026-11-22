# loom

PvZ-Portable 工作区的模组工具链，以一个 Gradle 插件交付。发布在 **Gradle Plugin Portal**（`./gradlew publishPlugins`），模组用下面一行即可拿到它，无需先发布任何本地构件；`./gradlew build publishToMavenLocal` 也会把它放进 mavenLocal（工作区开发者与 Portal 尚未提供的开发版本）。

| | |
|---|---|
| 插件 id | `io.github.pvzjre.loom` |
| 构件 | `io.github.pvzjre:loom:1.0.16` |
| 标记 | `io.github.pvzjre.loom:io.github.pvzjre.loom.gradle.plugin:1.0.16` |
| Portal | https://plugins.gradle.org/plugin/io.github.pvzjre.loom |

插件以 Kotlin 编写（`src/main/kotlin/io/github/pvzjre/loom/`），为 Gradle 自身 JVM 编译（`jvmTarget` 17），POM 声明其编译所用的 `kotlin-stdlib`——Kotlin 插件不能把运行时留给碰巧装了谁的人。POM 上没有别的东西：Kotlin Gradle 插件是本项目的构建工具，不是插件或模组的依赖。

## 用法

```kotlin
plugins { id("io.github.pvzjre.loom") version "1.0.16" }

group = "io.github.pvzjre"
version = "1.0.0"

dependencies {
    pvz("io.github.pvzjre:pvz-portable:1.2.5")   // 本模组针对的游戏发行版
}
```

工具链供给的是**游戏本身**——发行包里的游戏 jar 与版本 JSON 指名的全部库（真正的 Fabric Loader、Mixin、ASM、SLF4J/Log4j 栈、LWJGL 及本机 natives），它们以 Maven 坐标抵达，因此模组编译所见的版本与启动器加载的完全一致。

模组用到的其他一切由模组自己的构建文件声明（Kotlin 插件与 stdlib、`fabric-language-kotlin`、API 聚合 jar 与 `include`、JUnit）——工具链不替模组做任何决定。

## 游戏发行版（`pvz`）

| 构建写法 | 发行版来源 |
|---|---|
| 不写 | `pvz("io.github.pvzjre:pvz-portable:1.2.5")`，`PvzToolchain` 里的版本 |
| `pvz("io.github.pvzjre:pvz-portable:<version>")` | 该版本 GitHub release 的资产 `pvz-portable-<version>.zip` |
| `pvz(files("<zip>"))` | 磁盘上的发行归档（如 `./gradlew distZip` 刚写出的产物） |
| `pvz(files("<directory>"))` | 解压后的发行目录，就地使用，不复制 |
| `pvz(files("<directory-of-zips>"))` | 发行目录：构建所对版本的 `pvz-portable-<version>.zip`（如 `PvZ-Portable/build/distributions`） |

获取与缓存：

- **Release**（两种坐标形式）：资产下载一次进 `$GRADLE_USER_HOME/caches/pvz/pvz-portable-<version>.zip`（旁边有 `.sha256` 标识），解包到模组项目的 `build/pvz/dist/`。缓存过的资产绝不重复下载（`--offline` 对用过的版本可用）；中断的下载以 `.part` 落盘、完成后移入正式名。
- **本地归档 / 发行目录**：不下载，解包方式相同。
- **解压发行目录**：就地使用，两个目录是输入而非复制。
- 发行标签是纯版本号，`v<version>` 作为第二次尝试。发行源用 `pvz.release.baseUrl` Gradle 属性切换（镜像或 `file:` / HTTP 目录）。
- 解包幂等且自检：jar 旁的标记文件记录归档（路径与 SHA-256）与解出的 jar（名字和大小），未变化的归档不再解包；重建的归档、另一个发行版或被手动替换的 jar 会触发全新解包——类路径永远不会少一个 jar。新解包以单次移动整体替换目录，不会与上一次混杂。

失败提示皆可读，每条都说明该做什么（发行资产 404、发行目录缺对应版本、`pvz()` 不是游戏坐标、pvz 只取一个发行版等）。

## 工具链供给什么

- 游戏本体与版本 JSON 指名的全部库，进编译/测试/运行/启动类路径；
- 真正 Fabric Loader 上的 `runClient` 开发运行；
- 模组 jar 的 mavenLocal 发布、JUnit 测试约定、Kotlin 2.4.21 工具链与 `fabric-language-kotlin`。

**不供给**：Kotlin 插件与 stdlib、语言模块、JUnit、API 聚合 jar——这些是模组自己的构建声明（工具链文档：API 是"模组自己的声明"，与任何其他项目的依赖一样）。

## 为什么不是 fabric-loom

本工作区的游戏不是 Minecraft：它的 provider、版本 JSON 和内容管线都是自己的。自研工具链按本游戏的发行形状实现同样的功能，而不携带 Minecraft 工具链的任何假设。

## 发布本插件

`./gradlew publishPlugins`（凭据来自 `gradle.properties` 的 `gradle.publish.key`/`secret`）。新版本先 `publishToMavenLocal` 验证，再发布到 Portal。

## 许可证

见 `LICENSE`。
