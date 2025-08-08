# 创建型设计模式 (Creational Design Patterns)

## 原始定义

创建型设计模式是处理对象创建机制的设计模式，试图以适合当前情况的方式创建对象。基本的对象创建方式可能会给设计带来问题，或者增加设计的复杂性。创建型设计模式通过控制对象的创建过程来解决这个问题。

## 详细解释

创建型设计模式关注对象的创建机制，它们试图以适合当前情况的方式创建对象。在软件开发中，对象的创建可能会带来复杂性，特别是当系统需要创建多种类型的对象或者对象创建过程比较复杂时。创建型模式通过将对象的创建与使用分离，提供更大的灵活性和可维护性。

### 核心概念

1. **封装变化**：将对象的创建过程封装起来，使得系统更容易修改和扩展
2. **依赖解耦**：减少客户端代码与具体类之间的紧耦合
3. **可扩展性**：使得添加新的对象类型更容易
4. **复用性**：通过标准的创建接口提高代码复用性

### 创建型模式分类

创建型模式包括以下五种模式：
1. 单例模式 (Singleton Pattern)
2. 工厂方法模式 (Factory Method Pattern)
3. 抽象工厂模式 (Abstract Factory Pattern)
4. 建造者模式 (Builder Pattern)
5. 原型模式 (Prototype Pattern)

## 1. 单例模式 (Singleton Pattern)

### 原始定义

单例模式确保一个类只有一个实例，并提供一个全局访问点。

### 详细解释

单例模式是最简单但也是最容易被误用的设计模式之一。它限制一个类只能创建一个实例，并提供一个全局访问点来获取这个实例。单例模式通常用于控制资源的访问，如数据库连接、日志记录器等。

### 适用场景

1. 需要频繁实例化然后销毁的对象
2. 创建对象时耗时过多或耗资源过多，但又经常用到的对象
3. 有状态的工具类对象
4. 频繁访问数据库或文件的对象

### 实施步骤

#### 第一步：基础单例实现

```javascript
// JavaScript中的单例模式实现
class Singleton {
  constructor() {
    // 防止通过new直接创建实例
    if (Singleton.instance) {
      return Singleton.instance;
    }
    
    // 初始化实例
    this.data = [];
    Singleton.instance = this;
    return this;
  }
  
  // 业务方法
  addData(item) {
    this.data.push(item);
  }
  
  getData() {
    return this.data;
  }
}

// 使用示例
const singleton1 = new Singleton();
const singleton2 = new Singleton();
console.log(singleton1 === singleton2); // true
```

#### 第二步：模块模式实现单例

```javascript
// 使用模块模式实现单例
const Logger = (function() {
  let instance;
  
  function createInstance() {
    return {
      log: function(message) {
        console.log(`[${new Date().toISOString()}] ${message}`);
      },
      
      error: function(message) {
        console.error(`[${new Date().toISOString()}] ERROR: ${message}`);
      }
    };
  }
  
  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// 使用示例
const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();
console.log(logger1 === logger2); // true
```

#### 第三步：ES6模块实现单例

```javascript
// logger.js - 使用ES6模块实现单例
class Logger {
  constructor() {
    if (Logger.instance) {
      return Logger.instance;
    }
    
    this.logs = [];
    Logger.instance = this;
    return this;
  }
  
  log(message) {
    this.logs.push({
      timestamp: new Date(),
      message: message,
      level: 'info'
    });
    console.log(`[INFO] ${message}`);
  }
  
  error(message) {
    this.logs.push({
      timestamp: new Date(),
      message: message,
      level: 'error'
    });
    console.error(`[ERROR] ${message}`);
  }
  
  getLogs() {
    return this.logs;
  }
}

const logger = new Logger();
Object.freeze(logger); // 冻结对象，防止修改

export default logger;
```

### 优势

1. **内存中只有一个实例**：减少了内存开销
2. **避免对资源的多重占用**：如文件操作、数据库连接等
3. **设置全局访问点**：便于严格控制客户对它的访问
4. **允许对操作和表示的精化**：可以有子类化的版本

### 最佳实践

1. **谨慎使用**：单例模式容易被滥用，应谨慎使用
2. **线程安全**：在多线程环境中需要考虑线程安全问题
3. **延迟初始化**：可以采用延迟初始化来提高性能
4. **避免全局状态**：单例容易创建全局状态，应尽量避免

### 常见误区

1. **过度使用**：将所有类都设计为单例
2. **隐藏依赖**：单例隐藏了类之间的依赖关系
3. **测试困难**：单例模式使得单元测试变得困难
4. **违反单一职责原则**：单例类既要管理自己的实例，又要处理业务逻辑

## 2. 工厂方法模式 (Factory Method Pattern)

### 原始定义

工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法让类把实例化推迟到子类。

### 详细解释

工厂方法模式是一种创建型设计模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，而是引用一个共同的接口来指向新创建的对象。

### 适用场景

1. 当一个类不知道它所必须创建的对象的类的时候
2. 当一个类希望由它的子类来指定它所创建的对象的时候
3. 当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候

### 实施步骤

#### 第一步：定义产品接口

```javascript
// 产品接口
class HttpRequest {
  constructor(url, method) {
    this.url = url;
    this.method = method;
  }
  
  send() {
    throw new Error("send() method must be implemented");
  }
}
```

#### 第二步：实现具体产品

```javascript
// 具体产品
class GetRequest extends HttpRequest {
  constructor(url) {
    super(url, 'GET');
  }
  
  send() {
    console.log(`Sending GET request to ${this.url}`);
    // 实际的GET请求逻辑
    return fetch(this.url, { method: this.method });
  }
}

class PostRequest extends HttpRequest {
  constructor(url, data) {
    super(url, 'POST');
    this.data = data;
  }
  
  send() {
    console.log(`Sending POST request to ${this.url} with data:`, this.data);
    // 实际的POST请求逻辑
    return fetch(this.url, {
      method: this.method,
      body: JSON.stringify(this.data),
      headers: {
        'Content-Type': 'application/json'
      }
    });
  }
}
```

#### 第三步：定义工厂接口

```javascript
// 工厂接口
class RequestFactory {
  createRequest(url, options) {
    throw new Error("createRequest() method must be implemented");
  }
}
```

#### 第四步：实现具体工厂

```javascript
// 具体工厂
class HttpRequestFactory extends RequestFactory {
  createRequest(url, options = {}) {
    const { method = 'GET', data } = options;
    
    switch (method.toUpperCase()) {
      case 'GET':
        return new GetRequest(url);
      case 'POST':
        return new PostRequest(url, data);
      case 'PUT':
        // 可以添加PUT请求的实现
        throw new Error("PUT method not implemented yet");
      case 'DELETE':
        // 可以添加DELETE请求的实现
        throw new Error("DELETE method not implemented yet");
      default:
        throw new Error(`Unsupported HTTP method: ${method}`);
    }
  }
}
```

#### 第五步：使用工厂

```javascript
// 使用示例
const factory = new HttpRequestFactory();

// 创建GET请求
const getRequest = factory.createRequest('https://api.example.com/users');
getRequest.send();

// 创建POST请求
const postRequest = factory.createRequest('https://api.example.com/users', {
  method: 'POST',
  data: { name: 'John', email: 'john@example.com' }
});
postRequest.send();
```

### 优势

1. **符合开闭原则**：新增产品类时无需修改现有代码
2. **符合单一职责原则**：每个工厂类只负责一种产品的创建
3. **符合依赖倒置原则**：客户端依赖于抽象而不是具体类
4. **良好的封装性**：客户端不需要知道所创建的具体产品类

### 最佳实践

1. **将产品的创建延迟到子类**：工厂方法模式的核心思想
2. **使用工厂方法替代直接的类实例化**：提高系统的可扩展性
3. **为每种产品创建对应的工厂子类**：保持代码的清晰性
4. **考虑使用配置文件**：可以结合配置文件来决定实例化哪一个具体类

### 常见误区

1. **过度设计**：对于简单对象创建也使用工厂方法
2. **工厂类过多**：可能导致系统中类的数量急剧增加
3. **客户端需要知道具体工厂类**：客户端需要知道使用哪个工厂类来创建对象

## 3. 抽象工厂模式 (Abstract Factory Pattern)

### 原始定义

抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

### 详细解释

抽象工厂模式是一种创建型设计模式，它能创建一系列相关的对象，而无需指定其具体类。抽象工厂定义了一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。

### 适用场景

1. 系统要独立于其产品的创建、组合和表示时
2. 系统要由多个产品系列中的一个来配置时
3. 需要强调一系列相关的产品对象的设计以便进行联合使用时
4. 提供一个产品类库，而只想显示它们的接口而不是实现时

### 实施步骤

#### 第一步：定义抽象产品

```javascript
// 抽象产品A
class Button {
  render() {
    throw new Error("render() method must be implemented");
  }
}

// 抽象产品B
class Checkbox {
  render() {
    throw new Error("render() method must be implemented");
  }
}
```

#### 第二步：实现具体产品

```javascript
// 具体产品A1
class WindowsButton extends Button {
  render() {
    console.log("Rendering Windows style button");
    // Windows按钮的具体实现
  }
}

// 具体产品A2
class MacButton extends Button {
  render() {
    console.log("Rendering Mac style button");
    // Mac按钮的具体实现
  }
}

// 具体产品B1
class WindowsCheckbox extends Checkbox {
  render() {
    console.log("Rendering Windows style checkbox");
    // Windows复选框的具体实现
  }
}

// 具体产品B2
class MacCheckbox extends Checkbox {
  render() {
    console.log("Rendering Mac style checkbox");
    // Mac复选框的具体实现
  }
}
```

#### 第三步：定义抽象工厂

```javascript
// 抽象工厂
class UIFactory {
  createButton() {
    throw new Error("createButton() method must be implemented");
  }
  
  createCheckbox() {
    throw new Error("createCheckbox() method must be implemented");
  }
}
```

#### 第四步：实现具体工厂

```javascript
// 具体工厂1
class WindowsFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
  
  createCheckbox() {
    return new WindowsCheckbox();
  }
}

// 具体工厂2
class MacFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }
  
  createCheckbox() {
    return new MacCheckbox();
  }
}
```

#### 第五步：使用抽象工厂

```javascript
// 应用程序类
class Application {
  constructor(factory) {
    this.factory = factory;
  }
  
  createUI() {
    const button = this.factory.createButton();
    const checkbox = this.factory.createCheckbox();
    
    button.render();
    checkbox.render();
  }
}

// 使用示例
const os = process.platform; // 假设这是获取操作系统的方法
let factory;

if (os === 'win32') {
  factory = new WindowsFactory();
} else {
  factory = new MacFactory();
}

const app = new Application(factory);
app.createUI();
```

### 优势

1. **易于交换产品系列**：只需改变具体的工厂即可
2. **有利于产品的一致性**：确保同一工厂生产的产品是相互匹配的
3. **符合开闭原则**：增加新的产品系列时无需修改原有代码

### 最佳实践

1. **产品族必须一起使用**：确保相关的产品对象一起使用
2. **系统需要配置多个产品系列**：适用于需要支持多种外观或主题的系统
3. **强调产品的接口而非实现**：客户端只依赖于抽象接口

### 常见误区

1. **支持新种类的产品困难**：增加新的产品等级结构麻烦
2. **工厂类层次过于复杂**：可能导致系统中类的数量急剧增加
3. **客户端需要知道具体工厂类**：客户端需要知道使用哪个工厂类

## 4. 建造者模式 (Builder Pattern)

### 原始定义

建造者模式将一个复杂对象的构建与其表示分离，使得同样的构建过程可以创建不同的表示。

### 详细解释

建造者模式是一种创建型设计模式，它能将复杂对象的构建过程与其表示分离，这样同样的构建过程可以创建不同的表示。建造者模式主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的过程构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的过程却相对稳定。

### 适用场景

1. 需要生成的对象具有复杂的内部结构
2. 需要生成的对象的属性相互依赖，需要指定其生成顺序
3. 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品

### 实施步骤

#### 第一步：定义产品类

```javascript
// 产品类
class HttpRequestConfig {
  constructor() {
    this.method = 'GET';
    this.url = '';
    this.headers = {};
    this.body = null;
    this.timeout = 5000;
    this.retries = 0;
  }
  
  setMethod(method) {
    this.method = method;
    return this;
  }
  
  setUrl(url) {
    this.url = url;
    return this;
  }
  
  addHeader(key, value) {
    this.headers[key] = value;
    return this;
  }
  
  setBody(body) {
    this.body = body;
    return this;
  }
  
  setTimeout(timeout) {
    this.timeout = timeout;
    return this;
  }
  
  setRetries(retries) {
    this.retries = retries;
    return this;
  }
  
  build() {
    return {
      method: this.method,
      url: this.url,
      headers: this.headers,
      body: this.body,
      timeout: this.timeout,
      retries: this.retries
    };
  }
}
```

#### 第二步：定义抽象建造者

```javascript
// 抽象建造者
class RequestBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.request = new HttpRequestConfig();
  }
  
  setMethod(method) {
    throw new Error("setMethod() method must be implemented");
  }
  
  setUrl(url) {
    throw new Error("setUrl() method must be implemented");
  }
  
  addHeader(key, value) {
    throw new Error("addHeader() method must be implemented");
  }
  
  setBody(body) {
    throw new Error("setBody() method must be implemented");
  }
  
  setTimeout(timeout) {
    throw new Error("setTimeout() method must be implemented");
  }
  
  setRetries(retries) {
    throw new Error("setRetries() method must be implemented");
  }
  
  getResult() {
    const result = this.request;
    this.reset();
    return result.build();
  }
}
```

#### 第三步：实现具体建造者

```javascript
// 具体建造者
class HttpRequestBuilder extends RequestBuilder {
  setMethod(method) {
    this.request.setMethod(method);
    return this;
  }
  
  setUrl(url) {
    this.request.setUrl(url);
    return this;
  }
  
  addHeader(key, value) {
    this.request.addHeader(key, value);
    return this;
  }
  
  setBody(body) {
    this.request.setBody(body);
    return this;
  }
  
  setTimeout(timeout) {
    this.request.setTimeout(timeout);
    return this;
  }
  
  setRetries(retries) {
    this.request.setRetries(retries);
    return this;
  }
}
```

#### 第四步：定义指挥者

```javascript
// 指挥者
class RequestDirector {
  static constructGetRequest(builder, url) {
    return builder
      .setMethod('GET')
      .setUrl(url)
      .addHeader('Accept', 'application/json')
      .setTimeout(5000);
  }
  
  static constructPostRequest(builder, url, data) {
    return builder
      .setMethod('POST')
      .setUrl(url)
      .addHeader('Content-Type', 'application/json')
      .setBody(JSON.stringify(data))
      .setTimeout(10000)
      .setRetries(3);
  }
  
  static constructApiRequest(builder, url, token) {
    return builder
      .setMethod('GET')
      .setUrl(url)
      .addHeader('Authorization', `Bearer ${token}`)
      .addHeader('Accept', 'application/json')
      .setTimeout(8000);
  }
}
```

#### 第五步：使用建造者模式

```javascript
// 使用示例
const builder = new HttpRequestBuilder();

// 构建GET请求
const getRequest = RequestDirector.constructGetRequest(builder, 'https://api.example.com/users');
console.log(getRequest.getResult());

// 构建POST请求
const postRequest = RequestDirector.constructPostRequest(
  builder, 
  'https://api.example.com/users', 
  { name: 'John', email: 'john@example.com' }
);
console.log(postRequest.getResult());

// 构建API请求
const apiRequest = RequestDirector.constructApiRequest(
  builder, 
  'https://api.example.com/profile', 
  'your-jwt-token'
);
console.log(apiRequest.getResult());
```

### 优势

1. **客户端不必知道产品内部组成的细节**：将产品本身与产品的创建过程解耦
2. **每一个具体建造者都相对独立，而与其他的具体建造者无关**：可以很方便地替换具体建造者或增加新的具体建造者
3. **可以更加精细地控制产品的创建过程**：将复杂产品的创建步骤分解在不同的方法中

### 最佳实践

1. **使用指挥者类封装构建过程**：将构建步骤封装在指挥者中
2. **支持不同的表示**：同一个构建过程可以创建不同的产品表示
3. **提供获取结果的方法**：建造者应该提供获取最终产品的接口

### 常见误区

1. **过度设计**：对于简单对象也使用建造者模式
2. **建造者类过多**：可能导致系统中类的数量增加
3. **产品差异很大**：如果产品差异很大，不适合使用建造者模式

## 5. 原型模式 (Prototype Pattern)

### 原始定义

原型模式用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

### 详细解释

原型模式是一种创建型设计模式，它允许一个对象通过复制现有实例来创建新的实例，而不是通过实例化类。原型模式通过复制现有对象来创建新对象，避免了重复的初始化过程。

### 适用场景

1. 当一个系统应该独立于它的产品创建、构成和表示时
2. 当要实例化的类是在运行时刻指定时，例如，通过动态装载
3. 为了避免创建一个与产品类层次平行的工厂类层次时
4. 当一个类的实例只能有几个不同状态组合中的一种时

### 实施步骤

#### 第一步：定义原型接口

```javascript
// 原型接口
class Prototype {
  clone() {
    throw new Error("clone() method must be implemented");
  }
}
```

#### 第二步：实现具体原型

```javascript
// 具体原型
class HttpRequestPrototype extends Prototype {
  constructor(method = 'GET', url = '', headers = {}, body = null) {
    super();
    this.method = method;
    this.url = url;
    this.headers = { ...headers }; // 浅拷贝
    this.body = body;
    this.timestamp = new Date();
  }
  
  clone() {
    // 创建当前对象的浅拷贝
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    // 创建当前对象的深拷贝
    const clone = new HttpRequestPrototype();
    clone.method = this.method;
    clone.url = this.url;
    clone.headers = JSON.parse(JSON.stringify(this.headers));
    clone.body = typeof this.body === 'object' 
      ? JSON.parse(JSON.stringify(this.body)) 
      : this.body;
    clone.timestamp = new Date();
    return clone;
  }
  
  setMethod(method) {
    this.method = method;
    return this;
  }
  
  setUrl(url) {
    this.url = url;
    return this;
  }
  
  addHeader(key, value) {
    this.headers[key] = value;
    return this;
  }
  
  setBody(body) {
    this.body = body;
    return this;
  }
  
  toString() {
    return `${this.method} ${this.url}`;
  }
}
```

#### 第三步：实现原型管理器

```javascript
// 原型管理器
class PrototypeManager {
  constructor() {
    this.prototypes = new Map();
  }
  
  // 注册原型
  registerPrototype(name, prototype) {
    this.prototypes.set(name, prototype);
  }
  
  // 注销原型
  unregisterPrototype(name) {
    this.prototypes.delete(name);
  }
  
  // 获取原型并克隆
  createPrototype(name) {
    const prototype = this.prototypes.get(name);
    if (!prototype) {
      throw new Error(`Prototype '${name}' not found`);
    }
    
    return prototype.clone();
  }
  
  // 获取原型并深克隆
  createPrototypeDeep(name) {
    const prototype = this.prototypes.get(name);
    if (!prototype) {
      throw new Error(`Prototype '${name}' not found`);
    }
    
    return prototype.cloneDeep();
  }
}
```

#### 第四步：使用原型模式

```javascript
// 创建原型管理器
const prototypeManager = new PrototypeManager();

// 注册原型
const baseGetRequest = new HttpRequestPrototype('GET', '', {
  'Accept': 'application/json',
  'User-Agent': 'HackBar/1.0'
});

const basePostRequest = new HttpRequestPrototype('POST', '', {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'User-Agent': 'HackBar/1.0'
});

prototypeManager.registerPrototype('GET', baseGetRequest);
prototypeManager.registerPrototype('POST', basePostRequest);

// 使用原型创建新对象
const userRequest = prototypeManager.createPrototype('GET')
  .setUrl('https://api.example.com/users')
  .addHeader('Authorization', 'Bearer token123');

const productRequest = prototypeManager.createPrototype('POST')
  .setUrl('https://api.example.com/products')
  .setBody({ name: 'Product 1', price: 99.99 })
  .addHeader('X-API-Key', 'api-key-123');

console.log(userRequest.toString()); // GET https://api.example.com/users
console.log(productRequest.toString()); // POST https://api.example.com/products
```

#### 第五步：高级原型实现

```javascript
// 支持继承的原型实现
class ApiRequestPrototype extends HttpRequestPrototype {
  constructor(method, url, headers, body) {
    super(method, url, headers, body);
    this.apiVersion = 'v1';
    this.authType = 'Bearer';
  }
  
  clone() {
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    const clone = new ApiRequestPrototype();
    clone.method = this.method;
    clone.url = this.url;
    clone.headers = JSON.parse(JSON.stringify(this.headers));
    clone.body = typeof this.body === 'object' 
      ? JSON.parse(JSON.stringify(this.body)) 
      : this.body;
    clone.timestamp = new Date();
    clone.apiVersion = this.apiVersion;
    clone.authType = this.authType;
    return clone;
  }
  
  setApiVersion(version) {
    this.apiVersion = version;
    return this;
  }
  
  setAuthType(type) {
    this.authType = type;
    return this;
  }
  
  toString() {
    return `[${this.apiVersion}] ${super.toString()}`;
  }
}

// 注册API请求原型
const apiRequest = new ApiRequestPrototype();
prototypeManager.registerPrototype('API', apiRequest);

// 使用API原型
const apiUserRequest = prototypeManager.createPrototype('API')
  .setMethod('GET')
  .setUrl('https://api.example.com/users')
  .setApiVersion('v2')
  .addHeader('Authorization', 'Bearer token456');

console.log(apiUserRequest.toString()); // [v2] GET https://api.example.com/users
```

### 优势

1. **运行时增加和删除产品**：可以在运行时通过注册原型来增加或删除产品
2. **改变值以指定新对象**：通过修改实例的值来指定新对象
3. **减少子类的构造**：用类动态配置应用
4. **用类动态配置应用**：运行时通过装配部件来定义新类

### 最佳实践

1. **实现深拷贝和浅拷贝**：根据需求选择合适的克隆方式
2. **使用原型管理器**：集中管理原型对象
3. **初始化原型对象**：为原型对象设置合适的默认值

### 常见误区

1. **克隆复杂对象困难**：循环引用和深层嵌套对象的克隆可能很复杂
2. **深拷贝性能问题**：深拷贝大型对象可能影响性能
3. **原型污染**：不正确的原型使用可能导致安全问题

## 总结

创建型设计模式为对象的创建提供了灵活而强大的解决方案。每种模式都有其特定的适用场景和优势：

1. **单例模式** - 适用于确保一个类只有一个实例的场景
2. **工厂方法模式** - 适用于需要将对象创建延迟到子类的场景
3. **抽象工厂模式** - 适用于需要创建一系列相关产品对象的场景
4. **建造者模式** - 适用于构建复杂对象的场景
5. **原型模式** - 适用于通过复制现有对象来创建新对象的场景

正确选择和使用这些模式可以大大提高代码的可维护性、可扩展性和复用性。但需要注意的是，不应为了使用模式而使用模式，应该根据具体问题选择最合适的设计模式。