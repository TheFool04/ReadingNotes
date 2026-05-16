# 组件模式

**所属章节：解耦模式**

## 意图

*允许单一实体横跨多个领域，同时保持各领域之间互不耦合。*

## 动机

假设我们正在开发一款平台跳跃游戏。意大利水管工的题材已经有人做了，所以我们的主角换成一位丹麦<span name="baker">面包师</span>——比约恩（Bjørn）。理所当然地，我们需要一个类来表示这位亲切的糕点厨师，并把他在游戏中所做的一切都塞进去。

> **注：** 这种绝妙的游戏创意正是我选择当程序员而非设计师的原因。

由于玩家要操控他，就需要读取手柄输入并将其转化为移动。当然，他还要和关卡互动，所以要加上物理和碰撞。完成这些之后，他还得出现在屏幕上，于是再塞进动画和渲染。他大概还会发出一些声音。

等等，这已经失控了。软件架构101告诉我们：程序中不同的领域应当彼此隔离。写文字处理软件时，打印功能的代码不应该受到文件加载与保存代码的影响。游戏和企业应用的领域不同，但这条规则依然适用。

我们希望AI、物理、渲染、音效等领域尽可能互不知晓彼此，但现在我们把所有这些都塞进了同一个类。我们见过这条路通向哪里：一个5000行的垃圾场源文件，大到只有团队里最勇猛的忍者程序员才敢踏入其中。

这对少数能驾驭它的人来说是铁饭碗，但对其余的人来说却是地狱。一个如此庞大的类意味着，即使看似微不足道的改动也可能牵一发而动全身。很快，这个类积攒 *bug* 的速度就会超过积攒*功能*的速度。

### 戈尔迪之结

比规模问题更糟糕的是<span name="coupling">耦合</span>问题。游戏中所有不同的系统被缠绕成一团巨大的代码乱麻，就像这样：

```cpp
if (collidingWithFloor() && (getRenderState() != INVISIBLE))
{
  playSound(HIT_FLOOR);
}
```

任何试图修改此类代码的程序员都需要了解物理、图形和音效方面的知识，才能确保自己没有弄坏任何东西。

> **注：** 这种耦合在*任何*游戏中都很糟糕，但在使用并发的现代游戏中尤为严重。在多核硬件上，让代码同时在多个线程上运行至关重要。一种常见的做法是按领域边界来划分游戏的线程——AI运行在一个核心上，音效运行在另一个核心上，渲染运行在第三个核心上，依此类推。
>
> 一旦这样做，就必须让各领域保持解耦，以避免死锁或其他并发 bug。如果一个类有一个 `UpdateSounds()` 方法必须从某个线程调用，而另一个 `RenderGraphics()` 方法必须从另一个线程调用，这种设计简直是在主动招惹这类 bug。

这两个问题相互叠加：这个类涉及太多领域，每个程序员都不得不在其中工作，但它又如此庞大，让人痛苦不堪。如果情况足够糟糕，程序员会开始在代码库的其他地方打各种补丁，只为了远离 `Bjorn` 这个类所变成的那团乱麻。

### 斩断乱麻

我们可以像亚历山大大帝那样——用剑来解决。我们将把这个单体 `Bjorn` 类沿领域边界切分为独立的部分。举例来说，我们把处理用户输入的所有代码移入一个独立的 `InputComponent` 类，然后让 `Bjorn` 持有这个组件的实例。对 `Bjorn` 所涉及的每个领域，我们都重复这个过程。

完成之后，我们已将几乎所有内容从 `Bjorn` 中移出。剩下的只是一个将各组件粘合在一起的薄壳。我们通过简单地将其拆分成多个更小的类来解决了巨型类的问题，但我们所收获的不止于此。

### 松散的末端

现在，我们的组件类彼此解耦了。尽管 `Bjorn` 同时持有 `PhysicsComponent` 和 `GraphicsComponent`，但这两个组件对彼此毫不知情。这意味着负责物理的人可以修改自己的组件，而无需了解图形方面的任何知识，反之亦然。

实际上，这些组件之间还是需要有*一些*交互的。例如，AI组件可能需要告诉物理组件比约恩想要移动到哪里。不过，我们可以将这种通信限定在确实需要对话的组件之间，而不是把所有组件一股脑地扔进同一个围栏里。

### 重新绑合

这种设计的另一个优点是：这些<span name="inheritance">组件</span>现在成为了可复用的包。到目前为止，我们一直专注于面包师，但让我们考虑游戏世界中的其他几类对象。*装饰物*是玩家能看见但无法互动的东西：灌木、碎片及其他视觉细节。*道具*类似于装饰物，但可以被触碰：箱子、巨石和树木。*区域*是装饰物的反面——不可见但可以互动。它们在触发过场动画（比如比约恩进入某个区域时）方面很有用。

> **注：** 面向对象编程刚兴起时，继承是其工具箱里最闪亮的那把锤子。它被视为终极代码复用利器，程序员们频繁挥舞着它。此后，我们用惨痛的经历认识到，这把锤子确实很重。继承有其用武之地，但对于简单的代码复用来说，它往往过于笨重。
>
> 取而代之的是，软件设计中的流行趋势是尽可能使用组合而非继承。与其让两个类通过继承同一个父类来共享代码，不如让它们各自*持有同一个类的实例*。

现在，想象一下，如果不使用组件，我们会如何为这些类构建继承层次结构。初步方案可能如下：

*（图示：类图展示 Zone 包含碰撞代码并继承自 GameObject；Decoration 同样继承自 GameObject 并包含渲染代码；Prop 继承自 Zone，但包含冗余的渲染代码。）*

我们有一个包含位置和朝向等通用内容的基类 `GameObject`。`Zone` 继承自它并添加碰撞检测。类似地，`Decoration` 继承自 `GameObject` 并添加渲染。`Prop` 继承自 `Zone`，以便复用碰撞代码。然而，`Prop` 无法*同时*继承 `Decoration` 来复用*渲染*代码，否则就会陷入<span name="diamond">致命菱形</span>问题。

> **注：** "致命菱形"出现在具有多重继承的类层次结构中，即存在两条不同路径指向同一基类。这带来的麻烦有些超出本书的讨论范围，但要知道，"致命"二字可不是凭空而来的。

我们可以反过来让 `Prop` 继承自 `Decoration`，但这样就得重复*碰撞*代码。无论如何，如果不诉诸多重继承，就没有干净的方式在需要它们的类之间复用碰撞和渲染代码。唯一的出路是把所有东西都推到 `GameObject` 里，但这样 `Zone` 就白白浪费内存去存储它根本不需要的渲染数据，`Decoration` 也同样为物理数据付出无谓的内存代价。

现在，用组件来试试。那些<span name="menu">子类</span>完全消失了。取而代之的是，我们只有一个 `GameObject` 类和两个组件类：`PhysicsComponent` 和 `GraphicsComponent`。装饰物就是一个带有 `GraphicsComponent` 但没有 `PhysicsComponent` 的 `GameObject`。区域恰恰相反，道具则两者兼备。没有代码重复，没有多重继承，三个类就搞定了原本需要四个类的问题。

> **注：** 餐厅菜单是个很好的类比。如果每个实体都是一个单体类，就好比你只能点套餐。我们需要为每种可能的功能*组合*单独定义一个类。要满足所有顾客，就需要几十种套餐。
>
> 组件就是点菜式（à la carte）用餐——每位顾客可以只选自己想要的菜，菜单列出了所有可供选择的菜品。

组件本质上是对象的即插即用机制。它们让我们能够通过将不同的可复用组件对象插入实体的"插槽"来构建出行为丰富的复杂实体。把它想象成软件版的变形金刚（Voltron）。

## 模式结构

**单一实体横跨多个领域**。为了保持各领域的隔离，每个领域的代码被放入其专属的**<span name="component">组件</span>类**中。实体被简化为一个**组件容器**。

> **注：** "组件"和"对象"一样，是编程中那种什么都能意味、又什么都不意味的词。因此，它被用来描述几个不同的概念。在企业软件中，有一种"组件"设计模式，描述的是通过网络通信的解耦服务。
>
> 我尝试为游戏中这个不相干的模式另觅新名，但"组件"似乎是最通用的叫法。既然设计模式旨在记录已有实践，我没有自创新词的资格。所以，跟随 XNA（微软推出的游戏开发框架）、Delta3D（一款开源游戏引擎）等框架的脚步，就叫"组件"吧。

## 使用时机

组件最常见于定义游戏实体的核心类中，但在其他地方也可能派上用场。当以下任何一条成立时，这个模式就值得一用：

- 你有一个涉及多个领域的类，希望这些领域之间保持解耦。
- 一个类变得越来越庞大且难以维护。
- 你希望能够定义多种共享不同能力的对象，但继承不够精准，无法让你选择性地复用想要的部分。

## 注意事项

相比直接编写一个类并在其中写代码，组件模式引入了相当多的复杂性。每个概念上的"对象"都变成了一簇对象，需要被实例化、初始化并正确地连接在一起。不同组件之间的通信更具挑战性，对内存布局的控制也更为复杂。

对于大型代码库来说，这种复杂性所换来的解耦和代码复用可能物有所值，但在使用这个模式之前，请务必确认自己不是在对一个并不存在的问题过度设计。

使用组件的另一个后果是：做任何事都往往需要多一层间接调用。给定一个容器对象，你首先要获取你想要的组件，*然后*才能执行所需的操作。在<span name="perf">性能敏感</span>的内循环中，这种指针追踪可能导致性能下降。

> **注：** 这枚硬币还有另一面。组件模式往往能*提升*性能和缓存一致性。组件让你更容易运用[数据局部性](24-data-locality.md)模式，将数据按照CPU所期望的顺序来组织。

## 示例代码

对我来说，写这本书最大的挑战之一，就是如何将每个模式单独呈现。许多设计模式的存在是为了容纳那些本身并不属于该模式的代码。为了提炼出模式的精髓，我尽量删去这些内容，但这就有点像在不展示任何衣物的情况下解释如何整理衣橱。

组件模式尤其难以处理。不看各领域的一些代码，你就无法真正感受到它——所以我不得不比预期多展示一些比约恩的代码。这个模式的核心其实只是组件*类*本身，但类中的代码有助于阐明它们的用途。这是伪代码——它调用了其他未在此展示的类——但应该能让你感受到我们的目标是什么。

### 单体类

为了更清楚地说明该模式是如何应用的，我们从展示一个单体<span name="cat">`Bjorn`</span>类开始——它实现了我们需要的一切，但*不*使用这个模式：

> **注：** 我应该指出，在代码库中直接使用角色的真实姓名通常是个坏主意。市场部有个讨厌的习惯：在发布前几天突然要求改名。"焦点测试显示，11到15岁的男性对'Bjørn'反应消极，请改用'Sven'。"
>
> 这就是为什么许多软件项目使用内部代号的原因——当然，另一个原因是，告诉别人你在做"大电猫"（Big Electric Cat）要比"Photoshop下一个版本"酷多了。

```cpp
class Bjorn
{
public:
  Bjorn()
  : velocity_(0),
    x_(0), y_(0)
  {}

  void update(World& world, Graphics& graphics);

private:
  static const int WALK_ACCELERATION = 1;

  int velocity_;
  int x_, y_;

  Volume volume_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

`Bjorn` 有一个 `update()` 方法，每帧由游戏调用：

```cpp
void Bjorn::update(World& world, Graphics& graphics)
{
  // Apply user input to hero's velocity.
  switch (Controller::getJoystickDirection())
  {
    case DIR_LEFT:
      velocity_ -= WALK_ACCELERATION;
      break;

    case DIR_RIGHT:
      velocity_ += WALK_ACCELERATION;
      break;
  }

  // Modify position by velocity.
  x_ += velocity_;
  world.resolveCollision(volume_, x_, y_, velocity_);

  // Draw the appropriate sprite.
  Sprite* sprite = &spriteStand_;
  if (velocity_ < 0)
  {
    sprite = &spriteWalkLeft_;
  }
  else if (velocity_ > 0)
  {
    sprite = &spriteWalkRight_;
  }

  graphics.draw(*sprite, x_, y_);
}
```

它读取摇杆输入来决定如何加速这位面包师，然后用物理引擎解算新的位置，最后将比约恩绘制到屏幕上。

这里的示例实现极为简单：没有重力，没有动画，也没有让角色操控有趣的其他几十个细节。即便如此，我们已经能看出：有一个函数是团队里好几个人都可能需要花时间修改的，而它已经开始变得有些杂乱了。把这个想象成放大到一千行，你就能体会它会有多痛苦。

### 拆分出一个领域

从一个领域开始，我们从 `Bjorn` 中抽出一部分，放入一个独立的组件类。从第一个被处理的领域——输入——开始。`Bjorn` 做的第一件事是读取用户输入并据此调整速度。让我们把这个逻辑移出到一个独立的类中：

```cpp
class InputComponent
{
public:
  void update(Bjorn& bjorn)
  {
    switch (Controller::getJoystickDirection())
    {
      case DIR_LEFT:
        bjorn.velocity -= WALK_ACCELERATION;
        break;

      case DIR_RIGHT:
        bjorn.velocity += WALK_ACCELERATION;
        break;
    }
  }

private:
  static const int WALK_ACCELERATION = 1;
};
```

很简单。我们把 `Bjorn` 的 `update()` 方法的第一部分放入了这个类。`Bjorn` 的改动同样直观：

```cpp
class Bjorn
{
public:
  int velocity;
  int x, y;

  void update(World& world, Graphics& graphics)
  {
    input_.update(*this);

    // Modify position by velocity.
    x += velocity;
    world.resolveCollision(volume_, x, y, velocity);

    // Draw the appropriate sprite.
    Sprite* sprite = &spriteStand_;
    if (velocity < 0)
    {
      sprite = &spriteWalkLeft_;
    }
    else if (velocity > 0)
    {
      sprite = &spriteWalkRight_;
    }

    graphics.draw(*sprite, x, y);
  }

private:
  InputComponent input_;

  Volume volume_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

`Bjorn` 现在持有一个 `InputComponent` 对象。以前他直接在 `update()` 方法中处理用户输入，现在他将其委托给组件：

```cpp
input_.update(*this);
```

我们才刚刚开始，但已经消除了一些耦合——主 `Bjorn` 类不再引用 `Controller`。这在后面会很有用。

### 拆分剩余部分

现在，让我们对物理和图形代码做同样的剪切-粘贴操作。下面是新的 `PhysicsComponent`（物理组件）：

```cpp
class PhysicsComponent
{
public:
  void update(Bjorn& bjorn, World& world)
  {
    bjorn.x += bjorn.velocity;
    world.resolveCollision(volume_,
        bjorn.x, bjorn.y, bjorn.velocity);
  }

private:
  Volume volume_;
};
```

除了将物理*行为*从主 `Bjorn` 类移出之外，你可以看到我们也将*数据*一并移走了：`Volume` 对象现在归组件所有。

最后，渲染代码现在住在这里：

```cpp
class GraphicsComponent
{
public:
  void update(Bjorn& bjorn, Graphics& graphics)
  {
    Sprite* sprite = &spriteStand_;
    if (bjorn.velocity < 0)
    {
      sprite = &spriteWalkLeft_;
    }
    else if (bjorn.velocity > 0)
    {
      sprite = &spriteWalkRight_;
    }

    graphics.draw(*sprite, bjorn.x, bjorn.y);
  }

private:
  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

我们几乎把所有东西都掏空了，那这位朴实的糕点厨师还剩下什么？没多少了：

```cpp
class Bjorn
{
public:
  int velocity;
  int x, y;

  void update(World& world, Graphics& graphics)
  {
    input_.update(*this);
    physics_.update(*this, world);
    graphics_.update(*this, graphics);
  }

private:
  InputComponent input_;
  PhysicsComponent physics_;
  GraphicsComponent graphics_;
};
```

`Bjorn` 类现在基本上只做两件事：持有真正定义它的那组组件，以及持有跨领域共享的状态。位置和速度仍留在核心 `Bjorn` 类中，有两个原因：第一，它们是"泛领域"状态——几乎每个组件都会用到，所以如果真的想把它们下移，并不清楚*哪个*组件应该拥有它们。

第二，也是更重要的一点：这为各组件之间提供了一种无需彼此耦合的通信方式。让我们看看能否利用这一点。

### 机器人比约恩

至此，我们已将行为推入各独立的组件类，但尚未将行为*抽象化*。`Bjorn` 仍然知道定义其行为的具体类是哪些。让我们改变这一点。

我们将把处理用户输入的组件隐藏在一个接口后面，把 `InputComponent` 变成抽象基类：

```cpp
class InputComponent
{
public:
  virtual ~InputComponent() {}
  virtual void update(Bjorn& bjorn) = 0;
};
```

然后，将现有的用户输入处理代码下移到实现该接口的类中：

```cpp
class PlayerInputComponent : public InputComponent
{
public:
  virtual void update(Bjorn& bjorn)
  {
    switch (Controller::getJoystickDirection())
    {
      case DIR_LEFT:
        bjorn.velocity -= WALK_ACCELERATION;
        break;

      case DIR_RIGHT:
        bjorn.velocity += WALK_ACCELERATION;
        break;
    }
  }

private:
  static const int WALK_ACCELERATION = 1;
};
```

将 `Bjorn` 改为持有输入组件的指针，而非内联实例：

```cpp
class Bjorn
{
public:
  int velocity;
  int x, y;

  Bjorn(InputComponent* input)
  : input_(input)
  {}

  void update(World& world, Graphics& graphics)
  {
    input_->update(*this);
    physics_.update(*this, world);
    graphics_.update(*this, graphics);
  }

private:
  InputComponent* input_;
  PhysicsComponent physics_;
  GraphicsComponent graphics_;
};
```

现在，当我们实例化 `Bjorn` 时，可以传入一个输入组件给它使用，像这样：

```cpp
Bjorn* bjorn = new Bjorn(new PlayerInputComponent());
```

这个实例可以是任何实现了抽象 `InputComponent` 接口的具体类型。我们为此付出了一定代价——`update()` 现在是一个虚函数调用，速度稍慢。我们得到了什么作为回报？

大多数主机要求游戏支持"演示模式"。如果玩家在主菜单呆着不动，游戏会自动开始运行，由电脑代替玩家操作。这能防止游戏把主菜单烧进电视屏幕，也让游戏在商店展示机上更好看。

将输入组件类隐藏在接口后面，让我们得以实现这一需求。我们已经有了在正常游戏时使用的具体 `PlayerInputComponent`。现在来创建另一个：

```cpp
class DemoInputComponent : public InputComponent
{
public:
  virtual void update(Bjorn& bjorn)
  {
    // AI to automatically control Bjorn...
  }
};
```

当游戏进入演示模式时，我们不再像之前那样构造比约恩，而是用我们的新组件来组装他：

```cpp
Bjorn* bjorn = new Bjorn(new DemoInputComponent());
```

就这样，仅仅通过替换一个组件，我们就拥有了一个功能完整的、用于演示模式的AI控制玩家。比约恩的所有其他代码都得以复用——物理和图形甚至不知道有什么区别。或许我有点奇怪，但正是这类东西让我<span name="coffee">每天早上</span>有起床的动力。

> **注：** 还有咖啡。香气氤氲的热咖啡。

### 彻底告别比约恩？

现在看看我们的 `Bjorn` 类，你会发现它其实没什么"比约恩"的特质——它不过是一个组件袋。事实上，它看起来很像一个不错的"游戏对象"基类候选，可以用于游戏中的*每一个*对象。我们只需传入*所有*组件，就能像弗兰肯斯坦博士一样，通过挑选和组合零件来构建任何种类的对象。

让我们把剩下的两个具体组件——物理和图形——也隐藏在接口后面，就像我们对输入所做的那样：

```cpp
class PhysicsComponent
{
public:
  virtual ~PhysicsComponent() {}
  virtual void update(GameObject& obj, World& world) = 0;
};

class GraphicsComponent
{
public:
  virtual ~GraphicsComponent() {}
  virtual void update(GameObject& obj, Graphics& graphics) = 0;
};
```

然后把 `Bjorn` 重命名为<span name="id">通用的</span> `GameObject` 类，并让它使用这些接口：

```cpp
class GameObject
{
public:
  int velocity;
  int x, y;

  GameObject(InputComponent* input,
             PhysicsComponent* physics,
             GraphicsComponent* graphics)
  : input_(input),
    physics_(physics),
    graphics_(graphics)
  {}

  void update(World& world, Graphics& graphics)
  {
    input_->update(*this);
    physics_->update(*this, world);
    graphics_->update(*this, graphics);
  }

private:
  InputComponent* input_;
  PhysicsComponent* physics_;
  GraphicsComponent* graphics_;
};
```

> **注：** 有些组件系统走得更远。游戏实体不再是一个包含组件的 `GameObject`，而仅仅是一个 ID——一个数字。然后，你维护各自独立的组件集合，其中每个组件都知道自己所属实体的 ID。
>
> 这种[实体组件系统（Entity Component System）](http://en.wikipedia.org/wiki/Entity_component_system)将组件解耦推向极致，让你可以在实体甚至不知情的情况下为其添加新组件。[数据局部性](24-data-locality.md)章节有更多详细介绍。

现有的具体类将被重命名，并实现这些接口：

```cpp
class BjornPhysicsComponent : public PhysicsComponent
{
public:
  virtual void update(GameObject& obj, World& world)
  {
    // Physics code...
  }
};

class BjornGraphicsComponent : public GraphicsComponent
{
public:
  virtual void update(GameObject& obj, Graphics& graphics)
  {
    // Graphics code...
  }
};
```

现在，我们可以在不为他实际定义一个类的情况下，构建一个拥有比约恩全部原始行为的对象，就像这样：

<span name="factory"></span>

```cpp
GameObject* createBjorn()
{
  return new GameObject(new PlayerInputComponent(),
                        new BjornPhysicsComponent(),
                        new BjornGraphicsComponent());
}
```

> **注：** 这个 `createBjorn()` 函数，当然，是四人帮（GoF）经典的[工厂方法模式（Factory Method）](http://c2.com/cgi/wiki?FactoryMethod)的一个示例。

通过定义其他函数，用不同的组件来实例化 `GameObject`，我们就能创建游戏所需的各种不同对象。

## 设计决策

使用这个模式时，你需要回答的最重要的设计问题是："我需要哪些组件？"这个答案取决于你的游戏的需求和类型。引擎越大越复杂，你可能就越想把组件切分得更细。

除此之外，还有一些更具体的选项值得考量：

### 对象如何获得其组件？

一旦把单体对象拆分成几个独立的组件，我们就必须决定谁来把这些部件重新组合起来。

**如果由对象自行创建其组件：**

- *确保对象始终拥有它所需的组件。* 你不必担心有人忘记为对象连接正确的组件而破坏游戏。容器对象自己会处理这一切。
- *更难重新配置对象。* 这个模式的强大之处之一在于，只需重新组合组件就能构建新类型的对象。如果对象总是以同一套硬编码组件连接自身，我们就没有利用到这种灵活性。

**如果由外部代码提供组件：**

- *对象变得更灵活。* 我们可以通过给对象提供不同的组件来彻底改变其行为。推到极致，我们的对象成为一个通用的组件容器，可以反复用于不同目的。
- *对象可以与具体组件类型解耦。* 如果允许外部代码传入组件，那么很可能也允许它传入*派生*的组件类型。这时，对象只需知道组件的*接口*，而不必了解具体类型本身。这能带来封装良好的架构。

### 组件之间如何通信？

完全解耦、各自孤立运行的组件是个美好的理想，但在实践中行不通。这些组件同属一个*对象*，意味着它们是更大整体的一部分，需要协调配合。这就意味着通信。

那么组件之间如何对话呢？有几种选项，但与本书中大多数设计"备选方案"不同，这些选项并非互斥——你很可能在设计中同时支持不止一种。

**通过修改容器对象的状态：**

- *保持组件的解耦。* 当 `InputComponent` 设置比约恩的速度、`PhysicsComponent` 随后使用它时，这两个组件根本不知道对方的存在。对它们来说，比约恩的速度仿佛是通过巫术改变的。
- *需要将组件间共享的信息上移到容器对象中。* 往往存在只有部分组件才需要的状态。例如，动画组件和渲染组件可能需要共享图形专属的信息。把这些信息提升到容器对象，让*所有*组件都能访问，会污染对象类的设计。更糟糕的是，如果我们用同一个容器对象类配合不同的组件配置，可能会浪费内存去存储那些没有任何组件需要的状态。如果我们把某些渲染专属数据推入容器对象，任何不可见的对象都会白白为这些数据消耗内存。
- *通信是隐式的，且依赖于组件的处理顺序。* 在示例代码中，原始的单体 `update()` 方法有着精心设计的操作顺序：用户输入修改速度，速度被物理代码用来修改位置，位置又被渲染代码用来将比约恩画在正确的位置。当我们把代码拆分到组件中时，我们小心地保留了这个操作顺序。如果没有做到，就会引入<span name="pure">难以追踪的</span>微妙 bug。例如，如果我们*先*更新图形组件，就会错误地把比约恩渲染在*上一帧*的位置，而不是当前帧的位置。如果想象有更多的组件和更多的代码，就能体会避免此类 bug 有多难。

  > **注：** 这种大量代码读写同一份数据的共享可变状态（shared mutable state）出了名地难以正确处理。这也是学术界花大量精力研究像 Haskell（一种纯函数式编程语言）这样完全没有可变状态的纯函数式语言的重要原因之一。

**通过直接相互引用：**

这里的思路是，需要对话的组件直接持有彼此的引用，完全不必经过容器对象。

假设我们想让比约恩跳跃。图形代码需要知道该绘制跳跃精灵还是普通精灵，它可以通过询问物理引擎他当前是否在地面上来判断。一种简单的做法是让图形组件直接了解物理组件：

```cpp
class BjornGraphicsComponent
{
public:
  BjornGraphicsComponent(BjornPhysicsComponent* physics)
  : physics_(physics)
  {}

  void Update(GameObject& obj, Graphics& graphics)
  {
    Sprite* sprite;
    if (!physics_->isOnGround())
    {
      sprite = &spriteJump_;
    }
    else
    {
      // Existing graphics code...
    }

    graphics.draw(*sprite, obj.x, obj.y);
  }

private:
  BjornPhysicsComponent* physics_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
  Sprite spriteJump_;
};
```

构造比约恩的 `GraphicsComponent` 时，我们会给它传入对应 `PhysicsComponent` 的引用。

- *简单而快速。* 通信是从一个对象到另一个对象的直接方法调用。组件可以调用它所引用的组件所支持的任何方法，完全自由。
- *两个组件之间紧密耦合。* 这是"完全自由"的代价。我们基本上退回到了单体类的老路，尽管不像最初的单个类那么糟糕——至少，我们把耦合限定在了确实需要互动的组件对之间。

**通过发送消息：**

这是最复杂的方案。我们可以在容器对象中内建一个小型消息系统，让组件彼此广播信息。

下面是一种可能的实现。首先定义一个所有组件都会实现的 `Component` 基接口：

```cpp
class Component
{
public:
  virtual ~Component() {}
  virtual void receive(int message) = 0;
};
```

它有一个 `receive()` 方法，组件类通过实现它来监听传入的消息。这里我们只用一个 `int` 来标识消息，但更完整的实现可以为消息附加额外数据。

然后，我们为容器对象添加一个发送消息的方法：

```cpp
class ContainerObject
{
public:
  void send(int message)
  {
    for (int i = 0; i < MAX_COMPONENTS; i++)
    {
      if (components_[i] != NULL)
      {
        components_[i]->receive(message);
      }
    }
  }

private:
  static const int MAX_COMPONENTS = 10;
  Component* components_[MAX_COMPONENTS];
};
```

<span name="queue">现在</span>，如果一个组件能访问其容器，就可以向容器发送消息，容器会将消息广播给所有包含的组件（包括发送消息的那个组件本身——注意不要陷入反馈死循环！）这带来了如下结果：

> **注：** 如果你真想搞点花样，甚至可以让这个消息系统把消息*排队*，稍后再投递。更多内容请参阅[事件队列](21-event-queue.md)。

- *兄弟组件之间保持解耦。* 通过<span name="mediator">经由</span>父容器对象中转——类似于共享状态方案——我们确保各组件仍然互不知晓。在这个系统中，它们之间唯一的耦合就是消息值本身。

  > **注：** 四人帮把这种方式称为[中介者模式（Mediator）](http://c2.com/cgi-bin/wiki?MediatorPattern)——两个或多个对象通过中间对象间接通信。在本例中，容器对象本身就是中介者。

- *容器对象保持简单。* 与使用共享状态（容器对象本身拥有并感知组件所用数据）不同，这里容器只是盲目地转发消息。这对于让两个组件在彼此之间传递非常领域专属的信息时很有用，而不会让这些信息蔓延到容器对象中。

不出所料，这里没有唯一正确的答案。你最终很可能会混合使用这几种方式。共享状态适用于真正基础的内容——那些可以理所当然地认为每个对象都拥有的东西，例如位置和大小。

有些领域彼此独立，却又密切相关。想想动画与渲染、用户输入与AI、物理与碰撞。如果你为这些组合中的每一半都设置了独立的组件，直接让它们了解另一半往往是最简便的做法。

消息适用于"次要"的通信。它的即发即忘特性非常适合这类场景：当物理组件发消息说对象发生了碰撞，音效组件于是播放一个声音。

一如既往，我建议从简单的开始，在需要时再添加更多通信路径。

## 参考

- Unity（游戏引擎）的核心 [`GameObject`](http://docs.unity3d.com/Documentation/Manual/GameObjects.html) 类完全围绕[组件](http://docs.unity3d.com/Manual/UsingComponents.html)设计。
- 开源引擎 Delta3D（一款开源游戏引擎）有一个 `GameActor` 基类，通过贴切命名的 `ActorComponent` 基类实现了这个模式。
- 微软的 XNA（微软推出的游戏开发框架）游戏框架带有一个核心 `Game` 类，它持有一组 `GameComponent` 对象。我们的例子在单个游戏实体层面使用组件，而 XNA 在主游戏对象本身的层面实现了这个模式，但目的是一样的。
- 这个模式与四人帮的[策略模式（Strategy）](http://c2.com/cgi-bin/wiki?StrategyPattern)有几分相似。两种模式都是将对象部分行为抽取出来，委托给一个独立的从属对象。区别在于，策略模式中的"策略"对象通常是无状态的——它封装了一个算法，但没有数据。它定义了对象*如何*行为，而不是*是什么*。组件则更有自己的主见：它们往往持有描述对象并有助于定义其实际身份的状态。不过，界限可能会模糊。你可能有一些不需要任何本地状态的组件。在这种情况下，你完全可以在多个容器对象之间共用同一个组件*实例*，此时它的行为就更接近策略了。
