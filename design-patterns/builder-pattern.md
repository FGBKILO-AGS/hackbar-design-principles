# 建造者模式 (Builder Pattern)

## 原始定义

建造者模式将一个复杂对象的构建与其表示分离，使得同样的构建过程可以创建不同的表示。

## 详细解释

建造者模式是一种创建型设计模式，它能将复杂对象的构建过程与其表示分离，这样同样的构建过程可以创建不同的表示。建造者模式主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的过程构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的过程却相对稳定。

建造者模式的核心思想是将一个复杂对象的构建过程分解为多个简单的步骤，通过这些步骤的组合来创建不同的对象表示。这种模式使得同样的构建过程可以创建不同的表示，提高了系统的灵活性和可扩展性。

### 核心概念

1. **产品 (Product)**：表示被构造的复杂对象。ConcreteBuilder创建该产品的内部表示并定义它的装配过程
2. **抽象建造者 (Builder)**：为创建一个Product对象的各个部件指定抽象接口
3. **具体建造者 (ConcreteBuilder)**：实现Builder的接口以构造和装配该产品的各个部件。定义并明确它所创建的表示，并提供一个检索产品的接口
4. **指挥者 (Director)**：构造一个使用Builder接口的对象。指导构建过程，不依赖于具体的产品和建造者

### 适用场景

1. **当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时**
2. **当构造过程必须允许被构造的对象有不同的表示时**
3. **当需要创建的对象具有复杂的内部结构时**
4. **当需要生成的对象的属性相互依赖，需要指定其生成顺序时**

## 实施步骤

### 第一步：定义产品类

首先定义要构建的复杂对象：

```javascript
// 产品类 - 复杂对象
class House {
  constructor() {
    this.walls = null;
    this.doors = null;
    this.windows = null;
    this.roof = null;
    this.garage = null;
    this.swimmingPool = null;
    this.garden = null;
  }
  
  setWalls(walls) {
    this.walls = walls;
    return this;
  }
  
  setDoors(doors) {
    this.doors = doors;
    return this;
  }
  
  setWindows(windows) {
    this.windows = windows;
    return this;
  }
  
  setRoof(roof) {
    this.roof = roof;
    return this;
  }
  
  setGarage(garage) {
    this.garage = garage;
    return this;
  }
  
  setSwimmingPool(swimmingPool) {
    this.swimmingPool = swimmingPool;
    return this;
  }
  
  setGarden(garden) {
    this.garden = garden;
    return this;
  }
  
  getDescription() {
    const parts = [];
    if (this.walls) parts.push(`Walls: ${this.walls}`);
    if (this.doors) parts.push(`Doors: ${this.doors}`);
    if (this.windows) parts.push(`Windows: ${this.windows}`);
    if (this.roof) parts.push(`Roof: ${this.roof}`);
    if (this.garage) parts.push(`Garage: ${this.garage}`);
    if (this.swimmingPool) parts.push(`Swimming Pool: ${this.swimmingPool}`);
    if (this.garden) parts.push(`Garden: ${this.garden}`);
    
    return `House with ${parts.join(', ')}`;
  }
  
  getFeatures() {
    const features = [];
    if (this.walls) features.push('walls');
    if (this.doors) features.push('doors');
    if (this.windows) features.push('windows');
    if (this.roof) features.push('roof');
    if (this.garage) features.push('garage');
    if (this.swimmingPool) features.push('swimming pool');
    if (this.garden) features.push('garden');
    
    return features;
  }
}
```

### 第二步：定义抽象建造者

定义建造者的抽象接口：

```javascript
// 抽象建造者 - 定义创建产品各个部分的接口
class HouseBuilder {
  constructor() {
    this.house = new House();
  }
  
  buildWalls() {
    throw new Error("buildWalls() method must be implemented");
  }
  
  buildDoors() {
    throw new Error("buildDoors() method must be implemented");
  }
  
  buildWindows() {
    throw new Error("buildWindows() method must be implemented");
  }
  
  buildRoof() {
    throw new Error("buildRoof() method must be implemented");
  }
  
  buildGarage() {
    throw new Error("buildGarage() method must be implemented");
  }
  
  buildSwimmingPool() {
    throw new Error("buildSwimmingPool() method must be implemented");
  }
  
  buildGarden() {
    throw new Error("buildGarden() method must be implemented");
  }
  
  getHouse() {
    const result = this.house;
    this.house = new House(); // 重置为新实例
    return result;
  }
}
```

### 第三步：实现具体建造者

创建不同的具体建造者实现：

```javascript
// 具体建造者A - 普通房屋建造者
class StandardHouseBuilder extends HouseBuilder {
  buildWalls() {
    console.log("Building standard brick walls");
    this.house.setWalls("standard brick walls");
    return this;
  }
  
  buildDoors() {
    console.log("Installing standard wooden doors");
    this.house.setDoors("standard wooden doors");
    return this;
  }
  
  buildWindows() {
    console.log("Installing standard windows");
    this.house.setWindows("standard windows");
    return this;
  }
  
  buildRoof() {
    console.log("Building standard shingle roof");
    this.house.setRoof("standard shingle roof");
    return this;
  }
  
  buildGarage() {
    console.log("Building single-car garage");
    this.house.setGarage("single-car garage");
    return this;
  }
  
  buildSwimmingPool() {
    // 标准房屋不包含游泳池
    console.log("Standard house does not include swimming pool");
    return this;
  }
  
  buildGarden() {
    console.log("Planting basic garden");
    this.house.setGarden("basic garden");
    return this;
  }
}

// 具体建造者B - 豪华房屋建造者
class LuxuryHouseBuilder extends HouseBuilder {
  buildWalls() {
    console.log("Building luxury stone walls");
    this.house.setWalls("luxury stone walls");
    return this;
  }
  
  buildDoors() {
    console.log("Installing luxury wooden doors with brass handles");
    this.house.setDoors("luxury wooden doors with brass handles");
    return this;
  }
  
  buildWindows() {
    console.log("Installing luxury double-pane windows");
    this.house.setWindows("luxury double-pane windows");
    return this;
  }
  
  buildRoof() {
    console.log("Building luxury slate roof");
    this.house.setRoof("luxury slate roof");
    return this;
  }
  
  buildGarage() {
    console.log("Building three-car garage with automatic doors");
    this.house.setGarage("three-car garage with automatic doors");
    return this;
  }
  
  buildSwimmingPool() {
    console.log("Building luxury swimming pool with heating system");
    this.house.setSwimmingPool("luxury swimming pool with heating system");
    return this;
  }
  
  buildGarden() {
    console.log("Designing professional landscaped garden");
    this.house.setGarden("professional landscaped garden");
    return this;
  }
}

// 具体建造者C - 经济型房屋建造者
class EconomyHouseBuilder extends HouseBuilder {
  buildWalls() {
    console.log("Building economy concrete walls");
    this.house.setWalls("economy concrete walls");
    return this;
  }
  
  buildDoors() {
    console.log("Installing basic metal doors");
    this.house.setDoors("basic metal doors");
    return this;
  }
  
  buildWindows() {
    console.log("Installing basic windows");
    this.house.setWindows("basic windows");
    return this;
  }
  
  buildRoof() {
    console.log("Building basic metal roof");
    this.house.setRoof("basic metal roof");
    return this;
  }
  
  buildGarage() {
    console.log("Building small garage");
    this.house.setGarage("small garage");
    return this;
  }
  
  buildSwimmingPool() {
    // 经济型房屋不包含游泳池
    console.log("Economy house does not include swimming pool");
    return this;
  }
  
  buildGarden() {
    // 经济型房屋不包含花园
    console.log("Economy house does not include garden");
    return this;
  }
}
```

### 第四步：定义指挥者

创建指导建造过程的指挥者：

```javascript
// 指挥者 - 构造一个使用Builder接口的对象
class HouseDirector {
  constructor(builder) {
    this.builder = builder;
  }
  
  setBuilder(builder) {
    this.builder = builder;
  }
  
  // 构建标准房屋
  constructStandardHouse() {
    console.log("=== Constructing Standard House ===");
    return this.builder
      .buildWalls()
      .buildDoors()
      .buildWindows()
      .buildRoof()
      .buildGarage()
      .buildGarden()
      .getHouse();
  }
  
  // 构建豪华房屋
  constructLuxuryHouse() {
    console.log("=== Constructing Luxury House ===");
    return this.builder
      .buildWalls()
      .buildDoors()
      .buildWindows()
      .buildRoof()
      .buildGarage()
      .buildSwimmingPool()
      .buildGarden()
      .getHouse();
  }
  
  // 构建经济型房屋
  constructEconomyHouse() {
    console.log("=== Constructing Economy House ===");
    return this.builder
      .buildWalls()
      .buildDoors()
      .buildWindows()
      .buildRoof()
      .buildGarage()
      .getHouse();
  }
  
  // 构建自定义房屋
  constructCustomHouse(hasPool = false, hasGarden = true) {
    console.log("=== Constructing Custom House ===");
    let builder = this.builder
      .buildWalls()
      .buildDoors()
      .buildWindows()
      .buildRoof()
      .buildGarage();
    
    if (hasPool) {
      builder = builder.buildSwimmingPool();
    }
    
    if (hasGarden) {
      builder = builder.buildGarden();
    }
    
    return builder.getHouse();
  }
}
```

### 第五步：使用建造者模式

创建客户端代码来使用建造者模式：

```javascript
// 使用示例
console.log("=== Building Different Types of Houses ===");

// 构建标准房屋
const standardBuilder = new StandardHouseBuilder();
const standardDirector = new HouseDirector(standardBuilder);
const standardHouse = standardDirector.constructStandardHouse();
console.log("Standard House:", standardHouse.getDescription());
console.log("Features:", standardHouse.getFeatures().join(', '));

console.log("\n--- Separator ---\n");

// 构建豪华房屋
const luxuryBuilder = new LuxuryHouseBuilder();
const luxuryDirector = new HouseDirector(luxuryBuilder);
const luxuryHouse = luxuryDirector.constructLuxuryHouse();
console.log("Luxury House:", luxuryHouse.getDescription());
console.log("Features:", luxuryHouse.getFeatures().join(', '));

console.log("\n--- Separator ---\n");

// 构建经济型房屋
const economyBuilder = new EconomyHouseBuilder();
const economyDirector = new HouseDirector(economyBuilder);
const economyHouse = economyDirector.constructEconomyHouse();
console.log("Economy House:", economyHouse.getDescription());
console.log("Features:", economyHouse.getFeatures().join(', '));

console.log("\n--- Separator ---\n");

// 构建自定义房屋
const customHouse = standardDirector.constructCustomHouse(true, true);
console.log("Custom House:", customHouse.getDescription());
console.log("Features:", customHouse.getFeatures().join(', '));
```

### 第六步：链式建造者实现

实现支持链式调用的建造者模式：

```javascript
// 链式建造者 - 支持流畅的API调用
class FluentHouseBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.house = new House();
    return this;
  }
  
  withWalls(type) {
    console.log(`Adding ${type} walls`);
    this.house.setWalls(type);
    return this;
  }
  
  withDoors(type) {
    console.log(`Adding ${type} doors`);
    this.house.setDoors(type);
    return this;
  }
  
  withWindows(type) {
    console.log(`Adding ${type} windows`);
    this.house.setWindows(type);
    return this;
  }
  
  withRoof(type) {
    console.log(`Adding ${type} roof`);
    this.house.setRoof(type);
    return this;
  }
  
  withGarage(type) {
    console.log(`Adding ${type}`);
    this.house.setGarage(type);
    return this;
  }
  
  withSwimmingPool(type) {
    console.log(`Adding ${type}`);
    this.house.setSwimmingPool(type);
    return this;
  }
  
  withGarden(type) {
    console.log(`Adding ${type}`);
    this.house.setGarden(type);
    return this;
  }
  
  build() {
    const result = this.house;
    this.reset();
    return result;
  }
}

// 使用链式建造者
console.log("=== Using Fluent House Builder ===");
const fluentBuilder = new FluentHouseBuilder();

const customHouse1 = fluentBuilder
  .withWalls("brick walls")
  .withDoors("wooden doors")
  .withWindows("double-pane windows")
  .withRoof("shingle roof")
  .withGarage("two-car garage")
  .build();

console.log("Custom House 1:", customHouse1.getDescription());

const customHouse2 = fluentBuilder
  .withWalls("stone walls")
  .withDoors("steel doors")
  .withWindows("smart windows")
  .withRoof("metal roof")
  .withGarage("three-car garage")
  .withSwimmingPool("infinity pool")
  .withGarden("zen garden")
  .build();

console.log("Custom House 2:", customHouse2.getDescription());
```

### 第七步：建造者管理器

实现建造者管理器来管理多种建造者：

```javascript
// 建造者管理器 - 管理多种建造者
class HouseBuilderFactory {
  constructor() {
    this.builders = new Map();
    this.registerDefaultBuilders();
  }
  
  registerDefaultBuilders() {
    this.builders.set('standard', StandardHouseBuilder);
    this.builders.set('luxury', LuxuryHouseBuilder);
    this.builders.set('economy', EconomyHouseBuilder);
  }
  
  registerBuilder(name, builderClass) {
    this.builders.set(name, builderClass);
  }
  
  createBuilder(type) {
    const BuilderClass = this.builders.get(type);
    if (!BuilderClass) {
      throw new Error(`Unknown builder type: ${type}`);
    }
    return new BuilderClass();
  }
  
  getAvailableBuilders() {
    return Array.from(this.builders.keys());
  }
}

// 高级房屋建造服务
class AdvancedHouseBuildingService {
  constructor() {
    this.builderFactory = new HouseBuilderFactory();
    this.director = new HouseDirector(null);
  }
  
  buildHouse(builderType, houseType) {
    console.log(`=== Building ${houseType} house using ${builderType} builder ===`);
    
    try {
      const builder = this.builderFactory.createBuilder(builderType);
      this.director.setBuilder(builder);
      
      let house;
      switch (houseType.toLowerCase()) {
        case 'standard':
          house = this.director.constructStandardHouse();
          break;
        case 'luxury':
          house = this.director.constructLuxuryHouse();
          break;
        case 'economy':
          house = this.director.constructEconomyHouse();
          break;
        default:
          throw new Error(`Unknown house type: ${houseType}`);
      }
      
      return house;
    } catch (error) {
      console.error(`Error building house: ${error.message}`);
      throw error;
    }
  }
  
  getAvailableBuilderTypes() {
    return this.builderFactory.getAvailableBuilders();
  }
}

// 使用高级建造服务
console.log("=== Using Advanced House Building Service ===");
const buildingService = new AdvancedHouseBuildingService();

console.log("Available builders:", buildingService.getAvailableBuilderTypes());

try {
  const luxuryHouse = buildingService.buildHouse('luxury', 'luxury');
  console.log("Built luxury house:", luxuryHouse.getDescription());
  
  const economyHouse = buildingService.buildHouse('economy', 'economy');
  console.log("Built economy house:", economyHouse.getDescription());
} catch (error) {
  console.error("Failed to build house:", error.message);
}
```

## 优势

1. **建造代码与表示代码分离**：将复杂对象的构建过程与其表示分离，使得同样的构建过程可以创建不同的表示
2. **更好的控制产品的创建过程**：可以精细地控制产品的创建过程，分步骤构建复杂对象
3. **产品的不同表现形式**：可以使用相同的构建代码创建不同的产品表现形式
4. **符合开闭原则**：添加新的建造者或产品类型时无需修改现有代码
5. **单一职责原则**：将构建算法和具体产品类的实现分离，每个类只负责一项职责

## 最佳实践

1. **明确构建步骤**：将复杂对象的构建过程分解为清晰的步骤
2. **保持接口一致性**：所有具体建造者应该实现相同的接口
3. **合理使用指挥者**：指挥者应该只负责构建过程，不依赖具体产品
4. **支持链式调用**：建造者方法返回this以支持流畅的API调用
5. **提供默认实现**：为可选步骤提供默认实现或空实现
6. **考虑产品变体**：设计时考虑产品可能的不同变体

## 常见误区

1. **过度设计**：对于简单对象也使用建造者模式，增加了不必要的复杂性
2. **建造者过多**：为每个小变化都创建新的建造者，导致类爆炸
3. **违反开闭原则**：频繁修改抽象建造者接口来适应新产品
4. **指挥者过于复杂**：指挥者包含了过多的业务逻辑
5. **产品与建造过程紧耦合**：产品类与建造过程过于紧密，难以扩展

## 实际应用示例

### HTTP请求建造者

```javascript
// HTTP请求产品
class HttpRequest {
  constructor() {
    this.method = 'GET';
    this.url = '';
    this.headers = {};
    this.body = null;
    this.timeout = 5000;
    this.retries = 0;
    this.auth = null;
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
  
  setAuth(type, credentials) {
    this.auth = { type, credentials };
    return this;
  }
  
  build() {
    return {
      method: this.method,
      url: this.url,
      headers: { ...this.headers },
      body: this.body,
      timeout: this.timeout,
      retries: this.retries,
      auth: this.auth ? { ...this.auth } : null
    };
  }
  
  toString() {
    return `${this.method} ${this.url}`;
  }
}

// HTTP请求建造者
class HttpRequestBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.request = new HttpRequest();
    return this;
  }
  
  withMethod(method) {
    this.request.setMethod(method);
    return this;
  }
  
  withUrl(url) {
    this.request.setUrl(url);
    return this;
  }
  
  withHeader(key, value) {
    this.request.addHeader(key, value);
    return this;
  }
  
  withJsonBody(data) {
    this.request
      .addHeader('Content-Type', 'application/json')
      .setBody(JSON.stringify(data));
    return this;
  }
  
  withFormBody(data) {
    const formData = new URLSearchParams();
    for (const [key, value] of Object.entries(data)) {
      formData.append(key, value);
    }
    this.request
      .addHeader('Content-Type', 'application/x-www-form-urlencoded')
      .setBody(formData.toString());
    return this;
  }
  
  withTimeout(milliseconds) {
    this.request.setTimeout(milliseconds);
    return this;
  }
  
  withRetries(count) {
    this.request.setRetries(count);
    return this;
  }
  
  withBasicAuth(username, password) {
    const credentials = btoa(`${username}:${password}`);
    this.request.setAuth('basic', credentials);
    return this;
  }
  
  withBearerToken(token) {
    this.request.setAuth('bearer', token);
    return this;
  }
  
  build() {
    const result = this.request.build();
    this.reset();
    return result;
  }
}

// HTTP请求指挥者
class HttpRequestDirector {
  static buildGetRequest(builder, url, headers = {}) {
    return builder
      .withMethod('GET')
      .withUrl(url)
      .withHeader('Accept', 'application/json');
  }
  
  static buildPostJsonRequest(builder, url, data, headers = {}) {
    return builder
      .withMethod('POST')
      .withUrl(url)
      .withJsonBody(data)
      .withHeader('Accept', 'application/json');
  }
  
  static buildApiRequest(builder, url, token, data = null) {
    let request = builder
      .withUrl(url)
      .withHeader('Accept', 'application/json')
      .withBearerToken(token);
    
    if (data) {
      request = request.withJsonBody(data);
    }
    
    return request;
  }
}

// 使用示例
console.log("=== HTTP Request Builder Examples ===");

const builder = new HttpRequestBuilder();

// 构建GET请求
const getRequest = HttpRequestDirector
  .buildGetRequest(builder, 'https://api.example.com/users')
  .withTimeout(3000)
  .build();

console.log("GET Request:", getRequest);

// 构建POST请求
const postRequest = HttpRequestDirector
  .buildPostJsonRequest(builder, 'https://api.example.com/users', {
    name: 'John Doe',
    email: 'john@example.com'
  })
  .withRetries(3)
  .build();

console.log("POST Request:", postRequest);

// 构建API请求
const apiRequest = HttpRequestDirector
  .buildApiRequest(builder, 'https://api.example.com/profile', 'jwt-token-123', {
    bio: 'Software Developer'
  })
  .withTimeout(5000)
  .build();

console.log("API Request:", apiRequest);

// 使用链式调用直接构建
const customRequest = builder
  .withMethod('PUT')
  .withUrl('https://api.example.com/users/123')
  .withJsonBody({ name: 'Jane Doe' })
  .withHeader('X-Custom-Header', 'custom-value')
  .withTimeout(10000)
  .withRetries(2)
  .build();

console.log("Custom Request:", customRequest);
```

### 汽车配置建造者

```javascript
// 汽车产品
class Car {
  constructor() {
    this.make = '';
    this.model = '';
    this.year = new Date().getFullYear();
    this.color = 'white';
    this.engine = 'standard';
    this.transmission = 'manual';
    this.features = [];
    this.price = 0;
  }
  
  setMake(make) {
    this.make = make;
    return this;
  }
  
  setModel(model) {
    this.model = model;
    return this;
  }
  
  setYear(year) {
    this.year = year;
    return this;
  }
  
  setColor(color) {
    this.color = color;
    return this;
  }
  
  setEngine(engine) {
    this.engine = engine;
    return this;
  }
  
  setTransmission(transmission) {
    this.transmission = transmission;
    return this;
  }
  
  addFeature(feature) {
    if (!this.features.includes(feature)) {
      this.features.push(feature);
    }
    return this;
  }
  
  setPrice(price) {
    this.price = price;
    return this;
  }
  
  getDescription() {
    return `${this.year} ${this.make} ${this.model} (${this.color})`;
  }
  
  getSpecs() {
    return {
      make: this.make,
      model: this.model,
      year: this.year,
      color: this.color,
      engine: this.engine,
      transmission: this.transmission,
      features: [...this.features],
      price: this.price
    };
  }
  
  toString() {
    return `${this.getDescription()} - $${this.price.toLocaleString()}`;
  }
}

// 汽车建造者
class CarBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.car = new Car();
    return this;
  }
  
  withMake(make) {
    this.car.setMake(make);
    return this;
  }
  
  withModel(model) {
    this.car.setModel(model);
    return this;
  }
  
  withYear(year) {
    this.car.setYear(year);
    return this;
  }
  
  withColor(color) {
    this.car.setColor(color);
    return this;
  }
  
  withEngine(engine) {
    this.car.setEngine(engine);
    return this;
  }
  
  withTransmission(transmission) {
    this.car.setTransmission(transmission);
    return this;
  }
  
  withFeature(feature) {
    this.car.addFeature(feature);
    return this;
  }
  
  withPrice(price) {
    this.car.setPrice(price);
    return this;
  }
  
  build() {
    const result = this.car;
    this.reset();
    return result;
  }
}

// 汽车建造指挥者
class CarDirector {
  constructor(builder) {
    this.builder = builder;
  }
  
  setBuilder(builder) {
    this.builder = builder;
  }
  
  buildSportsCar() {
    return this.builder
      .withMake('Ferrari')
      .withModel('F8 Tributo')
      .withYear(2023)
      .withColor('red')
      .withEngine('V8 Twin-Turbo')
      .withTransmission('7-speed DCT')
      .withFeature('Carbon Fiber')
      .withFeature('Sports Exhaust')
      .withFeature('Racing Seats')
      .withPrice(270000)
      .build();
  }
  
  buildFamilyCar() {
    return this.builder
      .withMake('Toyota')
      .withModel('Camry')
      .withYear(2023)
      .withColor('silver')
      .withEngine('2.5L 4-Cylinder')
      .withTransmission('8-speed Automatic')
      .withFeature('Backup Camera')
      .withFeature('Bluetooth')
      .withFeature('Keyless Entry')
      .withPrice(28000)
      .build();
  }
  
  buildLuxuryCar() {
    return this.builder
      .withMake('Mercedes-Benz')
      .withModel('S-Class')
      .withYear(2023)
      .withColor('black')
      .withEngine('3.0L Turbo Inline-6')
      .withTransmission('9-speed Automatic')
      .withFeature('Massaging Seats')
      .withFeature('Air Suspension')
      .withFeature('HUD')
      .withFeature('Premium Sound')
      .withPrice(110000)
      .build();
  }
}

// 使用示例
console.log("=== Car Builder Examples ===");

const carBuilder = new CarBuilder();
const carDirector = new CarDirector(carBuilder);

// 构建跑车
const sportsCar = carDirector.buildSportsCar();
console.log("Sports Car:", sportsCar.toString());
console.log("Specs:", sportsCar.getSpecs());

console.log("\n--- Separator ---\n");

// 构建家用车
const familyCar = carDirector.buildFamilyCar();
console.log("Family Car:", familyCar.toString());
console.log("Specs:", familyCar.getSpecs());

console.log("\n--- Separator ---\n");

// 构建豪华车
const luxuryCar = carDirector.buildLuxuryCar();
console.log("Luxury Car:", luxuryCar.toString());
console.log("Specs:", luxuryCar.getSpecs());

console.log("\n--- Separator ---\n");

// 自定义汽车配置
const customCar = carBuilder
  .withMake('Tesla')
  .withModel('Model S')
  .withYear(2023)
  .withColor('blue')
  .withEngine('Electric')
  .withTransmission('Automatic')
  .withFeature('Autopilot')
  .withFeature('Premium Interior')
  .withFeature('Glass Roof')
  .withPrice(95000)
  .build();

console.log("Custom Car:", customCar.toString());
console.log("Specs:", customCar.getSpecs());
```

## 总结

建造者模式是一种非常实用的创建型设计模式，特别适用于构建具有复杂内部结构的对象。它通过将构建过程与表示分离，使得同样的构建过程可以创建不同的表示，提高了系统的灵活性和可扩展性。

建造者模式的主要优势包括：

1. **分离构建与表示**：将复杂对象的构建过程与其表示分离
2. **精细控制构建过程**：可以分步骤构建复杂对象，更好地控制构建过程
3. **支持不同表示**：同样的构建过程可以创建不同的产品表示
4. **符合开闭原则**：添加新的建造者或产品类型时无需修改现有代码

但在使用时也要注意避免过度设计，对于简单的对象创建，直接使用构造函数或工厂方法可能更加合适。

正确使用建造者模式可以大大提高代码的可读性、可维护性和可扩展性，是处理复杂对象创建问题的重要工具。