# 抽象工厂模式 (Abstract Factory Pattern)

## 原始定义

抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

## 详细解释

抽象工厂模式是一种创建型设计模式，它能创建一系列相关的对象，而无需指定其具体类。抽象工厂定义了一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。

与工厂方法模式不同，抽象工厂模式关注的是产品族（一系列相关或相互依赖的产品），而不是单一产品。它提供了一个接口，用于创建一系列相关或相互依赖的对象，而无需指定它们具体的类。

### 核心概念

1. **产品族**：一系列相关或相互依赖的产品对象
2. **产品等级结构**：产品等级结构即产品的继承结构
3. **抽象工厂**：声明一个创建一系列产品对象的接口
4. **具体工厂**：实现创建一系列产品对象的方法

### 适用场景

1. **系统要独立于其产品的创建、组合和表示时**
2. **系统要由多个产品系列中的一个来配置时**
3. **需要强调一系列相关的产品对象的设计以便进行联合使用时**
4. **提供一个产品类库，而只想显示它们的接口而不是实现时**

## 实施步骤

### 第一步：定义抽象产品

首先定义产品族中各类产品的抽象接口：

```javascript
// 抽象产品A - 按钮
class Button {
  render() {
    throw new Error("render() method must be implemented");
  }
  
  onClick(handler) {
    throw new Error("onClick() method must be implemented");
  }
}

// 抽象产品B - 复选框
class Checkbox {
  render() {
    throw new Error("render() method must be implemented");
  }
  
  toggle() {
    throw new Error("toggle() method must be implemented");
  }
  
  isChecked() {
    throw new Error("isChecked() method must be implemented");
  }
}

// 抽象产品C - 文本框
class TextBox {
  render() {
    throw new Error("render() method must be implemented");
  }
  
  setValue(value) {
    throw new Error("setValue() method must be implemented");
  }
  
  getValue() {
    throw new Error("getValue() method must be implemented");
  }
}
```

### 第二步：实现具体产品

为每个产品族实现具体的产品类：

```javascript
// 具体产品族A - Windows风格

// 具体产品A1 - Windows按钮
class WindowsButton extends Button {
  constructor() {
    super();
    this.element = null;
  }
  
  render() {
    console.log("Rendering Windows style button");
    this.element = {
      type: "button",
      style: "windows-button",
      text: "Windows Button"
    };
    return this.element;
  }
  
  onClick(handler) {
    console.log("Windows button click handler attached");
    // 实际的点击事件绑定逻辑
  }
}

// 具体产品B1 - Windows复选框
class WindowsCheckbox extends Checkbox {
  constructor() {
    super();
    this.checked = false;
    this.element = null;
  }
  
  render() {
    console.log("Rendering Windows style checkbox");
    this.element = {
      type: "checkbox",
      style: "windows-checkbox",
      checked: this.checked
    };
    return this.element;
  }
  
  toggle() {
    this.checked = !this.checked;
    console.log(`Windows checkbox toggled. Now ${this.checked ? 'checked' : 'unchecked'}`);
  }
  
  isChecked() {
    return this.checked;
  }
}

// 具体产品C1 - Windows文本框
class WindowsTextBox extends TextBox {
  constructor() {
    super();
    this.value = "";
    this.element = null;
  }
  
  render() {
    console.log("Rendering Windows style text box");
    this.element = {
      type: "textbox",
      style: "windows-textbox",
      value: this.value
    };
    return this.element;
  }
  
  setValue(value) {
    this.value = value;
    console.log(`Windows text box value set to: ${value}`);
  }
  
  getValue() {
    return this.value;
  }
}

// 具体产品族B - Mac风格

// 具体产品A2 - Mac按钮
class MacButton extends Button {
  constructor() {
    super();
    this.element = null;
  }
  
  render() {
    console.log("Rendering Mac style button");
    this.element = {
      type: "button",
      style: "mac-button",
      text: "Mac Button"
    };
    return this.element;
  }
  
  onClick(handler) {
    console.log("Mac button click handler attached");
    // 实际的点击事件绑定逻辑
  }
}

// 具体产品B2 - Mac复选框
class MacCheckbox extends Checkbox {
  constructor() {
    super();
    this.checked = false;
    this.element = null;
  }
  
  render() {
    console.log("Rendering Mac style checkbox");
    this.element = {
      type: "checkbox",
      style: "mac-checkbox",
      checked: this.checked
    };
    return this.element;
  }
  
  toggle() {
    this.checked = !this.checked;
    console.log(`Mac checkbox toggled. Now ${this.checked ? 'checked' : 'unchecked'}`);
  }
  
  isChecked() {
    return this.checked;
  }
}

// 具体产品C2 - Mac文本框
class MacTextBox extends TextBox {
  constructor() {
    super();
    this.value = "";
    this.element = null;
  }
  
  render() {
    console.log("Rendering Mac style text box");
    this.element = {
      type: "textbox",
      style: "mac-textbox",
      value: this.value
    };
    return this.element;
  }
  
  setValue(value) {
    this.value = value;
    console.log(`Mac text box value set to: ${value}`);
  }
  
  getValue() {
    return this.value;
  }
}
```

### 第三步：定义抽象工厂

定义创建产品族的抽象工厂接口：

```javascript
// 抽象工厂 - 定义创建产品族的接口
class UIFactory {
  createButton() {
    throw new Error("createButton() method must be implemented");
  }
  
  createCheckbox() {
    throw new Error("createCheckbox() method must be implemented");
  }
  
  createTextBox() {
    throw new Error("createTextBox() method must be implemented");
  }
}
```

### 第四步：实现具体工厂

为每个产品族实现具体的工厂类：

```javascript
// 具体工厂1 - Windows UI工厂
class WindowsFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
  
  createCheckbox() {
    return new WindowsCheckbox();
  }
  
  createTextBox() {
    return new WindowsTextBox();
  }
}

// 具体工厂2 - Mac UI工厂
class MacFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }
  
  createCheckbox() {
    return new MacCheckbox();
  }
  
  createTextBox() {
    return new MacTextBox();
  }
}
```

### 第五步：使用抽象工厂模式

创建客户端代码来使用抽象工厂模式：

```javascript
// 客户端代码 - 应用程序类
class Application {
  constructor(factory) {
    this.factory = factory;
    this.components = [];
  }
  
  createUI() {
    console.log("=== Creating UI Components ===");
    
    // 使用工厂创建一系列相关的产品
    const button = this.factory.createButton();
    const checkbox = this.factory.createCheckbox();
    const textBox = this.factory.createTextBox();
    
    // 存储组件
    this.components = [button, checkbox, textBox];
    
    // 渲染组件
    console.log("Rendering components:");
    button.render();
    checkbox.render();
    textBox.render();
    
    return this.components;
  }
  
  getComponent(index) {
    return this.components[index];
  }
  
  getAllComponents() {
    return [...this.components];
  }
}

// 使用示例
console.log("=== Windows UI ===");
const windowsFactory = new WindowsFactory();
const windowsApp = new Application(windowsFactory);
const windowsComponents = windowsApp.createUI();

console.log("\n=== Mac UI ===");
const macFactory = new MacFactory();
const macApp = new Application(macFactory);
const macComponents = macApp.createUI();
```

### 第六步：动态工厂选择

实现根据配置动态选择工厂：

```javascript
// 工厂选择器
class UIFactorySelector {
  static getFactory(platform) {
    switch (platform.toLowerCase()) {
      case 'windows':
        return new WindowsFactory();
      case 'mac':
      case 'macos':
        return new MacFactory();
      default:
        // 根据运行环境自动选择
        return UIFactorySelector.getFactoryForEnvironment();
    }
  }
  
  static getFactoryForEnvironment() {
    // 检测运行环境
    if (typeof process !== 'undefined' && process.platform) {
      // Node.js环境
      if (process.platform === 'win32') {
        return new WindowsFactory();
      } else {
        return new MacFactory();
      }
    } else if (typeof navigator !== 'undefined') {
      // 浏览器环境
      const userAgent = navigator.userAgent;
      if (userAgent.includes('Windows')) {
        return new WindowsFactory();
      } else if (userAgent.includes('Mac')) {
        return new MacFactory();
      }
    }
    
    // 默认返回Windows工厂
    return new WindowsFactory();
  }
}

// 使用工厂选择器
class CrossPlatformApplication {
  constructor(platform) {
    this.factory = UIFactorySelector.getFactory(platform);
    this.uiComponents = [];
  }
  
  initialize() {
    console.log("=== Initializing Cross-Platform Application ===");
    this.createUI();
  }
  
  createUI() {
    console.log("Creating UI with factory:", this.factory.constructor.name);
    
    const button = this.factory.createButton();
    const checkbox = this.factory.createCheckbox();
    const textBox = this.factory.createTextBox();
    
    this.uiComponents = [button, checkbox, textBox];
    
    // 渲染所有组件
    this.uiComponents.forEach(component => component.render());
  }
  
  interactWithUI() {
    console.log("=== Interacting with UI ===");
    
    const [button, checkbox, textBox] = this.uiComponents;
    
    // 与按钮交互
    button.onClick(() => console.log("Button clicked!"));
    
    // 与复选框交互
    checkbox.toggle();
    console.log("Checkbox is checked:", checkbox.isChecked());
    checkbox.toggle();
    console.log("Checkbox is checked:", checkbox.isChecked());
    
    // 与文本框交互
    textBox.setValue("Hello, Abstract Factory!");
    console.log("Text box value:", textBox.getValue());
  }
}

// 使用示例
console.log("=== Windows Platform ===");
const windowsApp = new CrossPlatformApplication('windows');
windowsApp.initialize();
windowsApp.interactWithUI();

console.log("\n=== Mac Platform ===");
const macApp = new CrossPlatformApplication('mac');
macApp.initialize();
macApp.interactWithUI();
```

### 第七步：产品族扩展

展示如何扩展产品族：

```javascript
// 扩展产品族 - 添加新的产品类型

// 新的抽象产品 - 下拉列表
class Dropdown {
  render() {
    throw new Error("render() method must be implemented");
  }
  
  select(option) {
    throw new Error("select() method must be implemented");
  }
  
  getSelectedOption() {
    throw new Error("getSelectedOption() method must be implemented");
  }
}

// Windows风格的下拉列表
class WindowsDropdown extends Dropdown {
  constructor() {
    super();
    this.selectedOption = null;
    this.options = ['Option 1', 'Option 2', 'Option 3'];
  }
  
  render() {
    console.log("Rendering Windows style dropdown");
    return {
      type: "dropdown",
      style: "windows-dropdown",
      options: this.options,
      selected: this.selectedOption
    };
  }
  
  select(option) {
    if (this.options.includes(option)) {
      this.selectedOption = option;
      console.log(`Windows dropdown selected: ${option}`);
    }
  }
  
  getSelectedOption() {
    return this.selectedOption;
  }
}

// Mac风格的下拉列表
class MacDropdown extends Dropdown {
  constructor() {
    super();
    this.selectedOption = null;
    this.options = ['Option 1', 'Option 2', 'Option 3'];
  }
  
  render() {
    console.log("Rendering Mac style dropdown");
    return {
      type: "dropdown",
      style: "mac-dropdown",
      options: this.options,
      selected: this.selectedOption
    };
  }
  
  select(option) {
    if (this.options.includes(option)) {
      this.selectedOption = option;
      console.log(`Mac dropdown selected: ${option}`);
    }
  }
  
  getSelectedOption() {
    return this.selectedOption;
  }
}

// 扩展抽象工厂接口
class ExtendedUIFactory extends UIFactory {
  createDropdown() {
    throw new Error("createDropdown() method must be implemented");
  }
}

// 扩展具体工厂
class ExtendedWindowsFactory extends ExtendedUIFactory {
  createButton() {
    return new WindowsButton();
  }
  
  createCheckbox() {
    return new WindowsCheckbox();
  }
  
  createTextBox() {
    return new WindowsTextBox();
  }
  
  createDropdown() {
    return new WindowsDropdown();
  }
}

class ExtendedMacFactory extends ExtendedUIFactory {
  createButton() {
    return new MacButton();
  }
  
  createCheckbox() {
    return new MacCheckbox();
  }
  
  createTextBox() {
    return new MacTextBox();
  }
  
  createDropdown() {
    return new MacDropdown();
  }
}

// 使用扩展的产品族
class ExtendedApplication {
  constructor(factory) {
    this.factory = factory;
    this.components = [];
  }
  
  createExtendedUI() {
    console.log("=== Creating Extended UI ===");
    
    const button = this.factory.createButton();
    const checkbox = this.factory.createCheckbox();
    const textBox = this.factory.createTextBox();
    const dropdown = this.factory.createDropdown();
    
    this.components = [button, checkbox, textBox, dropdown];
    
    // 渲染所有组件
    this.components.forEach(component => component.render());
    
    return this.components;
  }
  
  interactWithExtendedUI() {
    console.log("=== Interacting with Extended UI ===");
    
    const [button, checkbox, textBox, dropdown] = this.components;
    
    // 与下拉列表交互
    dropdown.select('Option 2');
    console.log("Selected option:", dropdown.getSelectedOption());
  }
}

// 使用示例
console.log("=== Extended Windows UI ===");
const extendedWindowsFactory = new ExtendedWindowsFactory();
const extendedWindowsApp = new ExtendedApplication(extendedWindowsFactory);
extendedWindowsApp.createExtendedUI();
extendedWindowsApp.interactWithExtendedUI();
```

## 优势

1. **易于交换产品系列**：只需改变具体的工厂即可轻松切换整个产品系列
2. **有利于产品的一致性**：确保同一工厂生产的产品是相互匹配的，保证了产品族中对象的一致性
3. **符合开闭原则**：增加新的产品系列时无需修改原有代码，只需添加新的具体工厂和产品类
4. **隔离具体类**：客户端不需要知道具体产品的类名，只需要知道抽象接口

## 最佳实践

1. **产品族必须一起使用**：确保相关的产品对象一起使用，以保证UI的一致性
2. **系统需要配置多个产品系列**：适用于需要支持多种外观或主题的系统
3. **强调产品的接口而非实现**：客户端只依赖于抽象接口，而不是具体实现
4. **合理设计产品族**：将相关的产品组织成族，避免产品族过于庞大
5. **考虑扩展性**：设计时考虑未来可能添加的新产品类型

## 常见误区

1. **支持新种类的产品困难**：增加新的产品等级结构比较麻烦，需要修改抽象工厂接口
2. **工厂类层次过于复杂**：可能导致系统中类的数量急剧增加
3. **客户端需要知道具体工厂类**：客户端需要知道使用哪个工厂类来创建产品族
4. **产品族过于庞大**：如果产品族包含太多不同类型的产品，会增加系统的复杂性

## 实际应用示例

### 数据库访问抽象工厂

```javascript
// 数据库连接产品
class DatabaseConnection {
  connect() {
    throw new Error("connect() method must be implemented");
  }
  
  disconnect() {
    throw new Error("disconnect() method must be implemented");
  }
}

// SQL查询构建器产品
class QueryBuilder {
  select(fields) {
    throw new Error("select() method must be implemented");
  }
  
  from(table) {
    throw new Error("from() method must be implemented");
  }
  
  where(condition) {
    throw new Error("where() method must be implemented");
  }
  
  build() {
    throw new Error("build() method must be implemented");
  }
}

// 数据访问对象产品
class DataAccessObject {
  insert(data) {
    throw new Error("insert() method must be implemented");
  }
  
  update(id, data) {
    throw new Error("update() method must be implemented");
  }
  
  delete(id) {
    throw new Error("delete() method must be implemented");
  }
  
  find(id) {
    throw new Error("find() method must be implemented");
  }
}

// MySQL产品族
class MySQLConnection extends DatabaseConnection {
  connect() {
    console.log("Connecting to MySQL database");
    return "MySQL connection established";
  }
  
  disconnect() {
    console.log("Disconnecting from MySQL database");
  }
}

class MySQLQueryBuilder extends QueryBuilder {
  constructor() {
    super();
    this.query = "";
  }
  
  select(fields) {
    this.query = `SELECT ${fields.join(', ')} `;
    return this;
  }
  
  from(table) {
    this.query += `FROM ${table} `;
    return this;
  }
  
  where(condition) {
    this.query += `WHERE ${condition} `;
    return this;
  }
  
  build() {
    return this.query.trim();
  }
}

class MySQLDAO extends DataAccessObject {
  constructor(connection) {
    super();
    this.connection = connection;
  }
  
  insert(data) {
    console.log(`MySQL: Inserting data into table - ${JSON.stringify(data)}`);
    return { id: Date.now(), ...data };
  }
  
  update(id, data) {
    console.log(`MySQL: Updating record ${id} with data - ${JSON.stringify(data)}`);
    return { id, ...data };
  }
  
  delete(id) {
    console.log(`MySQL: Deleting record ${id}`);
    return true;
  }
  
  find(id) {
    console.log(`MySQL: Finding record ${id}`);
    return { id, name: "Sample Data" };
  }
}

// PostgreSQL产品族
class PostgreSQLConnection extends DatabaseConnection {
  connect() {
    console.log("Connecting to PostgreSQL database");
    return "PostgreSQL connection established";
  }
  
  disconnect() {
    console.log("Disconnecting from PostgreSQL database");
  }
}

class PostgreSQLQueryBuilder extends QueryBuilder {
  constructor() {
    super();
    this.query = "";
  }
  
  select(fields) {
    this.query = `SELECT ${fields.join(', ')} `;
    return this;
  }
  
  from(table) {
    this.query += `FROM ${table} `;
    return this;
  }
  
  where(condition) {
    this.query += `WHERE ${condition} `;
    return this;
  }
  
  build() {
    return this.query.trim();
  }
}

class PostgreSQLDAO extends DataAccessObject {
  constructor(connection) {
    super();
    this.connection = connection;
  }
  
  insert(data) {
    console.log(`PostgreSQL: Inserting data into table - ${JSON.stringify(data)}`);
    return { id: Date.now(), ...data };
  }
  
  update(id, data) {
    console.log(`PostgreSQL: Updating record ${id} with data - ${JSON.stringify(data)}`);
    return { id, ...data };
  }
  
  delete(id) {
    console.log(`PostgreSQL: Deleting record ${id}`);
    return true;
  }
  
  find(id) {
    console.log(`PostgreSQL: Finding record ${id}`);
    return { id, name: "Sample Data" };
  }
}

// 抽象工厂
class DatabaseFactory {
  createConnection() {
    throw new Error("createConnection() method must be implemented");
  }
  
  createQueryBuilder() {
    throw new Error("createQueryBuilder() method must be implemented");
  }
  
  createDAO(connection) {
    throw new Error("createDAO() method must be implemented");
  }
}

// 具体工厂
class MySQLFactory extends DatabaseFactory {
  createConnection() {
    return new MySQLConnection();
  }
  
  createQueryBuilder() {
    return new MySQLQueryBuilder();
  }
  
  createDAO(connection) {
    return new MySQLDAO(connection);
  }
}

class PostgreSQLFactory extends DatabaseFactory {
  createConnection() {
    return new PostgreSQLConnection();
  }
  
  createQueryBuilder() {
    return new PostgreSQLQueryBuilder();
  }
  
  createDAO(connection) {
    return new PostgreSQLDAO(connection);
  }
}

// 使用示例
class DatabaseService {
  constructor(factory) {
    this.factory = factory;
    this.connection = null;
    this.queryBuilder = null;
    this.dao = null;
  }
  
  initialize() {
    console.log("=== Initializing Database Service ===");
    this.connection = this.factory.createConnection();
    this.queryBuilder = this.factory.createQueryBuilder();
    this.dao = this.factory.createDAO(this.connection);
    
    return this.connection.connect();
  }
  
  buildQuery() {
    const query = this.queryBuilder
      .select(['id', 'name'])
      .from('users')
      .where('age > 18')
      .build();
    
    console.log("Built query:", query);
    return query;
  }
  
  performCRUD() {
    console.log("=== Performing CRUD Operations ===");
    
    const newData = this.dao.insert({ name: "John Doe", age: 30 });
    console.log("Inserted:", newData);
    
    const updatedData = this.dao.update(newData.id, { age: 31 });
    console.log("Updated:", updatedData);
    
    const foundData = this.dao.find(newData.id);
    console.log("Found:", foundData);
    
    const deleted = this.dao.delete(newData.id);
    console.log("Deleted:", deleted);
  }
  
  close() {
    if (this.connection) {
      this.connection.disconnect();
    }
  }
}

// 使用MySQL
console.log("=== MySQL Database Service ===");
const mysqlFactory = new MySQLFactory();
const mysqlService = new DatabaseService(mysqlFactory);
mysqlService.initialize();
mysqlService.buildQuery();
mysqlService.performCRUD();
mysqlService.close();

console.log("\n=== PostgreSQL Database Service ===");
const postgresFactory = new PostgreSQLFactory();
const postgresService = new DatabaseService(postgresFactory);
postgresService.initialize();
postgresService.buildQuery();
postgresService.performCRUD();
postgresService.close();
```

### 游戏角色装备抽象工厂

```javascript
// 游戏装备产品族

// 武器产品
class Weapon {
  getType() {
    throw new Error("getType() method must be implemented");
  }
  
  getDamage() {
    throw new Error("getDamage() method must be implemented");
  }
  
  attack() {
    throw new Error("attack() method must be implemented");
  }
}

// 防具产品
class Armor {
  getType() {
    throw new Error("getType() method must be implemented");
  }
  
  getDefense() {
    throw new Error("getDefense() method must be implemented");
  }
  
  defend() {
    throw new Error("defend() method must be implemented");
  }
}

// 饰品产品
class Accessory {
  getType() {
    throw new Error("getType() method must be implemented");
  }
  
  getBonus() {
    throw new Error("getBonus() method must be implemented");
  }
  
  activate() {
    throw new Error("activate() method must be implemented");
  }
}

// 战士装备族
class WarriorWeapon extends Weapon {
  getType() {
    return "Warrior Sword";
  }
  
  getDamage() {
    return 50;
  }
  
  attack() {
    console.log("Warrior swings sword with great force!");
    return this.getDamage();
  }
}

class WarriorArmor extends Armor {
  getType() {
    return "Plate Armor";
  }
  
  getDefense() {
    return 80;
  }
  
  defend() {
    console.log("Plate armor absorbs the blow!");
    return this.getDefense();
  }
}

class WarriorAccessory extends Accessory {
  getType() {
    return "Strength Amulet";
  }
  
  getBonus() {
    return { strength: 10 };
  }
  
  activate() {
    console.log("Strength amulet increases warrior's power!");
    return this.getBonus();
  }
}

// 法师装备族
class MageWeapon extends Weapon {
  getType() {
    return "Magic Staff";
  }
  
  getDamage() {
    return 70;
  }
  
  attack() {
    console.log("Mage casts a powerful spell!");
    return this.getDamage();
  }
}

class MageArmor extends Armor {
  getType() {
    return "Robe";
  }
  
  getDefense() {
    return 30;
  }
  
  defend() {
    console.log("Robe provides magical protection!");
    return this.getDefense();
  }
}

class MageAccessory extends Accessory {
  getType() {
    return "Intelligence Ring";
  }
  
  getBonus() {
    return { intelligence: 15 };
  }
  
  activate() {
    console.log("Intelligence ring enhances magical power!");
    return this.getBonus();
  }
}

// 抽象工厂
class CharacterEquipmentFactory {
  createWeapon() {
    throw new Error("createWeapon() method must be implemented");
  }
  
  createArmor() {
    throw new Error("createArmor() method must be implemented");
  }
  
  createAccessory() {
    throw new Error("createAccessory() method must be implemented");
  }
}

// 具体工厂
class WarriorEquipmentFactory extends CharacterEquipmentFactory {
  createWeapon() {
    return new WarriorWeapon();
  }
  
  createArmor() {
    return new WarriorArmor();
  }
  
  createAccessory() {
    return new WarriorAccessory();
  }
}

class MageEquipmentFactory extends CharacterEquipmentFactory {
  createWeapon() {
    return new MageWeapon();
  }
  
  createArmor() {
    return new MageArmor();
  }
  
  createAccessory() {
    return new MageAccessory();
  }
}

// 角色类
class Character {
  constructor(name, factory) {
    this.name = name;
    this.factory = factory;
    this.weapon = null;
    this.armor = null;
    this.accessory = null;
  }
  
  equip() {
    console.log(`=== Equipping ${this.name} ===`);
    this.weapon = this.factory.createWeapon();
    this.armor = this.factory.createArmor();
    this.accessory = this.factory.createAccessory();
    
    console.log(`Equipped: ${this.weapon.getType()}`);
    console.log(`Equipped: ${this.armor.getType()}`);
    console.log(`Equipped: ${this.accessory.getType()}`);
  }
  
  attack() {
    if (this.weapon) {
      return this.weapon.attack();
    }
  }
  
  defend(damage) {
    if (this.armor) {
      const defense = this.armor.defend();
      const remainingDamage = Math.max(0, damage - defense);
      console.log(`Damage reduced by ${defense}. Remaining damage: ${remainingDamage}`);
      return remainingDamage;
    }
    return damage;
  }
  
  activateAccessory() {
    if (this.accessory) {
      return this.accessory.activate();
    }
  }
}

// 使用示例
console.log("=== Warrior Character ===");
const warriorFactory = new WarriorEquipmentFactory();
const warrior = new Character("Conan", warriorFactory);
warrior.equip();
warrior.attack();
warrior.defend(100);
warrior.activateAccessory();

console.log("\n=== Mage Character ===");
const mageFactory = new MageEquipmentFactory();
const mage = new Character("Gandalf", mageFactory);
mage.equip();
mage.attack();
mage.defend(100);
mage.activateAccessory();
```

## 总结

抽象工厂模式是一种强大的创建型设计模式，特别适用于需要创建一系列相关或相互依赖对象的场景。它通过提供一个创建产品族的接口，将客户端与具体的产品实现解耦。

抽象工厂模式的主要优势包括：

1. **产品族一致性**：确保同一工厂创建的产品是相互匹配的
2. **易于切换产品系列**：只需更换工厂即可切换整个产品系列
3. **符合开闭原则**：添加新的产品系列无需修改现有代码

但也要注意其局限性：

1. **难以支持新种类的产品**：添加新产品等级结构需要修改抽象工厂接口
2. **可能导致类数量激增**：每个产品族都需要对应的工厂类

正确使用抽象工厂模式可以大大提高系统的可维护性和可扩展性，特别是在需要支持多种产品系列的复杂系统中。它是面向对象设计中处理产品族创建问题的重要工具。