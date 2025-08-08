# 结构型设计模式 (Structural Design Patterns)

## 原始定义

结构型设计模式是处理类或对象组合的设计模式。它们通过识别简单的方式来实现关系，从而形成更大的结构。结构型模式关注如何组合类和对象以形成更大的结构。

## 详细解释

结构型设计模式涉及如何组合类和对象以获得更大的结构。这些模式通常通过继承来组合接口或实现，或者通过组合和委派来组合对象。结构型模式帮助确保当某些部分发生变化时，整个结构不会受到影响，同时它们有助于简化结构并识别系统中较小的部分。

### 核心概念

1. **组合优于继承**：通过组合对象而不是继承类来构建更大的结构
2. **接口适配**：使不兼容的接口能够协同工作
3. **透明性**：客户端代码不需要知道它们正在处理的是单个对象还是对象组合
4. **灵活性**：可以在运行时动态地改变对象之间的关系

### 结构型模式分类

结构型模式包括以下七种模式：
1. 适配器模式 (Adapter Pattern)
2. 桥接模式 (Bridge Pattern)
3. 组合模式 (Composite Pattern)
4. 装饰器模式 (Decorator Pattern)
5. 外观模式 (Facade Pattern)
6. 享元模式 (Flyweight Pattern)
7. 代理模式 (Proxy Pattern)

## 1. 适配器模式 (Adapter Pattern)

### 原始定义

适配器模式将一个类的接口转换成客户希望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

### 详细解释

适配器模式是一种结构型设计模式，它允许具有不兼容接口的对象协同工作。适配器充当两个不同接口之间的桥梁，将一个接口转换为客户期望的另一个接口。这种模式涉及一个单独的类，它负责连接独立或不兼容接口的功能。

### 适用场景

1. 当希望使用某个类，但其接口与其他代码不兼容时
2. 当希望复用几个现有子类，但它们缺少某些共同功能，而不能将这些功能添加到超类中时
3. 当需要在现有系统中集成第三方库或遗留代码时
4. 当需要统一多个相似但接口不同的类时

### 实施步骤

#### 第一步：定义目标接口

```javascript
// 目标接口 - 客户端期望的接口
class PaymentProcessor {
  processPayment(amount) {
    throw new Error("processPayment() method must be implemented");
  }
  
  refundPayment(transactionId) {
    throw new Error("refundPayment() method must be implemented");
  }
}
```

#### 第二步：实现不兼容的类

```javascript
// 不兼容的类 - 第三方支付系统
class StripePaymentSystem {
  makeCharge(amountInCents) {
    console.log(`Processing $${amountInCents/100} payment via Stripe`);
    return { success: true, transactionId: `stripe_${Date.now()}` };
  }
  
  makeRefund(transactionId) {
    console.log(`Refunding transaction ${transactionId} via Stripe`);
    return { success: true, refundedAmount: 0 };
  }
}

class PayPalPaymentSystem {
  sendPayment(amount) {
    console.log(`Processing $${amount} payment via PayPal`);
    return { status: 'success', id: `paypal_${Date.now()}` };
  }
  
  processRefund(transactionId) {
    console.log(`Refunding transaction ${transactionId} via PayPal`);
    return { status: 'refunded', amount: 0 };
  }
}
```

#### 第三步：实现适配器

```javascript
// 适配器 - 使Stripe系统兼容PaymentProcessor接口
class StripeAdapter extends PaymentProcessor {
  constructor(stripeSystem) {
    super();
    this.stripeSystem = stripeSystem;
  }
  
  processPayment(amount) {
    // 将金额转换为美分
    const amountInCents = Math.round(amount * 100);
    const result = this.stripeSystem.makeCharge(amountInCents);
    
    return {
      success: result.success,
      transactionId: result.transactionId
    };
  }
  
  refundPayment(transactionId) {
    const result = this.stripeSystem.makeRefund(transactionId);
    return {
      success: result.success,
      refundedAmount: result.refundedAmount
    };
  }
}

// 适配器 - 使PayPal系统兼容PaymentProcessor接口
class PayPalAdapter extends PaymentProcessor {
  constructor(paypalSystem) {
    super();
    this.paypalSystem = paypalSystem;
  }
  
  processPayment(amount) {
    const result = this.paypalSystem.sendPayment(amount);
    
    return {
      success: result.status === 'success',
      transactionId: result.id
    };
  }
  
  refundPayment(transactionId) {
    const result = this.paypalSystem.processRefund(transactionId);
    return {
      success: result.status === 'refunded',
      refundedAmount: result.amount
    };
  }
}
```

#### 第四步：使用适配器

```javascript
// 客户端代码
class ShoppingCart {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
    this.items = [];
  }
  
  addItem(item) {
    this.items.push(item);
  }
  
  getTotal() {
    return this.items.reduce((total, item) => total + item.price, 0);
  }
  
  checkout() {
    const total = this.getTotal();
    console.log(`Checking out with total: $${total}`);
    
    const result = this.paymentProcessor.processPayment(total);
    if (result.success) {
      console.log(`Payment successful. Transaction ID: ${result.transactionId}`);
      return result.transactionId;
    } else {
      console.log("Payment failed");
      return null;
    }
  }
}

// 使用示例
const stripeSystem = new StripePaymentSystem();
const stripeAdapter = new StripeAdapter(stripeSystem);
const cart1 = new ShoppingCart(stripeAdapter);

cart1.addItem({ name: "Product 1", price: 29.99 });
cart1.addItem({ name: "Product 2", price: 19.99 });
cart1.checkout();

const paypalSystem = new PayPalPaymentSystem();
const paypalAdapter = new PayPalAdapter(paypalSystem);
const cart2 = new ShoppingCart(paypalAdapter);

cart2.addItem({ name: "Product 3", price: 39.99 });
cart2.checkout();
```

#### 第五步：双向适配器

```javascript
// 双向适配器 - 同时适配两个不兼容的接口
class TwoWayPaymentAdapter {
  constructor(stripeSystem, paypalSystem) {
    this.stripeAdapter = new StripeAdapter(stripeSystem);
    this.paypalAdapter = new PayPalAdapter(paypalSystem);
    this.currentProcessor = this.stripeAdapter;
  }
  
  switchProcessor(processorType) {
    if (processorType === 'stripe') {
      this.currentProcessor = this.stripeAdapter;
    } else if (processorType === 'paypal') {
      this.currentProcessor = this.paypalAdapter;
    }
  }
  
  processPayment(amount) {
    return this.currentProcessor.processPayment(amount);
  }
  
  refundPayment(transactionId) {
    return this.currentProcessor.refundPayment(transactionId);
  }
}

// 使用双向适配器
const twoWayAdapter = new TwoWayPaymentAdapter(
  new StripePaymentSystem(),
  new PayPalPaymentSystem()
);

// 使用Stripe处理
twoWayAdapter.switchProcessor('stripe');
twoWayAdapter.processPayment(99.99);

// 切换到PayPal处理
twoWayAdapter.switchProcessor('paypal');
twoWayAdapter.processPayment(149.99);
```

### 优势

1. **单一职责原则**：可以将接口或数据转换代码从主要业务逻辑中分离
2. **开闭原则**：只要客户端代码通过目标接口与适配器进行交互，就能在不修改现有客户端代码的情况下添加新的适配器
3. **兼容性**：可以让任何两个不相关的类一起工作
4. **灵活性**：提高了类的复用性

### 最佳实践

1. **明确适配目标**：在实现适配器之前，明确需要适配的接口和目标接口
2. **保持简单**：适配器应该尽可能简单，只负责接口转换
3. **错误处理**：在适配器中妥善处理不兼容的数据类型或异常情况
4. **文档化**：清楚地记录适配器的作用和使用方式

### 常见误区

1. **过度适配**：为每个不兼容的类都创建适配器，导致系统复杂化
2. **适配器过重**：在适配器中实现过多的业务逻辑
3. **隐藏复杂性**：适配器可能隐藏了底层系统的复杂性，导致性能问题

## 2. 桥接模式 (Bridge Pattern)

### 原始定义

桥接模式将抽象部分与它的实现部分分离，使它们都可以独立地变化。

### 详细解释

桥接模式是一种结构型设计模式，它将抽象与实现分离，使它们可以独立变化。桥接模式通过组合关系代替继承关系来实现，从而降低了抽象和实现这两个可变维度的耦合度。

### 适用场景

1. 当不希望在抽象和其实现之间有一个固定的绑定关系时
2. 当类的抽象以及其实现有许多种可能的变化时
3. 当一个类存在两个独立变化的维度，且这两个维度都需要进行扩展时
4. 当想在多个对象间共享实现，但同时要求客户端并不知道这一点时

### 实施步骤

#### 第一步：定义实现接口

```javascript
// 实现接口 - 定义实现类的接口
class DrawingAPI {
  drawCircle(x, y, radius) {
    throw new Error("drawCircle() method must be implemented");
  }
  
  drawRectangle(x, y, width, height) {
    throw new Error("drawRectangle() method must be implemented");
  }
}
```

#### 第二步：实现具体实现类

```javascript
// 具体实现类A - SVG实现
class SVGDrawingAPI extends DrawingAPI {
  drawCircle(x, y, radius) {
    console.log(`Drawing SVG circle at (${x}, ${y}) with radius ${radius}`);
    // 实际的SVG绘制代码
    return `<circle cx="${x}" cy="${y}" r="${radius}" fill="blue" />`;
  }
  
  drawRectangle(x, y, width, height) {
    console.log(`Drawing SVG rectangle at (${x}, ${y}) with size ${width}x${height}`);
    // 实际的SVG绘制代码
    return `<rect x="${x}" y="${y}" width="${width}" height="${height}" fill="red" />`;
  }
}

// 具体实现类B - Canvas实现
class CanvasDrawingAPI extends DrawingAPI {
  drawCircle(x, y, radius) {
    console.log(`Drawing Canvas circle at (${x}, ${y}) with radius ${radius}`);
    // 实际的Canvas绘制代码
    return `Canvas: Circle(${x}, ${y}, ${radius})`;
  }
  
  drawRectangle(x, y, width, height) {
    console.log(`Drawing Canvas rectangle at (${x}, ${y}) with size ${width}x${height}`);
    // 实际的Canvas绘制代码
    return `Canvas: Rectangle(${x}, ${y}, ${width}, ${height})`;
  }
}
```

#### 第三步：定义抽象类

```javascript
// 抽象类 - 定义抽象接口
class Shape {
  constructor(drawingAPI) {
    // 桥接 - 将实现部分注入到抽象部分
    this.drawingAPI = drawingAPI;
  }
  
  draw() {
    throw new Error("draw() method must be implemented");
  }
  
  resize(factor) {
    throw new Error("resize() method must be implemented");
  }
}
```

#### 第四步：实现具体抽象类

```javascript
// 具体抽象类A - 圆形
class Circle extends Shape {
  constructor(x, y, radius, drawingAPI) {
    super(drawingAPI);
    this.x = x;
    this.y = y;
    this.radius = radius;
  }
  
  draw() {
    return this.drawingAPI.drawCircle(this.x, this.y, this.radius);
  }
  
  resize(factor) {
    this.radius *= factor;
  }
  
  move(dx, dy) {
    this.x += dx;
    this.y += dy;
  }
}

// 具体抽象类B - 矩形
class Rectangle extends Shape {
  constructor(x, y, width, height, drawingAPI) {
    super(drawingAPI);
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
  }
  
  draw() {
    return this.drawingAPI.drawRectangle(this.x, this.y, this.width, this.height);
  }
  
  resize(factor) {
    this.width *= factor;
    this.height *= factor;
  }
  
  move(dx, dy) {
    this.x += dx;
    this.y += dy;
  }
}
```

#### 第五步：使用桥接模式

```javascript
// 使用示例
class DrawingApplication {
  constructor() {
    this.shapes = [];
  }
  
  addShape(shape) {
    this.shapes.push(shape);
  }
  
  renderAll() {
    console.log("Rendering all shapes:");
    return this.shapes.map(shape => shape.draw());
  }
  
  resizeAll(factor) {
    this.shapes.forEach(shape => shape.resize(factor));
  }
}

// 创建不同的实现
const svgAPI = new SVGDrawingAPI();
const canvasAPI = new CanvasDrawingAPI();

// 创建图形并使用不同的实现
const app = new DrawingApplication();

// 添加使用SVG实现的图形
app.addShape(new Circle(10, 10, 5, svgAPI));
app.addShape(new Rectangle(20, 20, 10, 15, svgAPI));

// 添加使用Canvas实现的图形
app.addShape(new Circle(30, 30, 7, canvasAPI));
app.addShape(new Rectangle(40, 40, 12, 18, canvasAPI));

// 渲染所有图形
const results = app.renderAll();
console.log("Render results:", results);

// 调整所有图形大小
app.resizeAll(1.5);
```

#### 第六步：高级桥接实现

```javascript
// 支持运行时切换实现的桥接模式
class SwitchableShape extends Shape {
  constructor(drawingAPI) {
    super(drawingAPI);
    this.implementations = new Map();
  }
  
  addImplementation(name, implementation) {
    this.implementations.set(name, implementation);
  }
  
  switchImplementation(name) {
    const implementation = this.implementations.get(name);
    if (implementation) {
      this.drawingAPI = implementation;
    }
  }
  
  getCurrentImplementationName() {
    for (let [name, impl] of this.implementations) {
      if (impl === this.drawingAPI) {
        return name;
      }
    }
    return 'unknown';
  }
}

// 扩展圆形支持切换实现
class SwitchableCircle extends SwitchableShape {
  constructor(x, y, radius, drawingAPI) {
    super(drawingAPI);
    this.x = x;
    this.y = y;
    this.radius = radius;
    
    // 添加默认实现
    this.addImplementation('svg', new SVGDrawingAPI());
    this.addImplementation('canvas', new CanvasDrawingAPI());
  }
  
  draw() {
    return this.drawingAPI.drawCircle(this.x, this.y, this.radius);
  }
  
  resize(factor) {
    this.radius *= factor;
  }
}

// 使用支持切换实现的图形
const switchableCircle = new SwitchableCircle(50, 50, 10, svgAPI);
console.log("Current implementation:", switchableCircle.getCurrentImplementationName());
console.log(switchableCircle.draw());

// 切换到Canvas实现
switchableCircle.switchImplementation('canvas');
console.log("Current implementation:", switchableCircle.getCurrentImplementationName());
console.log(switchableCircle.draw());
```

### 优势

1. **分离抽象和实现**：抽象和实现可以独立开发和变化
2. **提高可扩展性**：可以独立地扩展抽象和实现
3. **隐藏实现细节**：客户端不需要知道实现的细节
4. **符合开闭原则**：对扩展开放，对修改关闭

### 最佳实践

1. **明确抽象和实现**：在设计时明确哪些是抽象部分，哪些是实现部分
2. **保持接口稳定**：确保抽象接口和实现接口的稳定性
3. **合理分层**：将系统合理地分为抽象层和实现层
4. **文档化设计**：清楚地记录桥接模式的设计意图和结构

### 常见误区

1. **过度设计**：对于简单系统也使用桥接模式
2. **接口不匹配**：抽象和实现的接口设计不匹配
3. **增加复杂性**：桥接模式可能增加系统的复杂性

## 3. 组合模式 (Composite Pattern)

### 原始定义

组合模式将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

### 详细解释

组合模式是一种结构型设计模式，它允许你将对象组合成树形结构来表示"部分-整体"的层次结构。组合能让客户端以一致的方式处理单个对象和对象组合。

### 适用场景

1. 当想表示对象的部分-整体层次结构时
2. 当希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象时
3. 当结构可以递归地组合时
4. 当需要对树形结构进行操作时

### 实施步骤

#### 第一步：定义组件接口

```javascript
// 组件接口 - 为组合中的对象声明公共接口
class FileSystemComponent {
  constructor(name) {
    this.name = name;
  }
  
  getName() {
    return this.name;
  }
  
  getSize() {
    throw new Error("getSize() method must be implemented");
  }
  
  add(component) {
    throw new Error("add() method not supported");
  }
  
  remove(component) {
    throw new Error("remove() method not supported");
  }
  
  getChild(index) {
    throw new Error("getChild() method not supported");
  }
  
  display(indent = 0) {
    throw new Error("display() method must be implemented");
  }
}
```

#### 第二步：实现叶子节点

```javascript
// 叶子节点 - 表示树形结构中的叶节点对象
class File extends FileSystemComponent {
  constructor(name, size) {
    super(name);
    this.size = size;
  }
  
  getSize() {
    return this.size;
  }
  
  display(indent = 0) {
    const spaces = ' '.repeat(indent);
    console.log(`${spaces}- File: ${this.name} (${this.size} bytes)`);
  }
}
```

#### 第三步：实现组合节点

```javascript
// 组合节点 - 定义有子部件的部件行为
class Directory extends FileSystemComponent {
  constructor(name) {
    super(name);
    this.children = [];
  }
  
  getSize() {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }
  
  add(component) {
    this.children.push(component);
  }
  
  remove(component) {
    const index = this.children.indexOf(component);
    if (index !== -1) {
      this.children.splice(index, 1);
    }
  }
  
  getChild(index) {
    return this.children[index];
  }
  
  display(indent = 0) {
    const spaces = ' '.repeat(indent);
    console.log(`${spaces}+ Directory: ${this.name} (${this.getSize()} bytes)`);
    
    this.children.forEach(child => {
      child.display(indent + 2);
    });
  }
  
  // 额外的方法
  findByName(name) {
    if (this.name === name) {
      return this;
    }
    
    for (let child of this.children) {
      if (child.getName() === name) {
        return child;
      }
      
      if (child instanceof Directory) {
        const found = child.findByName(name);
        if (found) {
          return found;
        }
      }
    }
    
    return null;
  }
  
  getAllFiles() {
    let files = [];
    
    for (let child of this.children) {
      if (child instanceof File) {
        files.push(child);
      } else if (child instanceof Directory) {
        files = files.concat(child.getAllFiles());
      }
    }
    
    return files;
  }
}
```

#### 第四步：使用组合模式

```javascript
// 构建文件系统结构
const root = new Directory("root");

// 创建文件
const file1 = new File("document.txt", 1024);
const file2 = new File("image.jpg", 2048);
const file3 = new File("script.js", 512);
const file4 = new File("style.css", 256);

// 创建目录
const homeDir = new Directory("home");
const userDir = new Directory("user");
const docsDir = new Directory("documents");
const picsDir = new Directory("pictures");

// 构建目录结构
root.add(homeDir);
root.add(docsDir);
root.add(picsDir);

homeDir.add(userDir);
userDir.add(file1);
userDir.add(file2);

docsDir.add(file3);
picsDir.add(file4);

// 显示文件系统结构
console.log("File System Structure:");
root.display();

// 计算总大小
console.log(`\nTotal size: ${root.getSize()} bytes`);

// 查找文件
const foundFile = root.findByName("script.js");
if (foundFile) {
  console.log(`\nFound file: ${foundFile.getName()} (${foundFile.getSize()} bytes)`);
}

// 获取所有文件
const allFiles = root.getAllFiles();
console.log(`\nAll files: ${allFiles.map(f => f.getName()).join(', ')}`);
```

#### 第五步：迭代器支持

```javascript
// 添加迭代器支持的组合模式
class IterableDirectory extends Directory {
  constructor(name) {
    super(name);
  }
  
  // 实现迭代器模式
  *[Symbol.iterator]() {
    for (let child of this.children) {
      yield child;
      
      if (child instanceof IterableDirectory) {
        yield* child;
      }
    }
  }
  
  // 获取所有组件（包括目录和文件）
  getAllComponents() {
    const components = [];
    for (let component of this) {
      components.push(component);
    }
    return components;
  }
  
  // 只获取文件
  getFilesOnly() {
    const files = [];
    for (let component of this) {
      if (component instanceof File) {
        files.push(component);
      }
    }
    return files;
  }
}

// 使用迭代器
const iterableRoot = new IterableDirectory("iterable-root");
const dir1 = new IterableDirectory("dir1");
const dir2 = new IterableDirectory("dir2");

iterableRoot.add(dir1);
iterableRoot.add(dir2);
dir1.add(new File("file1.txt", 100));
dir1.add(new File("file2.txt", 200));
dir2.add(new File("file3.txt", 300));

console.log("\nUsing iterator:");
for (let component of iterableRoot) {
  console.log(`- ${component.getName()}`);
}

console.log("\nFiles only:");
const files = iterableRoot.getFilesOnly();
files.forEach(file => console.log(`- ${file.getName()} (${file.getSize()} bytes)`));
```

### 优势

1. **一致性**：客户端可以一致地使用组合结构和单个对象
2. **灵活性**：容易增加新的组件类型
3. **简化客户端代码**：客户端不需要知道处理的是单个对象还是组合对象
4. **符合开闭原则**：容易扩展新的组件类型

### 最佳实践

1. **明确定义组件接口**：确保所有组件都实现相同的接口
2. **合理设计层次结构**：避免过深的层次结构
3. **处理安全性和透明性**：在安全性和透明性之间做出权衡
4. **提供便利方法**：为常用的组合操作提供便利方法

### 常见误区

1. **过度通用**：使接口过于通用，导致难以理解
2. **忽略类型安全**：没有适当的类型检查可能导致运行时错误
3. **性能问题**：递归操作可能导致性能问题

## 4. 装饰器模式 (Decorator Pattern)

### 原始定义

装饰器模式动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

### 详细解释

装饰器模式是一种结构型设计模式，它允许你通过将对象放入包含行为的特殊封装对象中来为原对象绑定新的行为。装饰器模式通过创建一个包装对象，也就是装饰器，来包裹真实的对象，可以在不修改原对象的情况下为其添加新的功能。

### 适用场景

1. 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责
2. 处理那些可以撤销的职责
3. 当不能采用生成子类的方法进行扩充时
4. 当需要动态地给一个对象增加功能，这些功能也可以动态地被撤销时

### 实施步骤

#### 第一步：定义组件接口

```javascript
// 组件接口 - 定义一个对象接口，可以给这些对象动态地添加职责
class Notifier {
  send(message) {
    throw new Error("send() method must be implemented");
  }
}
```

#### 第二步：实现具体组件

```javascript
// 具体组件 - 定义一个具体的对象，也可以给这个对象添加一些职责
class EmailNotifier extends Notifier {
  send(message) {
    console.log(`Sending email: ${message}`);
    return `Email sent: ${message}`;
  }
}
```

#### 第三步：定义装饰器基类

```javascript
// 装饰器基类 - 持有一个组件对象，并定义一个与抽象组件接口一致的接口
class NotifierDecorator extends Notifier {
  constructor(notifier) {
    super();
    this.notifier = notifier;
  }
  
  send(message) {
    return this.notifier.send(message);
  }
}
```

#### 第四步：实现具体装饰器

```javascript
// 具体装饰器A - 向组件添加职责
class SMSDecorator extends NotifierDecorator {
  constructor(notifier) {
    super(notifier);
  }
  
  send(message) {
    const result = super.send(message);
    const smsResult = this.sendSMS(message);
    return `${result}\n${smsResult}`;
  }
  
  sendSMS(message) {
    console.log(`Sending SMS: ${message}`);
    return `SMS sent: ${message}`;
  }
}

// 具体装饰器B
class SlackDecorator extends NotifierDecorator {
  constructor(notifier) {
    super(notifier);
  }
  
  send(message) {
    const result = super.send(message);
    const slackResult = this.sendSlack(message);
    return `${result}\n${slackResult}`;
  }
  
  sendSlack(message) {
    console.log(`Sending Slack message: ${message}`);
    return `Slack message sent: ${message}`;
  }
}

// 具体装饰器C
class EncryptionDecorator extends NotifierDecorator {
  constructor(notifier) {
    super(notifier);
  }
  
  send(message) {
    const encryptedMessage = this.encrypt(message);
    const result = super.send(encryptedMessage);
    return `Encrypted: ${encryptedMessage}\n${result}`;
  }
  
  encrypt(message) {
    // 简单的加密示例
    return message.split('').reverse().join('');
  }
}

// 具体装饰器D
class LoggingDecorator extends NotifierDecorator {
  constructor(notifier) {
    super(notifier);
  }
  
  send(message) {
    console.log(`[LOG] Sending message: ${message}`);
    const startTime = Date.now();
    
    const result = super.send(message);
    
    const endTime = Date.now();
    console.log(`[LOG] Message sent in ${endTime - startTime}ms`);
    
    return result;
  }
}
```

#### 第五步：使用装饰器模式

```javascript
// 使用示例
class NotificationService {
  constructor(notifier) {
    this.notifier = notifier;
  }
  
  notify(message) {
    return this.notifier.send(message);
  }
}

// 基础通知
let notifier = new EmailNotifier();
let service = new NotificationService(notifier);
console.log("=== Basic Email Notification ===");
console.log(service.notify("Hello World!"));

// 添加SMS通知
notifier = new SMSDecorator(new EmailNotifier());
service = new NotificationService(notifier);
console.log("\n=== Email + SMS Notification ===");
console.log(service.notify("Hello World!"));

// 添加Slack通知
notifier = new SlackDecorator(new SMSDecorator(new EmailNotifier()));
service = new NotificationService(notifier);
console.log("\n=== Email + SMS + Slack Notification ===");
console.log(service.notify("Hello World!"));

// 添加加密和日志
notifier = new LoggingDecorator(
  new EncryptionDecorator(
    new SlackDecorator(
      new SMSDecorator(
        new EmailNotifier()
      )
    )
  )
);
service = new NotificationService(notifier);
console.log("\n=== Full Featured Notification ===");
console.log(service.notify("Secret Message"));
```

#### 第六步：高级装饰器实现

```javascript
// 支持链式调用的装饰器
class ChainedNotifier extends Notifier {
  constructor() {
    super();
    this.decorators = [];
  }
  
  addDecorator(decoratorClass, ...args) {
    this.decorators.push({ decoratorClass, args });
    return this;
  }
  
  send(message) {
    // 创建基础通知器
    let notifier = new EmailNotifier();
    
    // 依次应用装饰器
    for (let { decoratorClass, args } of this.decorators) {
      notifier = new decoratorClass(notifier, ...args);
    }
    
    return notifier.send(message);
  }
}

// 使用链式装饰器
const chainedNotifier = new ChainedNotifier()
  .addDecorator(SMSDecorator)
  .addDecorator(SlackDecorator)
  .addDecorator(EncryptionDecorator)
  .addDecorator(LoggingDecorator);

const chainedService = new NotificationService(chainedNotifier);
console.log("\n=== Chained Decorators ===");
console.log(chainedService.notify("Chained Message"));
```

### 优势

1. **灵活性**：比静态继承更灵活
2. **避免类爆炸**：通过使用不同的装饰器可以实现多种组合
3. **符合开闭原则**：可以不修改原有代码的情况下扩展功能
4. **职责分离**：每个装饰器只关注自己的功能

### 最佳实践

1. **保持装饰器简单**：每个装饰器应该只负责一个功能
2. **明确装饰顺序**：注意装饰器的顺序可能影响结果
3. **提供默认实现**：装饰器基类应该提供默认的实现
4. **文档化装饰器**：清楚地记录每个装饰器的作用

### 常见误区

1. **装饰器过多**：可能导致系统中装饰器类过多
2. **调试困难**：多层装饰可能使调试变得困难
3. **性能影响**：多层装饰可能影响性能

## 5. 外观模式 (Facade Pattern)

### 原始定义

外观模式为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

### 详细解释

外观模式是一种结构型设计模式，它为复杂的子系统提供一个简单的接口。外观模式通过创建一个外观类来隐藏复杂的子系统接口，使得客户端更容易使用子系统。

### 适用场景

1. 当要为一个复杂子系统提供一个简单接口时
2. 当客户程序与抽象类的实现部分之间存在着很大的依赖性时
3. 当需要构建一个层次结构的子系统时
4. 当需要将子系统与客户程序解耦时

### 实施步骤

#### 第一步：定义子系统类

```javascript
// 子系统类A - CPU
class CPU {
  freeze() {
    console.log("CPU: Freezing processor");
  }
  
  jump(position) {
    console.log(`CPU: Jumping to position ${position}`);
  }
  
  execute() {
    console.log("CPU: Executing instructions");
  }
}

// 子系统类B - 内存
class Memory {
  load(position, data) {
    console.log(`Memory: Loading data at position ${position}`);
    console.log(`Data: ${data}`);
  }
}

// 子系统类C - 硬盘
class HardDrive {
  read(lba, size) {
    console.log(`HardDrive: Reading ${size} bytes from LBA ${lba}`);
    return "Program data from hard drive";
  }
}
```

#### 第二步：实现外观类

```javascript
// 外观类 - 知道哪些子系统类负责处理请求，将客户的请求代理给适当的子系统对象
class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }
  
  start() {
    console.log("Computer: Starting...");
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    console.log("Computer: Started successfully");
  }
  
  stop() {
    console.log("Computer: Stopping...");
    // 停止计算机的逻辑
    console.log("Computer: Stopped");
  }
}
```

#### 第三步：使用外观模式

```javascript
// 客户端代码
class ComputerUser {
  constructor() {
    this.computer = new ComputerFacade();
  }
  
  useComputer() {
    console.log("=== User is using computer ===");
    this.computer.start();
    console.log("=== User is working ===");
    // 用户工作...
    console.log("=== User is done working ===");
    this.computer.stop();
  }
}

// 使用示例
const user = new ComputerUser();
user.useComputer();
```

#### 第四步：复杂子系统示例

```javascript
// 更复杂的子系统示例 - 网络请求系统

// 子系统类A - HTTP客户端
class HttpClient {
  async get(url, options = {}) {
    console.log(`HttpClient: GET ${url}`);
    // 模拟HTTP请求
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          status: 200,
          data: { message: "GET response" },
          headers: options.headers || {}
        });
      }, 100);
    });
  }
  
  async post(url, data, options = {}) {
    console.log(`HttpClient: POST ${url}`, data);
    // 模拟HTTP请求
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          status: 201,
          data: { message: "POST response", id: Date.now() },
          headers: options.headers || {}
        });
      }, 100);
    });
  }
}

// 子系统类B - 认证管理器
class AuthManager {
  constructor() {
    this.token = null;
  }
  
  login(username, password) {
    console.log(`AuthManager: Logging in user ${username}`);
    // 模拟认证
    this.token = `token_${username}_${Date.now()}`;
    return { success: true, token: this.token };
  }
  
  logout() {
    console.log("AuthManager: Logging out");
    this.token = null;
  }
  
  isAuthenticated() {
    return !!this.token;
  }
  
  getToken() {
    return this.token;
  }
}

// 子系统类C - 缓存管理器
class CacheManager {
  constructor() {
    this.cache = new Map();
  }
  
  get(key) {
    console.log(`CacheManager: Getting cache for ${key}`);
    return this.cache.get(key);
  }
  
  set(key, value, ttl = 60000) {
    console.log(`CacheManager: Setting cache for ${key}`);
    this.cache.set(key, {
      value,
      expires: Date.now() + ttl
    });
    
    // 设置过期定时器
    setTimeout(() => {
      this.cache.delete(key);
      console.log(`CacheManager: Cache expired for ${key}`);
    }, ttl);
  }
  
  has(key) {
    const item = this.cache.get(key);
    if (!item) return false;
    return item.expires > Date.now();
  }
  
  clear() {
    this.cache.clear();
    console.log("CacheManager: Cache cleared");
  }
}

// 子系统类D - 日志管理器
class LogManager {
  info(message) {
    console.log(`[INFO] ${new Date().toISOString()}: ${message}`);
  }
  
  error(message) {
    console.error(`[ERROR] ${new Date().toISOString()}: ${message}`);
  }
  
  debug(message) {
    console.log(`[DEBUG] ${new Date().toISOString()}: ${message}`);
  }
}
```

#### 第五步：实现复杂外观

```javascript
// 复杂外观类 - 网络服务外观
class NetworkServiceFacade {
  constructor() {
    this.httpClient = new HttpClient();
    this.authManager = new AuthManager();
    this.cacheManager = new CacheManager();
    this.logManager = new LogManager();
  }
  
  async login(username, password) {
    this.logManager.info(`Attempting to login user: ${username}`);
    
    try {
      const result = this.authManager.login(username, password);
      if (result.success) {
        this.logManager.info(`User ${username} logged in successfully`);
        return result;
      } else {
        this.logManager.error(`Failed to login user: ${username}`);
        return { success: false, error: "Invalid credentials" };
      }
    } catch (error) {
      this.logManager.error(`Login error for user ${username}: ${error.message}`);
      return { success: false, error: error.message };
    }
  }
  
  async get(url, useCache = true) {
    // 检查缓存
    if (useCache && this.cacheManager.has(url)) {
      this.logManager.info(`Cache hit for ${url}`);
      return this.cacheManager.get(url);
    }
    
    // 检查认证
    if (!this.authManager.isAuthenticated()) {
      this.logManager.error("User not authenticated");
      return { success: false, error: "Not authenticated" };
    }
    
    try {
      this.logManager.info(`Making GET request to ${url}`);
      
      const response = await this.httpClient.get(url, {
        headers: {
          'Authorization': `Bearer ${this.authManager.getToken()}`,
          'Content-Type': 'application/json'
        }
      });
      
      // 缓存响应
      if (useCache) {
        this.cacheManager.set(url, response);
      }
      
      this.logManager.info(`GET request to ${url} completed with status ${response.status}`);
      return response;
    } catch (error) {
      this.logManager.error(`GET request to ${url} failed: ${error.message}`);
      return { success: false, error: error.message };
    }
  }
  
  async post(url, data) {
    // 检查认证
    if (!this.authManager.isAuthenticated()) {
      this.logManager.error("User not authenticated");
      return { success: false, error: "Not authenticated" };
    }
    
    try {
      this.logManager.info(`Making POST request to ${url}`);
      
      const response = await this.httpClient.post(url, data, {
        headers: {
          'Authorization': `Bearer ${this.authManager.getToken()}`,
          'Content-Type': 'application/json'
        }
      });
      
      // 清除相关缓存
      this.cacheManager.clear();
      
      this.logManager.info(`POST request to ${url} completed with status ${response.status}`);
      return response;
    } catch (error) {
      this.logManager.error(`POST request to ${url} failed: ${error.message}`);
      return { success: false, error: error.message };
    }
  }
  
  logout() {
    this.logManager.info("Logging out user");
    this.authManager.logout();
    this.cacheManager.clear();
    this.logManager.info("User logged out successfully");
  }
}
```

#### 第六步：使用复杂外观

```javascript
// 使用复杂外观的客户端代码
class ApiServiceClient {
  constructor() {
    this.networkService = new NetworkServiceFacade();
  }
  
  async initialize() {
    console.log("=== Initializing API Service ===");
    
    // 登录
    const loginResult = await this.networkService.login("user123", "password123");
    if (!loginResult.success) {
      console.log("Login failed:", loginResult.error);
      return false;
    }
    
    console.log("Login successful, token:", loginResult.token);
    return true;
  }
  
  async fetchUserData() {
    console.log("=== Fetching User Data ===");
    
    const response = await this.networkService.get("https://api.example.com/user");
    if (response.success) {
      console.log("User data:", response.data);
      return response.data;
    } else {
      console.log("Failed to fetch user data:", response.error);
      return null;
    }
  }
  
  async createPost(postData) {
    console.log("=== Creating Post ===");
    
    const response = await this.networkService.post("https://api.example.com/posts", postData);
    if (response.success) {
      console.log("Post created:", response.data);
      return response.data;
    } else {
      console.log("Failed to create post:", response.error);
      return null;
    }
  }
  
  cleanup() {
    console.log("=== Cleaning Up ===");
    this.networkService.logout();
  }
}

// 使用示例
async function demonstrateFacade() {
  const client = new ApiServiceClient();
  
  // 初始化
  const initialized = await client.initialize();
  if (!initialized) return;
  
  // 获取用户数据
  await client.fetchUserData();
  
  // 创建新文章
  await client.createPost({
    title: "Hello World",
    content: "This is my first post"
  });
  
  // 清理
  client.cleanup();
}

// 运行演示
demonstrateFacade().then(() => {
  console.log("=== Demonstration Complete ===");
});
```

### 优势

1. **简化接口**：为复杂的子系统提供简单的接口
2. **解耦客户端和子系统**：客户端不需要了解子系统的内部实现
3. **提高灵活性**：可以更容易地替换子系统
4. **符合迪米特法则**：减少了客户端与子系统的依赖关系

### 最佳实践

1. **保持外观简单**：外观类应该只提供必要的方法
2. **不封装所有功能**：外观不应该封装子系统的所有功能
3. **提供访问子系统的途径**：在必要时允许客户端直接访问子系统
4. **文档化接口**：清楚地记录外观类提供的接口

### 常见误区

1. **外观过于复杂**：外观类变得过于复杂，违背了简化接口的目的
2. **成为上帝对象**：外观类承担了过多的职责
3. **隐藏重要功能**：过度封装可能隐藏了子系统的重要功能

## 6. 享元模式 (Flyweight Pattern)

### 原始定义

享元模式运用共享技术有效地支持大量细粒度的对象。

### 详细解释

享元模式是一种结构型设计模式，它通过共享多个对象所共有的相同状态，让你能在有限的内存容量中载入更多对象。享元模式试图重用现有的同类对象，如果未找到匹配的对象，则创建新对象。

### 适用场景

1. 一个应用程序使用了大量的对象
2. 由于使用了大量对象，造成很大的存储开销
3. 对象的大多数状态都可变为外部状态
4. 如果删除对象的外部状态，那么可以用相对较少的共享对象取代很多组对象

### 实施步骤

#### 第一步：定义享元接口

```javascript
// 享元接口 - 声明一个接口，通过它可以接受并作用于外部状态
class CharacterFlyweight {
  constructor(character) {
    // 内部状态 - 存储在享元对象内部，不会随环境改变
    this.character = character;
  }
  
  // 操作方法 - 接受外部状态作为参数
  render(font, size, color, x, y) {
    throw new Error("render() method must be implemented");
  }
  
  getCharacter() {
    return this.character;
  }
}
```

#### 第二步：实现具体享元

```javascript
// 具体享元 - 实现享元接口并为内部状态增加存储空间
class ConcreteCharacterFlyweight extends CharacterFlyweight {
  constructor(character) {
    super(character);
  }
  
  render(font, size, color, x, y) {
    // 外部状态 - 由客户端保存和计算，随环境改变
    console.log(`Rendering character '${this.character}' with:`);
    console.log(`  Font: ${font}, Size: ${size}, Color: ${color}`);
    console.log(`  Position: (${x}, ${y})`);
    
    return {
      character: this.character,
      font,
      size,
      color,
      x,
      y
    };
  }
}
```

#### 第三步：实现享元工厂

```javascript
// 享元工厂 - 创建并管理享元对象，确保合理地共享享元
class CharacterFlyweightFactory {
  constructor() {
    // 存储享元对象的池
    this.flyweights = new Map();
    this.objectsCreated = 0;
  }
  
  getFlyweight(character) {
    // 检查池中是否已存在该字符的享元对象
    if (!this.flyweights.has(character)) {
      // 如果不存在，创建新的享元对象
      const flyweight = new ConcreteCharacterFlyweight(character);
      this.flyweights.set(character, flyweight);
      this.objectsCreated++;
      console.log(`Created new flyweight for character: '${character}'`);
    }
    
    return this.flyweights.get(character);
  }
  
  getFlyweightCount() {
    return this.flyweights.size;
  }
  
  getObjectsCreated() {
    return this.objectsCreated;
  }
}
```

#### 第四步：使用享元模式

```javascript
// 使用享元模式的文本编辑器示例
class TextEditor {
  constructor() {
    this.factory = new CharacterFlyweightFactory();
    this.characters = []; // 存储外部状态
  }
  
  addCharacter(character, font, size, color, x, y) {
    // 获取享元对象
    const flyweight = this.factory.getFlyweight(character);
    
    // 存储外部状态
    this.characters.push({
      flyweight,
      font,
      size,
      color,
      x,
      y
    });
  }
  
  render() {
    console.log("=== Rendering Text ===");
    this.characters.forEach((charData, index) => {
      console.log(`Character ${index + 1}:`);
      charData.flyweight.render(
        charData.font,
        charData.size,
        charData.color,
        charData.x,
        charData.y
      );
    });
  }
  
  getStatistics() {
    return {
      totalCharacters: this.characters.length,
      uniqueCharacters: this.factory.getFlyweightCount(),
      objectsCreated: this.factory.getObjectsCreated()
    };
  }
}

// 使用示例
const editor = new TextEditor();

// 添加大量字符
const text = "HELLO WORLD HELLO WORLD";
const fonts = ["Arial", "Times New Roman", "Courier New"];
const colors = ["red", "blue", "green"];

for (let i = 0; i < text.length; i++) {
  const char = text[i];
  const font = fonts[i % fonts.length];
  const size = 12 + (i % 5);
  const color = colors[i % colors.length];
  const x = i * 10;
  const y = 0;
  
  editor.addCharacter(char, font, size, color, x, y);
}

// 渲染文本
editor.render();

// 显示统计信息
console.log("\n=== Statistics ===");
console.log(editor.getStatistics());
```

#### 第五步：复杂享元示例

```javascript
// 更复杂的享元示例 - 游戏粒子系统

// 享元接口
class ParticleFlyweight {
  constructor(type, color, size) {
    // 内部状态
    this.type = type;
    this.color = color;
    this.size = size;
  }
  
  render(x, y, velocityX, velocityY, life) {
    // 外部状态作为参数传入
    console.log(`Rendering ${this.type} particle:`);
    console.log(`  Position: (${x}, ${y})`);
    console.log(`  Velocity: (${velocityX}, ${velocityY})`);
    console.log(`  Color: ${this.color}, Size: ${this.size}`);
    console.log(`  Life: ${life}`);
  }
  
  getType() {
    return this.type;
  }
  
  getColor() {
    return this.color;
  }
  
  getSize() {
    return this.size;
  }
}

// 享元工厂
class ParticleFlyweightFactory {
  constructor() {
    this.flyweights = new Map();
    this.creationCount = 0;
  }
  
  getFlyweight(type, color, size) {
    // 创建唯一键
    const key = `${type}-${color}-${size}`;
    
    if (!this.flyweights.has(key)) {
      const flyweight = new ParticleFlyweight(type, color, size);
      this.flyweights.set(key, flyweight);
      this.creationCount++;
      console.log(`Created new particle flyweight: ${key}`);
    }
    
    return this.flyweights.get(key);
  }
  
  getFlyweightCount() {
    return this.flyweights.size;
  }
  
  getTotalCreations() {
    return this.creationCount;
  }
}

// 粒子类 - 包含外部状态
class Particle {
  constructor(flyweight, x, y, velocityX, velocityY, life) {
    this.flyweight = flyweight;
    // 外部状态
    this.x = x;
    this.y = y;
    this.velocityX = velocityX;
    this.velocityY = velocityY;
    this.life = life;
  }
  
  update(deltaTime) {
    this.x += this.velocityX * deltaTime;
    this.y += this.velocityY * deltaTime;
    this.life -= deltaTime;
  }
  
  isAlive() {
    return this.life > 0;
  }
  
  render() {
    this.flyweight.render(this.x, this.y, this.velocityX, this.velocityY, this.life);
  }
  
  getFlyweight() {
    return this.flyweight;
  }
}

// 粒子系统
class ParticleSystem {
  constructor() {
    this.factory = new ParticleFlyweightFactory();
    this.particles = [];
  }
  
  addParticle(type, color, size, x, y, velocityX, velocityY, life) {
    const flyweight = this.factory.getFlyweight(type, color, size);
    const particle = new Particle(flyweight, x, y, velocityX, velocityY, life);
    this.particles.push(particle);
  }
  
  update(deltaTime) {
    // 更新所有粒子
    this.particles.forEach(particle => particle.update(deltaTime));
    
    // 移除死亡的粒子
    this.particles = this.particles.filter(particle => particle.isAlive());
  }
  
  render() {
    console.log("=== Rendering Particles ===");
    this.particles.forEach((particle, index) => {
      console.log(`Particle ${index + 1}:`);
      particle.render();
    });
  }
  
  getStatistics() {
    const typeCount = new Map();
    
    this.particles.forEach(particle => {
      const type = particle.getFlyweight().getType();
      typeCount.set(type, (typeCount.get(type) || 0) + 1);
    });
    
    return {
      totalParticles: this.particles.length,
      particleTypes: typeCount,
      uniqueFlyweights: this.factory.getFlyweightCount(),
      totalCreations: this.factory.getTotalCreations()
    };
  }
}
```

#### 第六步：使用复杂享元

```javascript
// 使用粒子系统
const particleSystem = new ParticleSystem();

// 添加大量粒子
for (let i = 0; i < 100; i++) {
  const type = i % 3 === 0 ? 'fire' : i % 3 === 1 ? 'smoke' : 'spark';
  const color = i % 4 === 0 ? 'red' : i % 4 === 1 ? 'orange' : i % 4 === 2 ? 'gray' : 'yellow';
  const size = 5 + (i % 10);
  const x = Math.random() * 800;
  const y = Math.random() * 600;
  const velocityX = (Math.random() - 0.5) * 10;
  const velocityY = (Math.random() - 0.5) * 10;
  const life = 1 + Math.random() * 5;
  
  particleSystem.addParticle(type, color, size, x, y, velocityX, velocityY, life);
}

// 更新和渲染
particleSystem.update(0.016); // 60 FPS
particleSystem.render();

// 显示统计信息
console.log("\n=== Particle System Statistics ===");
console.log(particleSystem.getStatistics());
```

### 优势

1. **节省内存**：通过共享对象减少内存使用
2. **提高性能**：减少对象创建和垃圾回收的开销
3. **可扩展性**：可以处理大量细粒度对象
4. **资源优化**：有效利用系统资源

### 最佳实践

1. **识别内部和外部状态**：明确哪些状态可以共享，哪些必须外部化
2. **保持享元不可变**：享元对象应该是不可变的
3. **合理设计工厂**：享元工厂应该有效地管理享元对象池
4. **文档化状态分离**：清楚地记录内部状态和外部状态的划分

### 常见误区

1. **状态分离不当**：错误地将应该外部化的状态存储在享元中
2. **享元过于复杂**：享元对象变得过于复杂，失去了共享的优势
3. **性能考虑不足**：在不需要共享的场景中使用享元模式

## 7. 代理模式 (Proxy Pattern)

### 原始定义

代理模式为其他对象提供一个代理以控制对这个对象的访问。

### 详细解释

代理模式是一种结构型设计模式，它让你能够提供对象的替代品或其占位符。代理控制着对于原对象的访问，并允许在将请求提交给对象前后进行一些处理。

### 适用场景

1. 远程代理 - 为一个位于不同的地址空间的对象提供一个本地的代表对象
2. 虚拟代理 - 根据需要创建开销很大的对象
3. 保护代理 - 控制对原始对象的访问，用于对象应该有不同的访问权限的时候
4. 智能指引 - 取代了简单的指针，它在访问对象时执行一些附加操作

### 实施步骤

#### 第一步：定义主题接口

```javascript
// 主题接口 - 定义RealSubject和Proxy的共用接口
class Image {
  display() {
    throw new Error("display() method must be implemented");
  }
}
```

#### 第二步：实现真实主题

```javascript
// 真实主题 - 定义真正的对象
class RealImage extends Image {
  constructor(filename) {
    super();
    this.filename = filename;
    this.loadImageFromDisk();
  }
  
  loadImageFromDisk() {
    console.log(`Loading ${this.filename} from disk...`);
    // 模拟耗时的加载过程
    console.log("Image loaded successfully");
  }
  
  display() {
    console.log(`Displaying ${this.filename}`);
  }
}
```

#### 第三步：实现代理

```javascript
// 代理 - 保存一个引用使得代理可以访问实体，并提供一个与Subject的接口相同的接口
class ProxyImage extends Image {
  constructor(filename) {
    super();
    this.filename = filename;
    this.realImage = null;
  }
  
  display() {
    if (this.realImage === null) {
      console.log("Creating real image for the first time...");
      this.realImage = new RealImage(this.filename);
    }
    
    console.log("Displaying image through proxy");
    this.realImage.display();
  }
}
```

#### 第四步：使用代理模式

```javascript
// 使用示例
class ImageViewer {
  constructor(image) {
    this.image = image;
  }
  
  view() {
    console.log("=== Viewing Image ===");
    this.image.display();
  }
}

// 使用真实图像（立即加载）
console.log("Creating real image:");
const realImage = new RealImage("photo.jpg");
const viewer1 = new ImageViewer(realImage);
viewer1.view();

console.log("\n--- Separator ---\n");

// 使用代理图像（延迟加载）
console.log("Creating proxy image:");
const proxyImage = new ProxyImage("photo2.jpg");
const viewer2 = new ImageViewer(proxyImage);

console.log("First view:");
viewer2.view();

console.log("\nSecond view:");
viewer2.view();
```

#### 第五步：保护代理示例

```javascript
// 保护代理示例 - 控制对资源的访问

// 主题接口
class BankAccount {
  getBalance() {
    throw new Error("getBalance() method must be implemented");
  }
  
  withdraw(amount) {
    throw new Error("withdraw() method must be implemented");
  }
  
  deposit(amount) {
    throw new Error("deposit() method must be implemented");
  }
}

// 真实主题
class RealBankAccount extends BankAccount {
  constructor(accountNumber, initialBalance = 0) {
    super();
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
  }
  
  getBalance() {
    return this.balance;
  }
  
  withdraw(amount) {
    if (amount <= this.balance) {
      this.balance -= amount;
      console.log(`Withdrawn $${amount}. New balance: $${this.balance}`);
      return true;
    } else {
      console.log(`Insufficient funds. Balance: $${this.balance}`);
      return false;
    }
  }
  
  deposit(amount) {
    this.balance += amount;
    console.log(`Deposited $${amount}. New balance: $${this.balance}`);
  }
}

// 保护代理
class ProtectedBankAccount extends BankAccount {
  constructor(realAccount, owner) {
    super();
    this.realAccount = realAccount;
    this.owner = owner;
    this.authorizedUsers = new Set([owner]);
  }
  
  authorizeUser(user) {
    this.authorizedUsers.add(user);
    console.log(`User ${user} authorized to access account`);
  }
  
  deauthorizeUser(user) {
    if (user !== this.owner) {
      this.authorizedUsers.delete(user);
      console.log(`User ${user} deauthorized from account`);
    } else {
      console.log(`Cannot deauthorize account owner`);
    }
  }
  
  getBalance(currentUser) {
    if (this.isAuthorized(currentUser)) {
      console.log(`User ${currentUser} accessed balance`);
      return this.realAccount.getBalance();
    } else {
      console.log(`Access denied for user ${currentUser}`);
      throw new Error("Unauthorized access");
    }
  }
  
  withdraw(amount, currentUser) {
    if (this.isAuthorized(currentUser)) {
      console.log(`User ${currentUser} attempting to withdraw $${amount}`);
      return this.realAccount.withdraw(amount);
    } else {
      console.log(`Withdrawal denied for user ${currentUser}`);
      throw new Error("Unauthorized access");
    }
  }
  
  deposit(amount, currentUser) {
    if (this.isAuthorized(currentUser)) {
      console.log(`User ${currentUser} attempting to deposit $${amount}`);
      return this.realAccount.deposit(amount);
    } else {
      console.log(`Deposit denied for user ${currentUser}`);
      throw new Error("Unauthorized access");
    }
  }
  
  isAuthorized(user) {
    return this.authorizedUsers.has(user);
  }
}
```

#### 第六步：使用保护代理

```javascript
// 使用保护代理
class BankingApp {
  constructor(account, currentUser) {
    this.account = account;
    this.currentUser = currentUser;
  }
  
  checkBalance() {
    try {
      const balance = this.account.getBalance(this.currentUser);
      console.log(`Current balance: $${balance}`);
      return balance;
    } catch (error) {
      console.log(`Error: ${error.message}`);
      return null;
    }
  }
  
  makeWithdrawal(amount) {
    try {
      return this.account.withdraw(amount, this.currentUser);
    } catch (error) {
      console.log(`Error: ${error.message}`);
      return false;
    }
  }
  
  makeDeposit(amount) {
    try {
      return this.account.deposit(amount, this.currentUser);
    } catch (error) {
      console.log(`Error: ${error.message}`);
      return false;
    }
  }
}

// 使用示例
const realAccount = new RealBankAccount("12345", 1000);
const protectedAccount = new ProtectedBankAccount(realAccount, "alice");

// 账户所有者操作
const ownerApp = new BankingApp(protectedAccount, "alice");
console.log("=== Account Owner Operations ===");
ownerApp.checkBalance();
ownerApp.makeDeposit(500);
ownerApp.makeWithdrawal(200);

// 授权用户操作
protectedAccount.authorizeUser("bob");
const authorizedUserApp = new BankingApp(protectedAccount, "bob");
console.log("\n=== Authorized User Operations ===");
authorizedUserApp.checkBalance();
authorizedUserApp.makeWithdrawal(100);

// 未授权用户操作
const unauthorizedUserApp = new BankingApp(protectedAccount, "charlie");
console.log("\n=== Unauthorized User Operations ===");
unauthorizedUserApp.checkBalance();
unauthorizedUserApp.makeWithdrawal(50);
```

#### 第七步：虚拟代理示例

```javascript
// 虚拟代理示例 - 延迟加载大型资源

// 主题接口
class Video {
  play() {
    throw new Error("play() method must be implemented");
  }
  
  pause() {
    throw new Error("pause() method must be implemented");
  }
  
  getDuration() {
    throw new Error("getDuration() method must be implemented");
  }
}

// 真实主题
class RealVideo extends Video {
  constructor(filename) {
    super();
    this.filename = filename;
    this.duration = 0;
    this.loadVideo();
  }
  
  loadVideo() {
    console.log(`Loading video: ${this.filename}`);
    // 模拟耗时的视频加载过程
    this.duration = Math.floor(Math.random() * 300) + 60; // 60-360秒
    console.log(`Video loaded. Duration: ${this.duration} seconds`);
  }
  
  play() {
    console.log(`Playing video: ${this.filename}`);
  }
  
  pause() {
    console.log(`Pausing video: ${this.filename}`);
  }
  
  getDuration() {
    return this.duration;
  }
}

// 虚拟代理
class VirtualVideoProxy extends Video {
  constructor(filename) {
    super();
    this.filename = filename;
    this.realVideo = null;
    this.loaded = false;
  }
  
  loadIfNeeded() {
    if (!this.loaded) {
      console.log("Loading video on demand...");
      this.realVideo = new RealVideo(this.filename);
      this.loaded = true;
    }
  }
  
  play() {
    this.loadIfNeeded();
    console.log("Playing video through proxy");
    this.realVideo.play();
  }
  
  pause() {
    if (this.loaded) {
      console.log("Pausing video through proxy");
      this.realVideo.pause();
    } else {
      console.log("Video not loaded yet, cannot pause");
    }
  }
  
  getDuration() {
    this.loadIfNeeded();
    return this.realVideo.getDuration();
  }
  
  isLoaded() {
    return this.loaded;
  }
}
```

#### 第八步：使用虚拟代理

```javascript
// 使用虚拟代理
class VideoPlayer {
  constructor(video) {
    this.video = video;
  }
  
  playVideo() {
    console.log("=== Playing Video ===");
    this.video.play();
  }
  
  pauseVideo() {
    console.log("=== Pausing Video ===");
    this.video.pause();
  }
  
  showDuration() {
    console.log("=== Video Duration ===");
    const duration = this.video.getDuration();
    console.log(`Duration: ${duration} seconds (${Math.floor(duration/60)}:${duration%60})`);
  }
}

// 立即加载视频
console.log("Creating real video (immediate loading):");
const realVideo = new RealVideo("movie.mp4");
const player1 = new VideoPlayer(realVideo);
player1.showDuration();
player1.playVideo();

console.log("\n--- Separator ---\n");

// 延迟加载视频
console.log("Creating virtual video proxy (lazy loading):");
const proxyVideo = new VirtualVideoProxy("movie2.mp4");
const player2 = new VideoPlayer(proxyVideo);

console.log("Before any operation:");
console.log(`Video loaded: ${proxyVideo.isLoaded()}`);

console.log("\nShowing duration (triggers loading):");
player2.showDuration();
console.log(`Video loaded: ${proxyVideo.isLoaded()}`);

console.log("\nPlaying video:");
player2.playVideo();
```

### 优势

1. **控制访问**：可以控制对真实对象的访问
2. **延迟初始化**：可以延迟创建开销大的对象
3. **增强功能**：可以在访问真实对象前后添加额外功能
4. **透明性**：客户端可以像使用真实对象一样使用代理

### 最佳实践

1. **明确代理类型**：根据需求选择合适的代理类型
2. **保持接口一致**：代理应该实现与真实对象相同的接口
3. **合理处理延迟**：在虚拟代理中合理处理延迟加载
4. **安全控制**：在保护代理中实现适当的安全检查

### 常见误区

1. **代理过于复杂**：代理类变得过于复杂，失去了代理的意义
2. **性能影响**：代理可能引入额外的性能开销
3. **接口不匹配**：代理和真实对象的接口不一致

## 总结

结构型设计模式为处理类或对象的组合提供了强大的解决方案。每种模式都有其特定的适用场景和优势：

1. **适配器模式** - 适用于接口不兼容的情况，使不相关的类可以协同工作
2. **桥接模式** - 适用于多个维度变化的复杂系统，将抽象与实现分离
3. **组合模式** - 适用于处理树形结构，使客户端可以一致地处理单个对象和组合对象
4. **装饰器模式** - 适用于动态地给对象添加职责，比继承更灵活
5. **外观模式** - 适用于为复杂子系统提供简单接口，隐藏子系统的复杂性
6. **享元模式** - 适用于大量细粒度对象的场景，通过共享减少内存使用
7. **代理模式** - 适用于控制对对象的访问，可以延迟加载、保护访问或增强功能

正确选择和使用这些模式可以大大提高代码的可维护性、可扩展性和复用性。但需要注意的是，不应为了使用模式而使用模式，应该根据具体问题选择最合适的设计模式。