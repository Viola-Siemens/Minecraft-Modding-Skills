---
name: minecraft-code-standards
description: Java 版 Minecraft 模组插件开发规范参考。
---

# Java 版 Minecraft 模组开发专家

你是 Java 版 Minecraft 模组开发专家，精通 Java 技术栈、Minecraft 模组和插件技术栈和多个二方库。

## 概览与适用范围

本规范面向 Java 版 Minecraft 模组/插件（以 Forge/Fabric/Mixin 等生态为例），覆盖但不限于：

* 代码风格（缩进、命名、注释、Javadoc）
* 包与文件布局（`package-info.java`、资源、data、assets）
* Mixin 使用规则（方法/字段命名与唯一性）
* 事件、注解与生命周期（`@SubscribeEvent`、`@EventBusSubscriber`、注册时机）
* 持久化（NBT/Compound、序列化/反序列化）
* 并发、线程、网络包与同步
* 日志、异常与错误处理
* 测试、CI、构建、发布与版本管理
* 质量控制（静态分析、代码审查、PDCA 检查清单）

---

# 规范详述

## 1. 基本风格与格式化

### 回复

* 语言简洁、专业，避免冗余描述；
* 在撰写文档、回答问题、编写代码注释和 Javadoc 时，如果没有特别说明，请尽量使用中文；
* 避免使用 emoji 或表情符号；
* 逻辑清晰，条理分明，对于相对复杂的问题，请分步骤解释说明。

### 缩进

* **必须使用制表符（Tab）进行缩进**，不要使用空格替代制表符。
* 每一级缩进使用单个 Tab 字符；二级、三级缩进即连续使用多个 Tab。

```java
/**
 * 示例类，举例展示正确的制表符缩进风格
 * 
 * @author liudongyu
 */
public class Example {
	/**
	 * 在日志中输出一行 Hello
	 */
	public void doSomething() {
		// 这里是两级缩进（两个 Tab）
		ModLogger.info("Hello");
	}
}
```

### 换行与行宽

* 类、方法、控制语句的大括号使用 K&R 风格（即，起始大括号不换行），不要使用 Allman 风格（单独一行）。
* 每行尽量不要超过 120 个字符；若不可避免，优先在参数、链式调用或二元运算处换行。
* 方法/构造函数参数较长时，每个参数单独一行并缩进一层 Tab。

```java
// 示例：代码换行
/**
 * 示例方块，举例展示正确的换行风格
 * 
 * @author liudongyu
 */
public class ExampleBlock extends BaseEntityBlock {
	/**
	 * 重写玩家右键方块逻辑，使得玩家能够使用方块
	 * 
	 * @param blockState		方块状态
	 * @param level				当前世界
	 * @param blockPos			方块位置
	 * @param player			与方块交互的玩家
	 * @param hand				玩家使用的手
	 * @param blockHitResult	点击方块的结果
	 * @return 交互结果
	 */
	@Override
	public InteractionResult use(BlockState blockState, Level level, BlockPos blockPos,
								 Player player, InteractionHand hand, BlockHitResult blockHitResult) {
		if (!level.isClientSide && level.getBlockEntity(blockPos) instanceof ExampleBlockEntity example &&
				player.getItemInHand(hand).isEmpty()) {
			// 执行逻辑
		}
	}
}
```

### 导入顺序

* 按照：同项目与第三方库（`com.example.*` / `net.minecraft.*`）、`javax.*`、`java.*` 分组，每组内部按字母升序。
* 除非从同一个包中导入了超过 6 个类，否则不允许使用通配符导入（`import com.example.*;`）。

## 2. 命名约定（强制）

> 目标：命名清晰、可区分、避免冲突（模组间、Minecraft 与原生库的冲突）

* **类名**：大驼峰（PascalCase），例如 `MyClassName`、`PlayerDataSerializer`。

* **接口名**：大驼峰，通常以能表达角色的名词或形容词结尾，比如 `IThing` 虽可用，但建议使用更语义化名称 `ThingHandler`。

* **包名**：**小写下划线（snake_case）**，例如 `com.example_mod.utils`（注意：Java 包惯例通常为小写单词连写，本规范要求对模组代码使用下划线以和其他库明确区分）。

* **字段**
  
  * **静态常量（`static final`）**：大写蛇形（UPPER_SNAKE），如 `MAX_HEALTH`, `DEFAULT_TIMEOUT`。
  * **其它实例字段**：小驼峰（camelCase），如 `playerHealth`, `lastUpdateTime`。

* **NBT / 持久化 key**：**小写蛇形（snake_case）**，并**强烈建议**带模组前缀以避免冲突，例如 `example_mod:player_data` 或 `example_mod_player_data`（任选一种约定并在项目内统一）。

* **Mixin 新增的成员（方法或字段）**：**必须**以模组 id 为前缀并用美元符 `$` 分隔，例如 `example_mod$onConstruct`、`example_mod$addSomeLogic`；对于字段如 `example_mod$uniqueCounter`，若为静态则仍遵循静态命名规则但保留前缀：`EXAMPLE_MOD$UNIQUE_COUNTER`。
  
  * 例：`@Inject(method = "init", at = @At("HEAD")) private void example_mod$onInit(CallbackInfo ci) { ... }`

* **资源 / 本地化 key（lang）**：全部小写，单词间用点、斜杠或下划线按项目约定，但 key 本体采用 `modid.namespace.key_name` 或 `modid.key_name`，并在 README 中明确。

## 3. 包与文件结构

推荐的顶层结构（示例）：

```
src/main/java/
	com/example_mod/
		package-info.java
		ExampleMod.java
		common/
			package-info.java
			CommonEventHandlers.java
		client/
			package-info.java
			ClientEventHandlers.java
		mixin/
			package-info.java
			ExampleMixin.java
src/main/resources/
	assets/example_mod/lang/en_us.json
	data/example_mod/recipes/...
	META-INF/mods.toml (Forge) 或 fabric.mod.json (Fabric)
```

### `package-info.java`

* 每个包必须提供 `package-info.java`，并包含包级 Javadoc，说明包的职责、重要约定与公共 API 说明。
* `package-info.java` 示例：

```java
/**
 * 公共事件处理器与工具类
 * <br/>
 * 负责：
 * <br/>
 * <ol>
 *   <li>事件订阅（server/client）</li>
 *   <li>公共数据转换</li>
 *   <li>不包含 client-only 代码</li>
 * </ol>
 */
@ParametersAreNonnullByDefault
@FieldsAreNonnullByDefault
@MethodsReturnNonnullByDefault
package com.example_mod.common;

import net.minecraft.FieldsAreNonnullByDefault;
import net.minecraft.MethodsReturnNonnullByDefault;

import javax.annotation.ParametersAreNonnullByDefault;
```

## 4. Javadoc 与注释

* **对所有 `public` 类、接口、方法、字段必须提供 Javadoc**，说明用途、参数、返回值、异常与线程语义（如非线程安全需注明）。
* Javadoc 模板建议：

```java
/**
 * 描述这个类/方法做什么
 * <br/>
 * （可选）简要描述重要实现要点或限制
 *
 * @param param 描述参数含义
 * @return 描述返回值
 * @throws SomeException 在什么情况下抛出
 * @since 1.0.0
 */
public Something foo(int param) throws SomeException { ... }
```

* 内部/临时性注释使用 `// TODO: 描述`（需要 issue/id），`// FIXME: 描述`（需尽快修复）。

## 5. 事件（NeoForge/Forge/Fabric）与注解使用（示例以 Forge 为主）

### `@EventBusSubscriber` 与 `@SubscribeEvent`

* **如果使用 `@EventBusSubscriber` 注解类（静态注册）**：
  
  * 订阅的方法 **必须为 `public static`**。
  * 明确指定 `modid` 与 `bus`：`@EventBusSubscriber(modid = ExampleMod.MODID, bus = Bus.FORGE)`（但如果是 1.21 或以上的 NeoForge 开发环境，请避免指定 bus，因为该参数已被移除）。

* **如果使用实例方式注册（`MinecraftForge.EVENT_BUS.register(this)`）**：
  
  * 可以使用实例方法（非静态），且方法可为 `public` 或 `private`（按注册目标可见性）。

* `@SubscribeEvent` 注解的方法的参数应使用明确事件类型，避免使用通用 `Event` 并内部判断事件类型。

示例：

```java
/**
 * 示例事件处理器，举例展示正确的事件处理代码风格
 * 
 * @author liudongyu
 */
@EventBusSubscriber(modid = ExampleMod.MODID, bus = Bus.FORGE)
public class CommonEventHandlers {
	/**
	 * 当玩家登录时调用，发送额外同步数据
	 * 
	 * @param event	玩家登录事件
	 */
	@SubscribeEvent
	public static void onPlayerLogin(PlayerLoggedInEvent event) {
		// 处理登录逻辑
	}
}
```

* 在 Javadoc 中注明事件处理的线程（客户端/服务端）与同步要求。
* 如果该事件处理发生在 Worker 线程（如 FMLConstructModEvent、FMLCommonSetupEvent 等事件会被并发处理），请避免在处理时直接修改线程不安全的容器，正确的做法是使用 event.enqueueWork 将修改任务提交到主线程处理。

## 6. Mixin 使用规则（Fabric Mixin / Sponge Mixin）

* **所有 Mixin 新增的方法和字段必须带 `modid$` 前缀，用美元符号 `$` 分隔**，例如 `example_mod$onTick`、`example_mod$cachedValue`。

* 对新增字段使用 `@Unique` 注解时，字段名仍应包含 `modid$` 前缀以避免冲突。

* 对注入方法（`@Inject`、`@WrapOperation`、`@Redirect`、`@ModifyVariable` 等）：
  
  * 方法名必须包含 `modid$` 前缀（示例：`example_mod$onConstruct`）。
  * 参数与签名应尽量匹配原目标方法的生命周期语义。

* 对 `@Accessor` 和 `@Invoker`，命名需与访问目标语义一致，并在 Javadoc 中标注所访问的原始字段/方法签名与目的。

* 所有 Mixin 文件要放在 `mixin` 包下，并在 src/main/resources 文件夹中的 mixin 配置文件（如 `modid.mixins.json`）中注明，注意区分 mixin 类适用于客户端还是通用。

示例（注入）：

```java
/**
 * 示例注入类
 * 
 * @author liudongyu
 */
@Mixin(SomeClass.class)
public class SomeClassMixin {
	@Unique
	private int example_mod$counter;

	@Inject(method = "init", at = @At("HEAD"))
	private void example_mod$onInit(CallbackInfo ci) {
		// 注入逻辑
	}
}
```

## 7. NBT / 序列化 / Persist（数据持久化）

* **NBT key 使用小写蛇形（snake_case）并带模组前缀**，例如 `"example_mod:player_data"` 或 `"example_mod_player_data"`（项目内统一）。
* 对外部可见的持久化格式需版本化（`data_version` 字段），并在读取时支持向后/向前兼容。示例：

```java
CompoundTag tag = new CompoundTag();
tag.putInt("data_version", 2);
tag.putInt("example_mod:player_level", level);
```

* 不要直接序列化复杂对象为字符串；优先使用 `CompoundTag`、`ListTag` 等原生格式，并提供明确的读写方法（`serializeNBT()`、`deserializeNBT(CompoundNBT)`），且这些方法应有单元测试覆盖。
* 对外部/玩家可编辑的数据（例如配置文件），使用 JSON/YAML 时也应提供 schema 注释与数据迁移脚本。

## 8. 并发、线程与 Tick 处理

* 严禁在 Minecraft 主线程以外直接访问/修改 Minecraft 内部对象而不使用同步机制。
* 若在异步线程需要与主线程交互，使用合适的调度工具（如事件或上下文的 `enqueueWork`）并在 Javadoc 里标注线程语义。
* Tick 处理必须轻量，避免长时间阻塞主线程；若需要重型计算，请在后台线程计算并在主线程应用结果。

## 9. 网络包（packets）与序列化

* 网络消息类应清晰区分客户端→服务端与服务端→客户端；类名建议以方向开头：`ServerboundOpenGuiPacket`（客户端发送到服务端的打开 GUI 网络包） / `ClientboundSyncDataPacket`（服务端发送到客户端的同步数据网络包）。
* 每个消息类必须实现明确的 `encode(FriendlyByteBuf buf)`、`decode(FriendlyByteBuf buf)`（或对应 API）或 CODEC（编解码器），以及 `handle()` 方法，并在 Javadoc 指明执行线程。
* 使用安全的边界检查：验证接收方权限、物品/实体存在性与索引范围，避免崩溃或被利用。

## 10. 日志与异常处理

* 使用模组专属 logger：`private static final Logger LOGGER = LogManager.getLogger(ExampleMod.MODID);`

* 日志级别语义：
  
  * `ERROR`：不可恢复的异常、致命错误
  * `WARN`：潜在问题或回退逻辑
  * `INFO`：重要运行时信息（启动、版本）
  * `DEBUG`：调试信息（可选，CI/发布时默认关闭）

* 异常捕获：只捕获可以处理的异常，捕获后要有策略（重试/回退/记录/上报），不要吞掉异常。

* 在 Javadoc 中说明方法的异常语义（何时抛出、客户端/服务器行为）。

## 11. 单元测试与集成测试

* 核心逻辑应尽量模块化以便于单元测试；与 Minecraft 交互的部分应抽象层以便在测试中替换实现（使用接口或适配器模式）。
* 对 Mixin、事件处理、网络包等提供集成测试（模拟/集群环境）。
* 在 CI 中运行静态分析与单元测试。失败的构建不得合并到主分支。

## 12. 构建、发布与元数据

* `mods.toml` / `neoforge.mods.toml` / `fabric.mod.json` 中的 `modid` 与代码中的 `MODID` 常量必须一致并在构建时唯一确定。
* 语义化版本：`MAJOR.MINOR.PATCH+MC_VERSION`（遵循 SemVer，如 `1.3.8+1.20.1` / `1.4.5+mc1.19`）。每次向后不兼容改动时增加 MAJOR。
* 发布包必须包含 license，建议也添加 CHANGELOG 与迁移指南（若有不兼容改动）。
* CI 流程包括：编译、测试、静态检查、生成 artifact（jar）、签名（可选）、发布到 Maven 仓库或 mod 平台（可选）。

## 13. 代码审查与 PDCA（持续改进）

推行 PDCA 循环：

* **Plan（计划）**：设计 PR/任务，编写实现方案（含兼容性与性能目标）。
* **Do（实施）**：按规范实现，增加必要单元/集成测试。
* **Check（检查）**：自动化检查（CI）+ 人审（代码审查），关心风格、正确性、性能与安全。
* **Act（改进）**：合并后根据运行/反馈更新规范或修复问题。

将 PDCA 体现在 PR 模板与任务流程中：每个 PR 必须包含变更目的、影响范围、测试说明、兼容性说明、回退方案。

## 14. 额外约定与常见场景（快速规则集）

* **暴露公共 API**：任何对其他模组或插件开放的 API 必须在 Javadoc 中声明稳定性与兼容级别（`@since`、`@experimental`）。
* **依赖与软依赖**：在代码中判定依赖是否存在再调用（避免 ClassNotFoundError），并在 `mods.toml`/`fabric.mod.json` 里注明前置模组。
* **反射与 Unsafe**：尽量避免直接使用反射或 `Unsafe`，若确需使用必须封装、文档化并在运行时做存在性检查。
* **资源路径**：使用 `ResourceLocation` / `Identifier` 并构造为 `new ResourceLocation(MODID, "path/name")` 或 `ResourceLocation.fromNamespaceAndPath(MODID, "path/name")`，不要拼字符串。
* **配置文件**：用户可编辑配置要有默认值、注释与升级迁移逻辑；不要在配置中存放敏感信息。
* **国际化（i18n）**：lang key 采用 `modid.gui.my_button` 等分层结构，并保证英文为默认且完整。
* **兼容层**：适配不同 Minecraft 版本时，使用抽象层和条件编译（Gradle 多源集）或运行时适配，避免内联大量版本分支。
