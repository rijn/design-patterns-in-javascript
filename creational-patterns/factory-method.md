# 工厂（Factory Method）

工厂模式是最常见的创建型设计模式，你可以在各个地方见到它。

\[sample\]

在工厂模式中，创建对象时，我们无需调用任何创建的逻辑，仅需要通过一个创建的接口，即可获得一个新的对象。例如我们有一个生产电脑的工厂，我们仅需要通知工厂型号，工厂就可以生产出一台电脑。而消费者只要拿到电脑即可开机，无需做其他的操作或添加别的设备。

```js
var laptop = LaptopFactory.makeLaptop("XYZ");
laptop.powerOn();
```

这种方法的优劣显而易见。无需顾及Factory中对象创建的逻辑；可复用，一个工厂可以提供产品给多个消费者，且保证产品的质量；扩展性好，可以建造多个工厂，或一个工厂建立多个生产线生产不同的产品。劣势在于，每个对象都需要一次创建，且属性、方法不共用内存，对于内存的开销较大。每增加一个产品，就需要建立一个工厂，对于类的依赖较大。当工厂很多时，系统容易变得非常复杂。

## 实现

我们还是用生产电脑来举例。

### 假如没有工厂

假如没有工厂，消费者就需要自己动手。意味着消费者需要自己购入零件，自己装配。对于一台电脑，我们要做的事情有：

```js
// 定义一个模型，假装它是我们的电脑
var laptop = new Laptop();

// 给这个电脑装上CPU，内存，硬盘
laptop.cpu = new CPU('i7-2600');
laptop.memory = new Memory(8);
laptop.hardDisk = new HardDisk(512);

// 给这个电脑安装系统
laptop = Windows.installOn(laptop);

// 安装开关和LED，使这台电脑可以开关机
laptop.power = false;
laptop.powerOn = function () { this.power = true; };
laptop.powerOff = function () { this.power = false; };
```

试想一下，当我们有多个消费者都需要购买这台电脑时，或是消费者对于电脑有不同的需求，你就需要new多次，或是修改CPU的配置，这样的实现就十分繁琐。

### 简单工厂

下面我们来尝试用JavaScript实现一个简单的工厂。请看下面的例子：

```js
// 定义一个电脑工厂
var LaptopFactory = {
    // 工厂可以生产Laptop
    makeLaptop: function () {
        var laptop = new Laptop();

        laptop.cpu = new CPU('i7-2600');
        laptop.memory = new Memory(8);
        laptop.hardDisk = new HardDisk(512);

        laptop = Windows.installOn(laptop);

        laptop.power = false;
        laptop.powerOn = function () { this.power = true; };
        laptop.powerOff = function () { this.power = false; };

        return laptop;
    }
};
```

这个电脑工厂现在只做了一件事情，就是生产笔记本，并且只能生产一种型号。但我们已经可以初见工厂的效力：

```js
// 想要一台电脑？向工厂下订单！
var laptop = LaptopFactory.makeLaptop();
```

扩展生产线，让工厂可以生产更多型号的电脑！

```js
var LaptopFactory = {
    makeLaptop: function ({ cpu = 'i7-2600', memory = 8, hardDisk = 512 }) {
        var laptop = new Laptop();

        laptop.cpu = new CPU(cpu);
        laptop.memory = new Memory(memory);
        laptop.hardDisk = new HardDisk(hardDisk);

        laptop = Windows.installOn(laptop);

        laptop.power = false;cpu
        laptop.powerOn = function () { this.power = true; };
        laptop.powerOff = function () { this.power = false; };

        return laptop;
    }
};

// 我想要更多内存
var laptop = LaptopFactory.makeLaptop({ memory: 16 });
```

在这里的简单工厂模式，又叫静态工厂方法（Static Factory Method），是最简单的工厂方法。实际上，这个方法违背了我们的开闭原则。使用简单模式的电脑工厂，只能生产同类型的产品，即电脑。如果我们修改工厂产生Laptop的基类，想让这个工厂生产更多不同类型的产品，比如台式机，那么不仅需要修改生产的方法，也需要修改消费者下订单的代码。

### 工厂方法

想让工厂生产台式机？让我们再加一个抽象层，即建立多个车间。

