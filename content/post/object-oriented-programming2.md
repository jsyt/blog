---
title: "面向对象编程与 SOLID 原则（二）"
date: 2020-04-20T13:12:34+08:00
tags: ["Software Architecture"]
categories: ["Software Architecture"]
image: "https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/aswathy-n-hnG9DFkMLsI-unsplash.jpg"
---

在文章的 [第一部分](http://ytop.net/post/object-oriented-programming/)，我们主要讨论了面向对象与前两个 SOLID 原则，它们分别是单一职责原则和开闭原则。在这一部分，我们将讨论接下来的三个原则。

## 面向对象五大原则(SOLID)：

### 里氏替换原则（Liskov-Substituion Principle）

**子类可以替换父类并且出现在父类能够出现的任何地方**

在这个原则中父类应尽可能使用接口或者抽象类来实现。子类通过实现了父类接口，能够替父类的使用地方。通过这个原则，我们客户端在使用父类接口的时候，通过子类实现。

意思就是说我们依赖父类接口，在客户端声明一个父类接口，通过其子类来实现

这个时候就要求子类必须能够替换父类所出现的任何地方，这样做的好处就是，在根据新要求扩展父类接口的新子类的时候而不影响当前客户端的使用！

让我们看一个关于多个自行车的示例。Bike 基类如下：

```
class Bike {
  void pedal() {
    // pedal code
  }

  void steer() {
    // steering code
  }
  void handBrakeFront() {
    // hand braking front code
  }
  void handBrakeBack() {
    // hand braking back code
  }
}
```

山地自行车类 MountainBike 继承自基类 Bike （山地车有通过齿轮的机械原理调整档位的特性）：

```
class MountainBike extends Bike {
  void changeGear() {
    // change gear code
  }
}
```

MountainBike 类遵循了里氏替换原则，因为它能够被当作一个 Bike 类的对象使用。如果我们有一个自行车类型数组，并用 Bike 和 MountainBike 的实例对象来填充它，那么我们完全可以正确无误地调用 steer() 、pedal() 等 Bike 基类的所有方法。所以，我们可以在不经过特殊处理的情况下，把 MountainBike 类型的元素当作 Bike 类型的元素来使用。

现在想象一下，我们添加了一个名为 ClassicBike 的类，如下所示：

```
class ClassicBike extends Bike {
  void footBrake() {
    // foot braking code
  }
}
```

这个类代表了一种经典自行车，你可以通过向后踩踏板来进行制动。这种自行车没有手刹。基于此，如果我们有一个 ClassicBike 类型的元素混在了上述的自行车数组中，我们仍然能够无误地调用 steer 和 pedal 方法。但是，当我们尝试调用 handBrakeFront 或者 handBrakeBack 的时候，问题就暴露出来了。取决于具体的实现，调用这些方法可能导致系统崩溃或者什么也不会做。我们可以通过检查当前元素是否是 ClassicBike 的实例来解决这个问题：

```
foreach(var bike in bikes) {
  bike.pedal()
  bike.steer()

  if(bike is ClassicBike) {
    bike.footBrake()
  } else {
    bike.handBrakeFront()
    bike.handBrakeBack()
  }
}
```

如你所见，假如没有类似上面的类型判断，我们就不能再把一个 ClassicBike 的实例看作一个 Bike 实例了。这显然违背了里氏替换原则。有多种方法可以解决这个问题，当我们讨论到 SOLID 中的 I(接口隔离) 原则时，就会看到一个解决之道。遵循里氏替换原则的一个有趣的结果，就是你编写的代码将很难不符合开闭原则。

面向对象编程存在的一个问题就是我们常常忘记正在打交道的数据，以及处理这些数据和真实世界里对象的关系。现实生活中的有些事情无法在代码中直接建立模型，因此我们必须牢记：抽象本身并不神奇，底层的数据仅是数据而已（并不是一个真正的自行车）。


### 接口隔离原则(Interface-Segregation Principle)

多个专用接口优于一个单一的通用接口。
本原则是单一职责原则用于接口设计的自然结果。一个接口应该保证，实现该接口的实例对象可以只呈现为单一的角色；这样，当某个客户程序的要求发生变化，而迫使接口发生改变时，影响到其他客户程序的可能生性小。

为了减少接口的定义，将许多类似的方法都放在一个接口中，最后发现，维护和实现接口的时候花了太多精力，而接口所定义的操作相当于对客户端的一种承诺，这种承诺当然是越少越好，越精练越好，过多的承诺带来的就是你的大量精力和时间去维护！

我们应该保持接口短小，在实现时选择实现多个小接口而不是庞大的单个接口。我会再次使用自行车的例子，但是这次我用一个 Bike 接口而不是 Bike 基类：

```
interface Bike {
  void pedal()
  void steer()
  void handBrakeFront()
  void handBrakeBack()
}
```

MountainBike 类必须实现这个接口的所有方法：

```
class MountainBike implements Bike {
  override void pedal() {
    // pedal implementation
  }

  override void steer() {
    // steer implementation
  }

  override void handBrakeFront() {
    // front hand brake implementation
  }

  override void handBrakeBack() {
    // back hand brake implementation
  }

  void changeGear() {
    // change gear code
  }
}
```

目前尚好。对于有问题的带有脚刹功能的 ClassicBike 类，我们可以采用下面这种笨拙的实现：

```
class ClassicBike implements Bike {
  override pedal() {
    // pedal implementation
  }

  override steer() {
    // steer implementation
  }

  override handBrakeFront() {
    // no code or throw an exception
  }

  override handBrakeBack() {
    // no code or throw an exception
  }

  void brake() {
    // foot brake code
  }
}
```

在这个例子中，我们不得不重写手刹的两个方法，尽管不需要它们。正如前所述，我们打破了里氏替换原则。

比较好的一个做法就是重构这个接口：

```
interface Bike() {
  void pedal()
  void steer()
}

interface HandBrakeBike {
  void handBrakeFront()
  void handBrakeBack()
}

interface FootBrakeBike {
  void footBrake()
}

```

MountainBike 类将实现 Bike 和 HandBrakeBike 接口，如下所示：

```
class MountainBike implements Bike, HandBrakeBike {
  // same code as before
}
```

ClassicBike 将实现 Bike 和 FootBrakeBike ，如下所示：

```
class ClassicBike implements Bike, FootBrakeBike {
  override pedal() {
    // pedal implementation
  }
  override steer() {
    // steer implementation
  }

  override footBrake() {
    // code that handles foot braking
  }
}
```

接口隔离原则的优势之一就是我们可以在多个对象上组合匹配接口，这提高了我们代码的灵活性和模块化。

我们也可以有一个 MultipleGearsBike 接口，在它里面添加 changeGear() 方法。现在，我们就可以构建一个拥有脚刹和换挡功能的自行车了。


此外，我们的类现在也遵循了里氏替换原则，ClassicBike 与 MountainBike 都能够看作 Bike 而毫无问题了。如前所述，遵循里氏替换原则也有助于开闭原则的实现。


### 依赖倒置原则（Dependecy-Inversion Principle）

传统的结构化编程中，最上层的模块通常都要依赖下面的子模块来实现，也称为高层依赖低层！

而DIP原则就是要逆转这种依赖关系，让高层模块不要依赖低层模块。

抽象不应依赖于细节，细节应该依赖于抽象。细节依赖于抽象，就意味着后来的依赖于先前的，这是自然而然的重用之道。
 
我仍将使用自行车的示例来尝试给你解释这个原则。首选看下这个 Bike 接口：

```
interface Bike {
  void pedal()
  void backPedal()
}
```

MountainBike 和 ClassicBike 这两个类实现了上面的接口：

```
// 山地车
class MountainBike implements Bike {
  override void pedal() {
    // complex code that computes the inner workings of what happens
    // when pedalling on a mountain bike, which includes taking into
    // account the gear in which the bike currently is.
  }

  override void backPedal() {
    // complex code that computes what happens when we back pedal
    // on a mountain bike, which is that you pedal in the wrong
    // direction with no discernible effect on the bike
  }
}

// 传统自行车
class ClassicBike implements Bike {
  override void pedal() {
    // the same as for the mountain bike with the distinction that
    // there is a single gear on a classic bike
  }

  override void backPedal() {
    // complex code that actually triggers the brake function on the
    // bike
  }
}
```

正如你所看到的，踩脚踏板（pedal）会让自行车向前行驶，但是山地车 MountainBike 因为有多个齿轮，所以它的 pedal 会更加复杂。另外，向后踩脚踏板（back pedal）时，山地车不会做任何事，而传统自行车 ClassicBike 则会触发刹车操作。

我之所以在每个方法的注释中都有提到“complex code”，是因为我想指出我们应该把上述代码移动到不同的模块中。我们这样做是为了简化自行车类以及遵循单一职责原则（自行车类不应该担起在你向前或向后踩脚踏板时究竟发生了什么的计算工作，它们应该处理有关自行车的更高级别的事情）。

为了做到这一点，我们将为每种类型的 pedalling 创建一些行为类。

```
class MountainBikePedalBehaviour {
  void pedal() {
    //complex code
  }
}

class MountainBikeBackPedalBehaviour {
  void backPedal() {
    // complex code
  }
}

class ClassicBikePedalBehaviour {
  void pedal() {
    // complex code
  }
}

class ClassicBikeBackPedalBehaviour {
  void backPedal() {
    // complex code
  }
}
```

然后像下面这样使用这些类：

```
// 山地车
class MountainBike implements Bike {
  override void pedal() {
    var pedalBehaviour = new MountainBikePedalBehaviour()
    pedalBehaviour.pedal()
  }

  override void backPedal() {
    var backPedalBehaviour = new MountainBikeBackPedalBehaviour()
    backPedalBehaviour.backPedal()
  }
}

// 传统自行车
class ClassicBike implements Bike {
  override void pedal() {
    var pedalBehaviour = new ClassicBikePedalBehaviour()
    pedalBehaviour.pedal()
  }

  override void backPedal() {
    var backPedalBehaviour = new ClassicBikeBackPedalBehaviour()
    backPedalBehaviour.backPedal()
  }
}
```

这个时候，我们可以很清楚地看到高级模块 MountainBike 依赖于某些具体的低级模块 MountainBikePedalBehaviour 和 MountainBikeBackPedalBehaviour。ClassicBike 以及它的低级模块同样如此。根据依赖倒置原则，高级模块和低级模块都应该依赖抽象。为此，我们需要以下接口：

```
interface PedalBehaviour {
  void pedal()
}

interface BackPedalBehaviour {
  void backPedal()
}
```

除了需要实现上面的接口外，行为类的代码与之前无异：

```
class MountainBikePedalBehaviour implements PedalBehaviour {
  override void pedal() {
    // same as before
  }
}
```

剩下的其他行为类同上。

现在我们需要一种方法将 PedalBehaviour 和 BackPedalBehaviour 传递给 MountainBike 和 ClassicBike 类。我们可以选择在构造方法、pedal() 、pedalBack() 中完成这件事。本例中，我们使用构造方法。

```
class MountainBike implements Bike {

  PedalBehaviour pedalBehaviour;
  BackPedalBehaviour backPedalBehaviour;

  public MountainBike(PedalBehaviour pedalBehaviour,
                      BackPedalBehaviour backPedalBehaviour) {
    this.pedalBehaviour = pedalBehaviour;
    this.backPedalBehaviour = backPedalBehaviour;
  }

  override void pedal() {
    pedalBehaviour.pedal();
  }

  override void backPedal() {
    backPedalBehaviour.backPedal();
  }
}
```

ClassicBike 类同上。

我们的高级模块（MountainBike 和 ClassicBike）不再依赖于具体的低级模块，而是依赖于抽象的 PedalBehaviour 和 BackPedalBehaviour。

在我们的例子中，我们应用的主模块可能看起来向下面这样：

```
class MainModule {
  MountainBike mountainBike;
  ClassicBike classicBike;
  MountainBikePedalBehaviour mountainBikePedalBehaviour;
  ClassicBikePedalBehaviour classicBikePedalBehaviour;
  MountainBikeBackPedalBehaviour mountainBikeBackPedalBehaviour;
  ClassicBikeBackPedalBehaviour classicBikeBackPedalBehaviour;

  public MainModule() {
    mountainBikePedalBehaviour = new MountainBikePedalBehaviour();
    mountainBikeBackPedalBehaviour =
      new MountainBikeBackPedalBehaviour();
    mountainBike = new MountainBike(mountainBikePedalBehaviour,
                     mountainBikeBackPedalBehaviour);

    classicBikePedalBehaviour = new ClassicBikePedalBehaviour();
    classicBikeBackPedalBehaviour =
      new ClassicBikeBackPedalBehaviour();
    classicBike = new ClassicBike(classicBikePedalBehaviour,
                    classicBikeBackPedalBehaviour);
  }

  public void pedalBikes() {
    mountainBike.pedal()
    classicBike.pedal()
  }

  public void backPedalBikes() {
    mountainBike.backPedal();
    classicBike.backPedal();
  }
}
```

可以看到，我们的 MainModule 依赖了具体的低级模块而不是抽象层。我们可以通过向构造方法中传递依赖来改善这种情况：

```
public MainModule(Bike mountainBike, Bike classicBike,
 PedalBehaviour mBikePB, BackPedalBehaviour mBikeBPB,
 PedalBehaviour cBikePB, BackPedalBehaviour cBikeBPB)...
```

现在，MainModule 部分依赖了抽象层，部分依赖了低级模块，这些低级模块也依赖了那些抽象层。所有这些模块之间的关系不再依赖于实现细节。

在我们到达应用程序中的最高模块之前，为了尽可能地延迟一个具体类的实例化，我们通常要靠依赖注入和实现了依赖注入的框架。我们可以将依赖注入视为帮助我们实现依赖倒置的工具。我们不断地向依赖链中传递依赖关系以避免具体类的实例化。

那么为什么要经历这一切呢？不依赖于具象的一个优点就是我们可以模拟一个类，从而使测试更容易进行。我们来看一个简单的例子。

```
interface Network {
  public String getServerResponse(URL serverURL);
}

class NetworkRequestHandler implements Network {
  override public String getServerResponse(URL serverURL) {
    // network code implementation
  }
}
```

假设我们还有一个 NetworkManager 类，它有一个公共方法，通过使用一个 Network 的实例返回服务器响应：

```
public String getResponse(Network networkRequestHandler, URL url) {
  return networkRequestHandler.getServerResponse(url)
}
```

因为这样的代码结构，我们可以测试代码如何处理来自服务器的“404”响应。为此，我们将创 NetworkRequestHandler的模拟版本。我们之所以可以这么做，是因为 NetworkManager 依赖于抽象层，即 Network，而不是某个具体的 NetworkRequestHandler。

```
class Mock404 implements Network {
  override public String getServerResponse(URL serverURL) {
    return "404"
  }
}
```

通过调用 getResponse 方法，传递 Mock404 类的实例，我们可以很容易地测试我们期望的行为。

除了易于测试，我们的应用在多变情景下也能应对自如。因为模块之间的关系是基于抽象的，我们可以更改具体模块的实现，而无需大范围地更改代码。

最后同样重要的是这会让事情变得更简单。如果你有留意自行车的示例，你会发现 MountainBike 和 ClassicBike 类非常相似。这就意味着我们不再需要单独的类了。我们可以创建一个简单的实现了 Bike 接口的类 GenericBike，然后山地车和传统自行车的实例化就像下面这样：

```
GenericBike mountainBike = new GenericBike(mbPedalB, mbBackPedalB);
GenericBike classicBike = new GenericBike(cbPedalB, cbBackPedalB);
```

我们减少了一半数量的具体自行车类的实现，这意味着我们的代码更容易管理。

## 总结

所有这些原则可能看起来有点矫枉过正，你可能会排斥它们。在很长的一段时间里，我和你一样。随着时间的推移，我开始逐渐把我的代码向增强可测试性和更易于维护的方向转变。渐渐地，我开始这样来思考事情：“如果只有一种方法可以把两个部分的内容分开，并将其放在不同的类中，以便我能……”。通常，答案是的确存在这样的一种方法，并且别人已经实现过了。大多数时候，这种方法都受到 SOLID 原则的启发。当然，紧迫的工期和其他现实生活中的因素可能会不允许你遵守所有这些原则。虽然很难 100% 实现 SOLID 原则，但是有比没有强吧。也许你可以尝试只在那些当需求变更时最容易受影响的部分遵守这些原则。你不必过分遵循它们，可以把这些原则视为你提高代码质量的指南。如果你不得不需要制作一个快速原型或者验证一个概念应用的可行性，那么你没有必要尽力去搭一个最佳架构。SOLID更像是一个长期策略，对于必须经得起时间考验的软件非常有用。



*参考链接：*
*https://en.wikipedia.org/wiki/Object-oriented_programming
https://medium.com/@razvan_57516/whats-the-deal-with-the-solid-principles-part-1-cdc4bad5d5b5
https://medium.com/@razvan_57516/whats-the-deal-with-the-solid-principles-part-2-8ff927c52d20
https://medium.com/@razvan_57516/whats-the-deal-with-the-solid-principles-part-3-47dbb85af396*