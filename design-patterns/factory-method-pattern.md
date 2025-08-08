# 工厂方法模式 (Factory Method Pattern)

## 原始定义

工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法让类把实例化推迟到子类。

## 详细解释

工厂方法模式是一种创建型设计模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，而是引用一个共同的接口来指向新创建的对象。

工厂方法模式的核心思想是定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。这种模式创建一个工厂接口，让工厂的子类决定要实例化的产品类是哪一个。

### 核心概念

1. **创建接口**：定义一个用于创建对象的接口
2. **延迟实例化**：将对象的创建延迟到子类
3. **多态性**：通过继承实现不同的创建逻辑
4. **可扩展性**：易于添加新的产品类型

### 适用场景

1. **当一个类不知道它所必须创建的对象的类的时候**
2. **当一个类希望由它的子类来指定它所创建的对象的时候**
3. **当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候**
4. **当创建对象的过程比较复杂，需要根据不同的条件创建不同的对象时**

## 实施步骤

### 第一步：定义产品接口

首先定义产品的公共接口，所有具体产品都需要实现这个接口：

```javascript
// 产品接口 - 定义产品的公共接口
class PaymentProcessor {
  processPayment(amount) {
    throw new Error("processPayment() method must be implemented");
  }
  
  refundPayment(transactionId) {
    throw new Error("refundPayment() method must be implemented");
  }
  
  validatePaymentData(data) {
    throw new Error("validatePaymentData() method must be implemented");
  }
}
```

### 第二步：实现具体产品

创建实现产品接口的具体产品类：

```javascript
// 具体产品A - 信用卡支付处理器
class CreditCardProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing credit card payment of $${amount}`);
    // 实际的信用卡支付处理逻辑
    return {
      success: true,
      transactionId: `cc_${Date.now()}`,
      processor: "CreditCard"
    };
  }
  
  refundPayment(transactionId) {
    console.log(`Refunding credit card transaction ${transactionId}`);
    // 实际的信用卡退款逻辑
    return {
      success: true,
      refundedAmount: 0,
      processor: "CreditCard"
    };
  }
  
  validatePaymentData(data) {
    // 验证信用卡数据
    return data.cardNumber && data.expiryDate && data.cvv;
  }
}

// 具体产品B - PayPal支付处理器
class PayPalProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing PayPal payment of $${amount}`);
    // 实际的PayPal支付处理逻辑
    return {
      success: true,
      transactionId: `pp_${Date.now()}`,
      processor: "PayPal"
    };
  }
  
  refundPayment(transactionId) {
    console.log(`Refunding PayPal transaction ${transactionId}`);
    // 实际的PayPal退款逻辑
    return {
      success: true,
      refundedAmount: 0,
      processor: "PayPal"
    };
  }
  
  validatePaymentData(data) {
    // 验证PayPal数据
    return data.email && data.password;
  }
}

// 具体产品C - 银行转账支付处理器
class BankTransferProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing bank transfer payment of $${amount}`);
    // 实际的银行转账支付处理逻辑
    return {
      success: true,
      transactionId: `bt_${Date.now()}`,
      processor: "BankTransfer"
    };
  }
  
  refundPayment(transactionId) {
    console.log(`Refunding bank transfer transaction ${transactionId}`);
    // 实际的银行转账退款逻辑
    return {
      success: true,
      refundedAmount: 0,
      processor: "BankTransfer"
    };
  }
  
  validatePaymentData(data) {
    // 验证银行转账数据
    return data.accountNumber && data.routingNumber;
  }
}
```

### 第三步：定义工厂接口

定义创建产品的工厂接口：

```javascript
// 工厂接口 - 定义创建产品的接口
class PaymentProcessorFactory {
  createProcessor() {
    throw new Error("createProcessor() method must be implemented");
  }
  
  getProcessorType() {
    throw new Error("getProcessorType() method must be implemented");
  }
}
```

### 第四步：实现具体工厂

创建实现工厂接口的具体工厂类：

```javascript
// 具体工厂A - 信用卡支付处理器工厂
class CreditCardProcessorFactory extends PaymentProcessorFactory {
  createProcessor() {
    return new CreditCardProcessor();
  }
  
  getProcessorType() {
    return "CreditCard";
  }
}

// 具体工厂B - PayPal支付处理器工厂
class PayPalProcessorFactory extends PaymentProcessorFactory {
  createProcessor() {
    return new PayPalProcessor();
  }
  
  getProcessorType() {
    return "PayPal";
  }
}

// 具体工厂C - 银行转账支付处理器工厂
class BankTransferProcessorFactory extends PaymentProcessorFactory {
  createProcessor() {
    return new BankTransferProcessor();
  }
  
  getProcessorType() {
    return "BankTransfer";
  }
}
```

### 第五步：使用工厂方法模式

创建客户端代码来使用工厂方法模式：

```javascript
// 客户端代码
class PaymentService {
  constructor(factory) {
    this.factory = factory;
    this.processor = factory.createProcessor();
  }
  
  processPayment(amount, paymentData) {
    console.log(`=== Processing ${this.factory.getProcessorType()} Payment ===`);
    
    // 验证支付数据
    if (!this.processor.validatePaymentData(paymentData)) {
      throw new Error("Invalid payment data");
    }
    
    // 处理支付
    const result = this.processor.processPayment(amount);
    
    if (result.success) {
      console.log(`Payment successful. Transaction ID: ${result.transactionId}`);
      return result;
    } else {
      console.log("Payment failed");
      throw new Error("Payment processing failed");
    }
  }
  
  refundPayment(transactionId) {
    console.log(`=== Refunding ${this.factory.getProcessorType()} Payment ===`);
    
    const result = this.processor.refundPayment(transactionId);
    
    if (result.success) {
      console.log(`Refund successful.`);
      return result;
    } else {
      console.log("Refund failed");
      throw new Error("Refund processing failed");
    }
  }
}

// 使用示例
console.log("=== Credit Card Payment ===");
const creditCardFactory = new CreditCardProcessorFactory();
const creditCardService = new PaymentService(creditCardFactory);

try {
  const paymentResult = creditCardService.processPayment(99.99, {
    cardNumber: "1234-5678-9012-3456",
    expiryDate: "12/25",
    cvv: "123"
  });
  
  creditCardService.refundPayment(paymentResult.transactionId);
} catch (error) {
  console.log(`Error: ${error.message}`);
}

console.log("\n=== PayPal Payment ===");
const paypalFactory = new PayPalProcessorFactory();
const paypalService = new PaymentService(paypalFactory);

try {
  const paymentResult = paypalService.processPayment(149.99, {
    email: "user@example.com",
    password: "password123"
  });
  
  paypalService.refundPayment(paymentResult.transactionId);
} catch (error) {
  console.log(`Error: ${error.message}`);
}
```

### 第六步：参数化工厂方法

实现支持参数的工厂方法：

```javascript
// 参数化工厂接口
class ParameterizedPaymentProcessorFactory {
  createProcessor(processorType) {
    switch (processorType) {
      case "CreditCard":
        return new CreditCardProcessor();
      case "PayPal":
        return new PayPalProcessor();
      case "BankTransfer":
        return new BankTransferProcessor();
      default:
        throw new Error(`Unknown processor type: ${processorType}`);
    }
  }
  
  getSupportedProcessors() {
    return ["CreditCard", "PayPal", "BankTransfer"];
  }
}

// 使用参数化工厂
class FlexiblePaymentService {
  constructor() {
    this.factory = new ParameterizedPaymentProcessorFactory();
  }
  
  processPayment(processorType, amount, paymentData) {
    console.log(`=== Processing ${processorType} Payment ===`);
    
    try {
      const processor = this.factory.createProcessor(processorType);
      
      if (!processor.validatePaymentData(paymentData)) {
        throw new Error("Invalid payment data");
      }
      
      const result = processor.processPayment(amount);
      
      if (result.success) {
        console.log(`Payment successful. Transaction ID: ${result.transactionId}`);
        return result;
      } else {
        throw new Error("Payment processing failed");
      }
    } catch (error) {
      console.log(`Error: ${error.message}`);
      throw error;
    }
  }
}

// 使用参数化工厂
const flexibleService = new FlexiblePaymentService();

flexibleService.processPayment("CreditCard", 79.99, {
  cardNumber: "1234-5678-9012-3456",
  expiryDate: "12/25",
  cvv: "123"
});

flexibleService.processPayment("PayPal", 59.99, {
  email: "user@example.com",
  password: "password123"
});
```

### 第七步：模板方法与工厂方法结合

将工厂方法与模板方法模式结合使用：

```javascript
// 抽象创建者 - 结合模板方法模式
class PaymentProcessorCreator {
  // 模板方法
  processPaymentFlow(amount, paymentData) {
    console.log("=== Starting Payment Process ===");
    
    // 步骤1: 创建处理器
    const processor = this.createProcessor();
    
    // 步骤2: 验证数据
    if (!this.validatePaymentData(processor, paymentData)) {
      throw new Error("Invalid payment data");
    }
    
    // 步骤3: 处理前准备
    this.beforeProcess(processor, amount, paymentData);
    
    // 步骤4: 执行支付
    const result = this.executePayment(processor, amount);
    
    // 步骤5: 处理后操作
    this.afterProcess(processor, result);
    
    console.log("=== Payment Process Completed ===");
    return result;
  }
  
  // 工厂方法 - 子类必须实现
  createProcessor() {
    throw new Error("createProcessor() method must be implemented");
  }
  
  // 钩子方法 - 可以被子类重写
  validatePaymentData(processor, paymentData) {
    return processor.validatePaymentData(paymentData);
  }
  
  // 钩子方法 - 可以被子类重写
  beforeProcess(processor, amount, paymentData) {
    console.log("Preparing for payment processing...");
  }
  
  // 具体方法 - 执行支付
  executePayment(processor, amount) {
    return processor.processPayment(amount);
  }
  
  // 钩子方法 - 可以被子类重写
  afterProcess(processor, result) {
    if (result.success) {
      console.log(`Payment completed successfully with ${processor.constructor.name}`);
    }
  }
}

// 具体创建者A
class CreditCardProcessorCreator extends PaymentProcessorCreator {
  createProcessor() {
    return new CreditCardProcessor();
  }
  
  beforeProcess(processor, amount, paymentData) {
    console.log("Validating credit card information...");
    console.log("Checking card limits...");
  }
  
  afterProcess(processor, result) {
    super.afterProcess(processor, result);
    if (result.success) {
      console.log("Sending receipt email...");
      console.log("Updating loyalty points...");
    }
  }
}

// 具体创建者B
class PayPalProcessorCreator extends PaymentProcessorCreator {
  createProcessor() {
    return new PayPalProcessor();
  }
  
  beforeProcess(processor, amount, paymentData) {
    console.log("Authenticating with PayPal...");
    console.log("Checking PayPal balance...");
  }
  
  afterProcess(processor, result) {
    super.afterProcess(processor, result);
    if (result.success) {
      console.log("Sending PayPal notification...");
      console.log("Updating PayPal transaction history...");
    }
  }
}
```

### 第八步：使用模板方法与工厂方法结合

```javascript
// 使用结合了模板方法的工厂方法
console.log("=== Credit Card Payment with Template Method ===");
const creditCardCreator = new CreditCardProcessorCreator();
try {
  const result = creditCardCreator.processPaymentFlow(199.99, {
    cardNumber: "1234-5678-9012-3456",
    expiryDate: "12/25",
    cvv: "123"
  });
  console.log("Final result:", result);
} catch (error) {
  console.log(`Error: ${error.message}`);
}

console.log("\n=== PayPal Payment with Template Method ===");
const paypalCreator = new PayPalProcessorCreator();
try {
  const result = paypalCreator.processPaymentFlow(149.99, {
    email: "user@example.com",
    password: "password123"
  });
  console.log("Final result:", result);
} catch (error) {
  console.log(`Error: ${error.message}`);
}
```

## 优势

1. **符合开闭原则**：新增产品类时无需修改现有代码，只需添加新的具体工厂类
2. **符合单一职责原则**：每个工厂类只负责一种产品的创建，职责清晰
3. **符合依赖倒置原则**：客户端依赖于抽象工厂接口，而不是具体工厂类
4. **良好的封装性**：客户端不需要知道所创建的具体产品类，只需要知道产品的接口
5. **扩展性好**：增加新产品时，只需增加相应的具体工厂类和具体产品类

## 最佳实践

1. **将产品的创建延迟到子类**：工厂方法模式的核心思想是让子类决定创建哪种产品
2. **使用工厂方法替代直接的类实例化**：提高系统的可扩展性和可维护性
3. **为每种产品创建对应的工厂子类**：保持代码的清晰性和可读性
4. **考虑使用配置文件**：可以结合配置文件来决定实例化哪一个具体类
5. **合理使用模板方法**：将工厂方法与模板方法结合，可以更好地控制创建过程

## 常见误区

1. **过度设计**：对于简单对象创建也使用工厂方法，增加了不必要的复杂性
2. **工厂类过多**：可能导致系统中类的数量急剧增加，增加维护成本
3. **客户端需要知道具体工厂类**：客户端需要知道使用哪个工厂类来创建对象，增加了耦合
4. **违反里氏替换原则**：如果具体产品类之间差异过大，可能难以替换使用

## 实际应用示例

### HTTP客户端工厂

```javascript
// HTTP客户端产品接口
class HttpClient {
  get(url, options) {
    throw new Error("get() method must be implemented");
  }
  
  post(url, data, options) {
    throw new Error("post() method must be implemented");
  }
}

// 具体产品 - Fetch客户端
class FetchHttpClient extends HttpClient {
  async get(url, options = {}) {
    console.log(`Fetching GET ${url} with Fetch API`);
    const response = await fetch(url, { method: 'GET', ...options });
    return response.json();
  }
  
  async post(url, data, options = {}) {
    console.log(`Fetching POST ${url} with Fetch API`);
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' },
      ...options
    });
    return response.json();
  }
}

// 具体产品 - XMLHttpRequest客户端
class XhrHttpClient extends HttpClient {
  get(url, options = {}) {
    console.log(`Fetching GET ${url} with XMLHttpRequest`);
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.open('GET', url);
      xhr.onload = () => resolve(JSON.parse(xhr.responseText));
      xhr.onerror = () => reject(new Error('Request failed'));
      xhr.send();
    });
  }
  
  post(url, data, options = {}) {
    console.log(`Fetching POST ${url} with XMLHttpRequest`);
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.open('POST', url);
      xhr.setRequestHeader('Content-Type', 'application/json');
      xhr.onload = () => resolve(JSON.parse(xhr.responseText));
      xhr.onerror = () => reject(new Error('Request failed'));
      xhr.send(JSON.stringify(data));
    });
  }
}

// 工厂接口
class HttpClientFactory {
  createClient() {
    throw new Error("createClient() method must be implemented");
  }
}

// 具体工厂 - Fetch客户端工厂
class FetchHttpClientFactory extends HttpClientFactory {
  createClient() {
    return new FetchHttpClient();
  }
}

// 具体工厂 - XHR客户端工厂
class XhrHttpClientFactory extends HttpClientFactory {
  createClient() {
    return new XhrHttpClient();
  }
}

// 使用示例
class ApiService {
  constructor(factory) {
    this.factory = factory;
    this.client = factory.createClient();
  }
  
  async fetchUserData(userId) {
    return this.client.get(`/api/users/${userId}`);
  }
  
  async createUser(userData) {
    return this.client.post('/api/users', userData);
  }
}

// 根据环境选择客户端
const isModernBrowser = 'fetch' in window;
const factory = isModernBrowser 
  ? new FetchHttpClientFactory() 
  : new XhrHttpClientFactory();
  
const apiService = new ApiService(factory);
```

### 数据库连接工厂

```javascript
// 数据库连接产品接口
class DatabaseConnection {
  connect() {
    throw new Error("connect() method must be implemented");
  }
  
  query(sql) {
    throw new Error("query() method must be implemented");
  }
  
  close() {
    throw new Error("close() method must be implemented");
  }
}

// 具体产品 - MySQL连接
class MySqlConnection extends DatabaseConnection {
  constructor(config) {
    super();
    this.config = config;
    this.connected = false;
  }
  
  connect() {
    console.log(`Connecting to MySQL database ${this.config.database}`);
    // 实际的连接逻辑
    this.connected = true;
    return this;
  }
  
  query(sql) {
    if (!this.connected) {
      throw new Error("Not connected to database");
    }
    console.log(`Executing MySQL query: ${sql}`);
    return { rows: [], affectedRows: 0 };
  }
  
  close() {
    console.log("Closing MySQL connection");
    this.connected = false;
  }
}

// 具体产品 - PostgreSQL连接
class PostgresConnection extends DatabaseConnection {
  constructor(config) {
    super();
    this.config = config;
    this.connected = false;
  }
  
  connect() {
    console.log(`Connecting to PostgreSQL database ${this.config.database}`);
    // 实际的连接逻辑
    this.connected = true;
    return this;
  }
  
  query(sql) {
    if (!this.connected) {
      throw new Error("Not connected to database");
    }
    console.log(`Executing PostgreSQL query: ${sql}`);
    return { rows: [], rowCount: 0 };
  }
  
  close() {
    console.log("Closing PostgreSQL connection");
    this.connected = false;
  }
}

// 工厂接口
class DatabaseConnectionFactory {
  createConnection(config) {
    throw new Error("createConnection() method must be implemented");
  }
}

// 具体工厂 - MySQL连接工厂
class MySqlConnectionFactory extends DatabaseConnectionFactory {
  createConnection(config) {
    return new MySqlConnection(config);
  }
}

// 具体工厂 - PostgreSQL连接工厂
class PostgresConnectionFactory extends DatabaseConnectionFactory {
  createConnection(config) {
    return new PostgresConnection(config);
  }
}

// 使用示例
class DatabaseService {
  constructor(factory, config) {
    this.factory = factory;
    this.config = config;
    this.connection = null;
  }
  
  async connect() {
    this.connection = this.factory.createConnection(this.config);
    return this.connection.connect();
  }
  
  async executeQuery(sql) {
    if (!this.connection) {
      await this.connect();
    }
    return this.connection.query(sql);
  }
  
  async disconnect() {
    if (this.connection) {
      this.connection.close();
    }
  }
}

// 根据配置选择数据库
const dbConfig = {
  type: 'mysql', // 或 'postgres'
  host: 'localhost',
  port: 3306,
  database: 'myapp',
  username: 'user',
  password: 'password'
};

const factory = dbConfig.type === 'mysql' 
  ? new MySqlConnectionFactory() 
  : new PostgresConnectionFactory();
  
const dbService = new DatabaseService(factory, dbConfig);
```

## 总结

工厂方法模式是一种非常实用的创建型设计模式，它通过将对象的创建延迟到子类来实现解耦。这种模式特别适用于以下场景：

1. **需要创建多种类型的对象**：当系统需要根据不同的条件创建不同类型的对象时
2. **希望将对象的创建与使用分离**：让客户端代码不依赖于具体的产品类
3. **需要扩展产品类型**：当需要添加新的产品类型时，只需添加相应的工厂类

工厂方法模式的主要优势是符合开闭原则，新增产品时无需修改现有代码。但也要注意避免过度使用，对于简单的对象创建，直接实例化可能更加合适。

正确使用工厂方法模式可以提高代码的可维护性、可扩展性和可测试性，是面向对象设计中的重要工具。