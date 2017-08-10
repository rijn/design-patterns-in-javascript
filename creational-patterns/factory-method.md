# 工厂（Factory Method）

工厂模式是最常见的创建型设计模式，你可以在各个地方见到它。

\[sample\]

在工厂模式中，创建对象时，我们无需调用任何创建的逻辑，仅需要通过一个创建的接口，即可获得一个新的对象。它让你的代码变得更加干净利落。例如我们有一个生产电脑的工厂，我们仅需要通知工厂型号，工厂就可以生产出一台电脑。而消费者只要拿到电脑即可开机，无需处理组装时的逻辑，也无需添加别的设备。

```js
var laptop = LaptopFactory.makeLaptop("XYZ");
laptop.powerOn();
```

这种方法的优劣显而易见。无需顾及Factory中对象创建的逻辑；可复用，一个工厂可以提供产品给多个消费者，且保证产品的质量；扩展性好，可以建造多个工厂，或一个工厂建立多个生产线生产不同的产品。劣势在于，每个对象都需要一次创建，且属性、方法不共用内存，对于内存的开销较大。每增加一个产品，就需要建立一个工厂，对于类的依赖较大。当工厂很多时，系统容易变得非常复杂。

## 假如没有工厂

我们还是用生产电脑来举例。假如没有工厂，消费者就需要自己动手。意味着消费者需要自己购入零件，自己装配。对于一台笔记本，我们要做的事情有：

```js
const CPU = function () {};
const Memory = function () {};
const HardDisk = function () {};
const Windows = { installOn: function (laptop) { return laptop; } };

var Laptop = function () {
    // 笔记本的固有属性
    return {
        power: false
    };
};

// 我想要一台笔记本
var laptop = new Laptop();

// 还需要给这个电脑装上CPU，内存，硬盘
laptop.cpu = new CPU('i7-2600');
laptop.memory = new Memory(8);
laptop.hardDisk = new HardDisk(512);

// 还需要给这个电脑安装系统
laptop = Windows.installOn(laptop);

// 还需要安装开关，使这台电脑可以开关机
laptop.powerOn = function () { this.power = true; };
laptop.powerOff = function () { this.power = false; };
```

试想一下，当我们有多个消费者都需要购买这台电脑时，或是消费者对于电脑有不同的需求，你就需要new多次，同时修改CPU的配置，这样的实现十分繁琐。所以我们希望将制造笔记本的工作交给工厂来完成，而消费者只需要向工厂派发订单即可拿到他们想要的笔记本。

## 车间

下面我们来尝试用JavaScript实现一个简单的车间。创造车间的意义在于，我们不想让消费者涉及到组装笔记本的逻辑，让这些繁琐的工作都交给专业的车间来解决。请看下面的例子：

```js
const CPU = function () {};
const Memory = function () {};
const HardDisk = function () {};
const Windows = { installOn: function (laptop) { return laptop; } };

var Laptop = function () {
    return {
        power: false
    };
};

// 定义一个笔记本工厂
var LaptopFactory = {
    // 工厂可以生产Laptop
    makeLaptop () {
        var laptop = new Laptop();

        laptop.cpu = new CPU('i7-2600');
        laptop.memory = new Memory(8);
        laptop.hardDisk = new HardDisk(512);

        laptop = Windows.installOn(laptop);

        laptop.powerOn = function () { this.power = true; };
        laptop.powerOff = function () { this.power = false; };

        return laptop;
    }
};
```

这个笔记本车间现在只做了一件事情，就是生产笔记本电脑，并且只能生产一种型号。抽象地来说，我们定义了一个叫做`LaptopFactory`的类，这个类只有一个方法，用来创建和返回`Laptop`对象。但我们已经可以初见它的效力：

```js
// 想要一台笔记本？向工厂下订单！
var laptop = LaptopFactory.makeLaptop();
```

然而这个车间只可以生产一种型号的笔记本，这样怎么可以忽悠土豪呢？让我们扩展生产线，使车间变得更好，这样就可以生产更多型号的笔记本！

```js
const CPU = function () {};
const Memory = function () {};
const HardDisk = function () {};
const Windows = function () { return { installOn: function (laptop) { laptop.system = 'Windows'; return laptop; } }; };
const MacOs = function () { return { installOn: function (laptop) { laptop.system = 'macOS'; return laptop; } }; };

var Laptop = function () {
    return {
        power: false
    };
};

var LaptopFactory = {
    Windows: new Windows(),
    MacOs: new MacOs(),
    makeLaptop: function ({ cpu = 'i7-2600', memory = 8, hardDisk = 512, system = null } = {}) {
        var laptop = new Laptop();

        laptop.cpu = new CPU(cpu);
        laptop.memory = new Memory(memory);
        laptop.hardDisk = new HardDisk(hardDisk);

        if (system === 'Windows') {
            laptop = this.Windows.installOn(laptop);
        } else if (system === 'macOS') {
            laptop = this.MacOs.installOn(laptop);
        }

        laptop.powerOn = function () { this.power = true; };
        laptop.powerOff = function () { this.power = false; };

        return laptop;
    }
};

// 我想要更多内存
var laptop = LaptopFactory.makeLaptop({ memory: 16, system: 'macOS' });
```

车间还不能够被称之为工厂，因为它的产量有限，且产品太单一了。如果客户想要其他类型的电脑，比如台式机、服务器，或是其他品牌的电脑，一个车间就无能为力了。当然，我们可以再次扩充生产线，使它在一定程度上表现出工厂的效能。

## 简单工厂

土豪说，我还想买台式机，可是工厂只能生产笔记本，怎么办呢？我们想让生产线的工人身兼数职，不仅可以组装笔记本，也可以组装台式机。车间经理在订单上添加了一个属性`type`让用户选择自己需要什么类型的机器，工人就可以根据订单生产出笔记本和台式机了！

同样的，笔记本和台式机都属于电脑，所以他们共有电源的属性，也都有CPU、内存和硬盘。

```js
const CPU = function () {};
const Memory = function () {};
const HardDisk = function () {};
const Windows = function () { return { installOn: function (laptop) { laptop.system = 'Windows'; return laptop; } }; };
const MacOs = function () { return { installOn: function (laptop) { laptop.system = 'macOS'; return laptop; } }; };

// 定义两种电脑：笔记本和台式机
var Computer = function () {
    this.power = false;
};
var Laptop = function () {};
Laptop.prototype = new Computer();
var Pc = function () {};
Pc.prototype = new Computer();

// 再次扩展我们的生产线
var ComputerFactory = {
    Windows: new Windows(),
    MacOs: new MacOs(),
    make: function ({ cpu = 'i7-2600', memory = 8, hardDisk = 512, system = null, type = 'laptop' } = {}) {
        var computer;

        // 根据用户的选择生产不同的电脑
        switch (type) {
            case 'laptop':
                computer = new Laptop();
                break;
            case 'pc':
                computer = new Pc();
                break;
            default:
                throw 'Cannot make ' + type;
        }

        computer.cpu = new CPU(cpu);
        computer.memory = new Memory(memory);
        computer.hardDisk = new HardDisk(hardDisk);

        if (system === 'Windows') {
            computer = this.Windows.installOn(computer);
        } else if (system === 'macOS') {
            computer = this.MacOs.installOn(computer);
        }

        computer.powerOn = function () { this.power = true; };
        computer.powerOff = function () { this.power = false; };

        return computer;
    }
};

// 我想要一台笔记本
var laptop = ComputerFactory.make({ type: 'laptop' });

// 我想要一台台式机
var pc = ComputerFactory.make({ type: 'pc' });
```

在这里的简单工厂模式，又叫静态工厂方法（Static Factory Method），是最简单的工厂方法。实际上，这个方法违背了我们的开闭原则。使用简单模式的笔记本工厂，只能生产同类型的产品，即笔记本。如果我们修改工厂产生Laptop的基类，想让这个工厂生产更多不同类型的产品，比如台式机，那么不仅需要修改生产的方法，也需要修改消费者下订单的代码。

## 工厂方法

真正的工厂方法比起简单工厂更加抽象。根据工厂方法的定义，我们需要使一个类的实例化推迟到子类中进行

想让工厂生产台式机？让我们再加一个抽象层，即建立多个车间。

```js
var Computer = function () {
    this.power = false;
};

Computer.prototype.powerOn = function () { this.power = true; };
Computer.prototype.powerOff = function () { this.power = false; };

var Factory = function () {};
Factory.prototype = {
    installBasicHardware: function (computer = {}, { cpu = 'i7-2600', memory = 8, hardDisk = 512 }) {
        computer.cpu = new CPU(cpu);
        computer.memory = new Memory(memory);
        computer.hardDisk = new HardDisk(hardDisk);
    },
    make: function () {
        console.log('Abstract factory cannot make computer');
    }, 
    installSystem: function () {
        Windows.installOn(this);
    }
};

var LaptopFactory = function () {};
LaptopFactory.prototype = new Factory();
LaptopFactory.prototype.make = function (config) {
    var laptop = new Computer();

    this.installBasicHardware(laptop, config);

    laptop.screen = false;
    laptop.openLid = function () { this.screen = true; };
    laptop.closeLid = function () { this.screen = false; };

    return laptop;
};

var PcFactory = function () {};
PcFactory.prototype = new Factory();
PcFactory.prototype.make = function (config) {
    var pc = new Computer();

    this.installBasicHardware(pc, config);

    return pc;
};
```



