# 单例模式 (Singleton Pattern)

## 原始定义

单例模式确保一个类只有一个实例，并提供一个全局访问点。

## 详细解释

单例模式是最简单但也是最容易被误用的设计模式之一。它属于创建型模式，限制一个类只能创建一个实例，并提供一个全局访问点来获取这个实例。单例模式通常用于控制资源的访问，如数据库连接、日志记录器、配置管理器等。

单例模式的核心思想是让类自身负责保存它的唯一实例，并提供一个全局访问点。这个模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。

### 核心概念

1. **唯一实例**：确保一个类只有一个实例
2. **全局访问点**：提供一个访问其唯一实例的全局访问点
3. **延迟初始化**：在需要时才创建实例
4. **线程安全**：在多线程环境中保证实例的唯一性

### 适用场景

1. **资源共享**：当需要频繁创建和销毁的对象，如数据库连接池
2. **控制资源**：当需要控制对某些资源的访问，如日志文件、打印机等
3. **全局状态**：当需要维护全局状态或配置信息时
4. **工具类**：当类只需要一个实例，如计算器、格式化工具等

## 实施步骤

### 第一步：基础单例实现

最基本的单例实现方式是通过构造函数控制实例的创建：

```javascript
// JavaScript中的基础单例模式实现
class Singleton {
  constructor() {
    // 检查是否已经存在实例
    if (Singleton.instance) {
      return Singleton.instance;
    }
    
    // 初始化实例属性
    this.data = [];
    this.createdAt = new Date();
    
    // 保存实例引用
    Singleton.instance = this;
    
    // 返回实例
    return this;
  }
  
  // 业务方法
  addData(item) {
    this.data.push(item);
  }
  
  getData() {
    return this.data;
  }
  
  getInfo() {
    return {
      createdAt: this.createdAt,
      dataCount: this.data.length
    };
  }
}

// 使用示例
const singleton1 = new Singleton();
const singleton2 = new Singleton();

console.log(singleton1 === singleton2); // true
console.log(singleton1.getInfo()); // { createdAt: ..., dataCount: 0 }

singleton1.addData("Item 1");
console.log(singleton2.getData()); // ["Item 1"]
```

### 第二步：模块模式实现单例

使用模块模式可以更清晰地实现单例：

```javascript
// 使用模块模式实现单例
const Logger = (function() {
  let instance;
  
  function createInstance() {
    return {
      logs: [],
      
      log: function(message) {
        const logEntry = {
          timestamp: new Date(),
          message: message,
          level: 'info'
        };
        this.logs.push(logEntry);
        console.log(`[INFO] ${logEntry.timestamp.toISOString()}: ${message}`);
      },
      
      error: function(message) {
        const logEntry = {
          timestamp: new Date(),
          message: message,
          level: 'error'
        };
        this.logs.push(logEntry);
        console.error(`[ERROR] ${logEntry.timestamp.toISOString()}: ${message}`);
      },
      
      getLogs: function() {
        return this.logs;
      },
      
      clearLogs: function() {
        this.logs = [];
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

logger1.log("First message");
logger2.log("Second message");

console.log(logger1.getLogs().length); // 2
console.log(logger2.getLogs().length); // 2
```

### 第三步：ES6模块实现单例

使用ES6模块可以创建天然的单例：

```javascript
// logger.js - 使用ES6模块实现单例
class Logger {
  constructor() {
    // 防止通过new直接创建多个实例
    if (Logger.instance) {
      return Logger.instance;
    }
    
    this.logs = [];
    this.createdAt = new Date();
    
    // 冻结实例防止修改
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
  
  warn(message) {
    this.logs.push({
      timestamp: new Date(),
      message: message,
      level: 'warn'
    });
    console.warn(`[WARN] ${message}`);
  }
  
  getLogs() {
    return this.logs;
  }
  
  getLogCount() {
    return this.logs.length;
  }
  
  getLogsByLevel(level) {
    return this.logs.filter(log => log.level === level);
  }
  
  clearLogs() {
    this.logs = [];
  }
}

// 创建并导出单例实例
const logger = new Logger();
Object.freeze(logger); // 冻结对象，防止修改

export default logger;
```

### 第四步：线程安全的单例实现

在多线程环境中，需要考虑线程安全问题：

```javascript
// 线程安全的单例实现（JavaScript中通过异步操作模拟）
class ThreadSafeSingleton {
  constructor() {
    if (ThreadSafeSingleton.instance) {
      return ThreadSafeSingleton.instance;
    }
    
    this.data = new Map();
    this.locked = false;
    
    ThreadSafeSingleton.instance = this;
    return this;
  }
  
  // 模拟线程安全的操作
  async setData(key, value) {
    // 模拟获取锁
    while (this.locked) {
      await new Promise(resolve => setTimeout(resolve, 1));
    }
    
    this.locked = true;
    
    try {
      // 模拟一些处理时间
      await new Promise(resolve => setTimeout(resolve, 10));
      this.data.set(key, value);
      console.log(`Set data: ${key} = ${value}`);
    } finally {
      this.locked = false;
    }
  }
  
  getData(key) {
    return this.data.get(key);
  }
  
  getAllData() {
    return Object.fromEntries(this.data);
  }
}

// 使用示例
async function demonstrateThreadSafety() {
  const singleton = new ThreadSafeSingleton();
  
  // 模拟并发访问
  const promises = [];
  for (let i = 0; i < 5; i++) {
    promises.push(singleton.setData(`key${i}`, `value${i}`));
  }
  
  await Promise.all(promises);
  console.log(singleton.getAllData());
}

demonstrateThreadSafety();
```

### 第五步：带有配置的单例实现

实际应用中，单例通常需要支持配置：

```javascript
// 带配置的单例实现
class ConfigurableSingleton {
  constructor() {
    if (ConfigurableSingleton.instance) {
      return ConfigurableSingleton.instance;
    }
    
    this.config = {
      maxLogs: 1000,
      logLevel: 'info',
      enableTimestamp: true
    };
    
    this.logs = [];
    
    ConfigurableSingleton.instance = this;
    return this;
  }
  
  // 配置方法
  configure(newConfig) {
    this.config = { ...this.config, ...newConfig };
    return this;
  }
  
  getConfig() {
    return { ...this.config };
  }
  
  // 日志方法
  log(message, level = 'info') {
    // 检查日志级别
    if (!this.shouldLog(level)) {
      return;
    }
    
    const logEntry = {
      message,
      level,
      timestamp: this.config.enableTimestamp ? new Date() : null
    };
    
    // 限制日志数量
    if (this.logs.length >= this.config.maxLogs) {
      this.logs.shift(); // 移除最旧的日志
    }
    
    this.logs.push(logEntry);
    
    // 输出到控制台
    const prefix = this.config.enableTimestamp ? 
      `[${logEntry.timestamp?.toISOString()}] ` : '';
    console.log(`${prefix}[${level.toUpperCase()}] ${message}`);
  }
  
  error(message) {
    this.log(message, 'error');
  }
  
  warn(message) {
    this.log(message, 'warn');
  }
  
  info(message) {
    this.log(message, 'info');
  }
  
  debug(message) {
    this.log(message, 'debug');
  }
  
  getLogs() {
    return [...this.logs];
  }
  
  getLogsByLevel(level) {
    return this.logs.filter(log => log.level === level);
  }
  
  clearLogs() {
    this.logs = [];
  }
  
  // 私有方法：检查是否应该记录指定级别的日志
  shouldLog(level) {
    const levels = ['debug', 'info', 'warn', 'error'];
    const currentLevelIndex = levels.indexOf(this.config.logLevel);
    const messageLevelIndex = levels.indexOf(level);
    
    return messageLevelIndex >= currentLevelIndex;
  }
}

// 使用示例
const configSingleton = new ConfigurableSingleton();

// 配置单例
configSingleton.configure({
  maxLogs: 10,
  logLevel: 'warn',
  enableTimestamp: false
});

// 测试不同级别的日志
configSingleton.debug("Debug message"); // 不会输出
configSingleton.info("Info message");   // 不会输出
configSingleton.warn("Warn message");   // 会输出
configSingleton.error("Error message"); // 会输出

console.log("Current config:", configSingleton.getConfig());
console.log("Logs:", configSingleton.getLogs());
```

## 优势

1. **内存中只有一个实例**：减少了内存开销，特别是一个对象需要频繁创建和销毁时
2. **避免对资源的多重占用**：如文件操作、数据库连接等，避免资源冲突
3. **设置全局访问点**：便于严格控制客户对它的访问，可以精密地控制如何以及何时访问它
4. **允许对操作和表示的精化**：可以有子类化的版本，使用这个扩展实例的单例类

## 最佳实践

1. **谨慎使用**：单例模式容易被滥用，应谨慎使用，避免创建过多的全局状态
2. **线程安全**：在多线程环境中需要考虑线程安全问题，确保实例的唯一性
3. **延迟初始化**：可以采用延迟初始化来提高性能，只在需要时创建实例
4. **避免全局状态**：单例容易创建全局状态，应尽量避免，或者明确其作用域
5. **测试友好**：考虑单例对单元测试的影响，提供适当的方法来重置或替换实例

## 常见误区

1. **过度使用**：将所有类都设计为单例，违背了面向对象设计原则
2. **隐藏依赖**：单例隐藏了类之间的依赖关系，使得代码难以理解和维护
3. **测试困难**：单例模式使得单元测试变得困难，因为全局状态难以重置
4. **违反单一职责原则**：单例类既要管理自己的实例，又要处理业务逻辑
5. **紧耦合**：使用单例的类与单例类紧密耦合，难以替换或模拟

## 实际应用示例

### 数据库连接管理器

```javascript
class DatabaseConnectionManager {
  constructor() {
    if (DatabaseConnectionManager.instance) {
      return DatabaseConnectionManager.instance;
    }
    
    this.connections = new Map();
    this.maxConnections = 10;
    
    DatabaseConnectionManager.instance = this;
    return this;
  }
  
  getConnection(database) {
    if (this.connections.has(database)) {
      return this.connections.get(database);
    }
    
    if (this.connections.size >= this.maxConnections) {
      throw new Error("Maximum connections reached");
    }
    
    // 模拟创建数据库连接
    const connection = {
      id: Date.now(),
      database: database,
      connectedAt: new Date(),
      query: function(sql) {
        console.log(`Executing query on ${database}: ${sql}`);
        return `Result for: ${sql}`;
      }
    };
    
    this.connections.set(database, connection);
    return connection;
  }
  
  releaseConnection(database) {
    this.connections.delete(database);
  }
  
  getConnectionCount() {
    return this.connections.size;
  }
  
  getAllConnections() {
    return Array.from(this.connections.values());
  }
}

// 使用示例
const dbManager = new DatabaseConnectionManager();
const conn1 = dbManager.getConnection("users");
const conn2 = dbManager.getConnection("products");

console.log(dbManager.getConnectionCount()); // 2
console.log(conn1.query("SELECT * FROM users")); // 执行查询
```

### 应用配置管理器

```javascript
class AppConfig {
  constructor() {
    if (AppConfig.instance) {
      return AppConfig.instance;
    }
    
    this.settings = {
      apiUrl: "https://api.example.com",
      timeout: 5000,
      retries: 3,
      debug: false
    };
    
    // 从存储中加载配置
    this.loadFromStorage();
    
    AppConfig.instance = this;
    return this;
  }
  
  loadFromStorage() {
    try {
      const stored = localStorage.getItem("appConfig");
      if (stored) {
        this.settings = { ...this.settings, ...JSON.parse(stored) };
      }
    } catch (error) {
      console.warn("Failed to load config from storage:", error);
    }
  }
  
  saveToStorage() {
    try {
      localStorage.setItem("appConfig", JSON.stringify(this.settings));
    } catch (error) {
      console.warn("Failed to save config to storage:", error);
    }
  }
  
  get(key) {
    return this.settings[key];
  }
  
  set(key, value) {
    this.settings[key] = value;
    this.saveToStorage();
  }
  
  getAll() {
    return { ...this.settings };
  }
  
  update(newSettings) {
    this.settings = { ...this.settings, ...newSettings };
    this.saveToStorage();
  }
}

// 使用示例
const config = new AppConfig();
console.log(config.get("apiUrl")); // https://api.example.com

config.set("timeout", 10000);
console.log(config.get("timeout")); // 10000

config.update({
  debug: true,
  newFeature: true
});
console.log(config.getAll());
```

## 总结

单例模式是一种重要的创建型设计模式，在合适的场景下使用可以带来很多好处。它确保类只有一个实例，并提供全局访问点，适用于需要控制资源访问或维护全局状态的场景。

然而，单例模式也容易被滥用，可能导致代码紧耦合、难以测试等问题。在使用时需要谨慎考虑：

1. **明确使用场景**：只在确实需要唯一实例时使用
2. **考虑替代方案**：有时依赖注入等方式可能更合适
3. **注意线程安全**：在多线程环境中确保实例的唯一性
4. **便于测试**：提供适当的方法来重置或替换实例以支持单元测试

正确使用单例模式可以提高系统性能、减少资源消耗，并提供一致的全局访问点。