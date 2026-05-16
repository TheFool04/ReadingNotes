# Game Programming Patterns Practice

本仓库用于配合阅读《Game Programming Patterns》，通过纯 C++ 小练习和 UE5 C++ 实践，理解游戏开发中常见设计模式的核心思想、适用场景与工程取舍。

## 目标

- 用纯 C++ 练习设计模式本身，避免被引擎机制干扰
- 将关键模式迁移到 UE5 中，验证其在 Gameplay 系统中的实际价值
- 建立面向游戏客户端、Gameplay、工具链与作品集项目的工程设计意识
- 为后续 UE5 C++ Gameplay Demo 积累可复用的设计经验

## 仓库结构

```text
GameProgrammingPatterns/
  book/
  book-zh/
  cpp-practice/               # 纯 C++ 设计模式练习
    command/
    observer/
    state/
    prototype/
    object_pool/
    event_queue/

  ue-pattern-sandbox/          # UE5 C++ 设计模式实践工程
    Source/
    Content/

  docs/                        # 笔记、复盘、设计说明
    notes
```

## 练习方式

### 纯 C++ Practice

每个设计模式对应一个独立小练习，重点关注：

1. 不使用该模式时的问题
2. 使用该模式后的结构变化
3. 模式提供的扩展点
4. 该模式是否适合迁移到 UE5

### UE5 Pattern Sandbox

只迁移和 Gameplay 强相关的模式，例如：

```text
Command     -> 输入命令 / 技能释放
State       -> 角色战斗状态
Observer    -> UI 与 Gameplay 数据解耦
Component   -> HealthComponent / AbilityComponent
Object Pool -> 子弹、特效、伤害数字复用
Flyweight   -> 技能配置、敌人配置共享
```

## 原则

- 先理解模式动机，再考虑引擎落地
- 纯 C++ 练概念，UE5 练工程应用
- 小题目优先，不做大而全的系统
- 不为使用设计模式而使用设计模式
- 只为明确的变化做设计，避免过度抽象

## 技术环境

```text
Language: C++17 / UE C++
Build: CMake / Unreal Build Tool
Engine: Unreal Engine 5
IDE: JetBrains Rider
Platform: Windows
```

## 最终用途

本仓库不是完整游戏项目，而是设计模式训练场和 UE5 Gameplay 技术沙盒。

其中有价值的设计会被沉淀到后续的 UE5 C++ Gameplay Demo 中，用于作品集、技术复盘和长期游戏客户端能力建设。
