# 原型模式 (Prototype Pattern)

## 原始定义

原型模式用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

## 详细解释

原型模式是一种创建型设计模式，它允许一个对象通过复制现有实例来创建新的实例，而不是通过实例化类。原型模式通过复制现有对象来创建新对象，避免了重复的初始化过程。

原型模式的核心思想是通过复制现有对象来创建新对象，而不是通过new操作符创建。这种方式特别适用于创建成本较高的对象，或者需要创建大量相似对象的场景。

### 核心概念

1. **原型 (Prototype)**：声明一个克隆自身的接口
2. **具体原型 (ConcretePrototype)**：实现一个克隆自身的操作
3. **客户 (Client)**：让一个原型克隆自身从而创建一个新的对象

### 适用场景

1. **当一个系统应该独立于它的产品创建、构成和表示时**
2. **当要实例化的类是在运行时刻指定时，例如，通过动态装载**
3. **为了避免创建一个与产品类层次平行的工厂类层次时**
4. **当一个类的实例只能有几个不同状态组合中的一种时**
5. **当创建对象的成本比较昂贵时**

## 实施步骤

### 第一步：定义原型接口

首先定义原型的接口，声明克隆方法：

```javascript
// 原型接口 - 声明克隆自身的接口
class Prototype {
  clone() {
    throw new Error("clone() method must be implemented");
  }
  
  getType() {
    throw new Error("getType() method must be implemented");
  }
}
```

### 第二步：实现具体原型

创建具体的原型类，实现克隆方法：

```javascript
// 具体原型 - 实现克隆操作
class Shape extends Prototype {
  constructor(x, y, color) {
    super();
    this.x = x || 0;
    this.y = y || 0;
    this.color = color || 'black';
    this.id = Date.now() + Math.random();
  }
  
  clone() {
    // 浅拷贝
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    // 深拷贝实现
    const clone = new this.constructor();
    clone.x = this.x;
    clone.y = this.y;
    clone.color = this.color;
    clone.id = Date.now() + Math.random(); // 新的ID
    return clone;
  }
  
  getType() {
    return 'Shape';
  }
  
  draw() {
    throw new Error("draw() method must be implemented");
  }
  
  move(dx, dy) {
    this.x += dx;
    this.y += dy;
  }
  
  getInfo() {
    return {
      type: this.getType(),
      id: this.id,
      position: { x: this.x, y: this.y },
      color: this.color
    };
  }
}

// 具体原型A - 圆形
class Circle extends Shape {
  constructor(x, y, color, radius) {
    super(x, y, color);
    this.radius = radius || 10;
  }
  
  clone() {
    const clone = super.clone();
    clone.radius = this.radius;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.radius = this.radius;
    return clone;
  }
  
  getType() {
    return 'Circle';
  }
  
  draw() {
    console.log(`Drawing ${this.color} circle at (${this.x}, ${this.y}) with radius ${this.radius}`);
  }
  
  getArea() {
    return Math.PI * this.radius * this.radius;
  }
  
  getCircumference() {
    return 2 * Math.PI * this.radius;
  }
}

// 具体原型B - 矩形
class Rectangle extends Shape {
  constructor(x, y, color, width, height) {
    super(x, y, color);
    this.width = width || 10;
    this.height = height || 10;
  }
  
  clone() {
    const clone = super.clone();
    clone.width = this.width;
    clone.height = this.height;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.width = this.width;
    clone.height = this.height;
    return clone;
  }
  
  getType() {
    return 'Rectangle';
  }
  
  draw() {
    console.log(`Drawing ${this.color} rectangle at (${this.x}, ${this.y}) with size ${this.width}x${this.height}`);
  }
  
  getArea() {
    return this.width * this.height;
  }
  
  getPerimeter() {
    return 2 * (this.width + this.height);
  }
}

// 具体原型C - 三角形
class Triangle extends Shape {
  constructor(x, y, color, sideA, sideB, sideC) {
    super(x, y, color);
    this.sideA = sideA || 10;
    this.sideB = sideB || 10;
    this.sideC = sideC || 10;
  }
  
  clone() {
    const clone = super.clone();
    clone.sideA = this.sideA;
    clone.sideB = this.sideB;
    clone.sideC = this.sideC;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.sideA = this.sideA;
    clone.sideB = this.sideB;
    clone.sideC = this.sideC;
    return clone;
  }
  
  getType() {
    return 'Triangle';
  }
  
  draw() {
    console.log(`Drawing ${this.color} triangle at (${this.x}, ${this.y}) with sides ${this.sideA}, ${this.sideB}, ${this.sideC}`);
  }
  
  getArea() {
    // 使用海伦公式计算面积
    const s = (this.sideA + this.sideB + this.sideC) / 2;
    return Math.sqrt(s * (s - this.sideA) * (s - this.sideB) * (s - this.sideC));
  }
  
  getPerimeter() {
    return this.sideA + this.sideB + this.sideC;
  }
}
```

### 第三步：实现原型管理器

创建原型管理器来管理原型对象：

```javascript
// 原型管理器 - 管理原型对象的注册和获取
class PrototypeManager {
  constructor() {
    this.prototypes = new Map();
    this.initializePrototypes();
  }
  
  initializePrototypes() {
    // 注册默认原型
    this.registerPrototype('circle', new Circle(0, 0, 'red', 10));
    this.registerPrototype('rectangle', new Rectangle(0, 0, 'blue', 20, 15));
    this.registerPrototype('triangle', new Triangle(0, 0, 'green', 10, 10, 10));
  }
  
  registerPrototype(name, prototype) {
    if (!(prototype instanceof Prototype)) {
      throw new Error('Prototype must implement Prototype interface');
    }
    this.prototypes.set(name, prototype);
    console.log(`Registered prototype: ${name}`);
  }
  
  unregisterPrototype(name) {
    const removed = this.prototypes.delete(name);
    if (removed) {
      console.log(`Unregistered prototype: ${name}`);
    }
    return removed;
  }
  
  getPrototype(name) {
    const prototype = this.prototypes.get(name);
    if (!prototype) {
      throw new Error(`Prototype '${name}' not found`);
    }
    return prototype;
  }
  
  createClone(name) {
    const prototype = this.getPrototype(name);
    const clone = prototype.clone();
    console.log(`Created clone of ${name} prototype`);
    return clone;
  }
  
  createDeepClone(name) {
    const prototype = this.getPrototype(name);
    const clone = prototype.cloneDeep();
    console.log(`Created deep clone of ${name} prototype`);
    return clone;
  }
  
  listPrototypes() {
    return Array.from(this.prototypes.keys());
  }
  
  getPrototypeInfo(name) {
    const prototype = this.getPrototype(name);
    return prototype.getInfo();
  }
}
```

### 第四步：使用原型模式

创建客户端代码来使用原型模式：

```javascript
// 使用示例
console.log("=== Prototype Pattern Examples ===");

// 创建原型管理器
const prototypeManager = new PrototypeManager();

// 查看可用的原型
console.log("Available prototypes:", prototypeManager.listPrototypes());

// 创建圆形克隆
console.log("\n--- Creating Circle Clones ---");
const circle1 = prototypeManager.createClone('circle');
circle1.move(10, 20);
circle1.draw();
console.log("Circle 1 info:", circle1.getInfo());

const circle2 = prototypeManager.createClone('circle');
circle2.move(30, 40);
circle2.draw();
console.log("Circle 2 info:", circle2.getInfo());

// 创建矩形克隆
console.log("\n--- Creating Rectangle Clones ---");
const rectangle1 = prototypeManager.createClone('rectangle');
rectangle1.move(50, 60);
rectangle1.draw();
console.log("Rectangle 1 info:", rectangle1.getInfo());
console.log("Rectangle 1 area:", rectangle1.getArea());

const rectangle2 = prototypeManager.createClone('rectangle');
rectangle2.move(70, 80);
rectangle2.draw();
console.log("Rectangle 2 info:", rectangle2.getInfo());
console.log("Rectangle 2 perimeter:", rectangle2.getPerimeter());

// 创建三角形克隆
console.log("\n--- Creating Triangle Clones ---");
const triangle1 = prototypeManager.createClone('triangle');
triangle1.move(90, 100);
triangle1.draw();
console.log("Triangle 1 info:", triangle1.getInfo());
console.log("Triangle 1 area:", triangle1.getArea());

const triangle2 = prototypeManager.createClone('triangle');
triangle2.move(110, 120);
triangle2.draw();
console.log("Triangle 2 info:", triangle2.getInfo());
console.log("Triangle 2 perimeter:", triangle2.getPerimeter());
```

### 第五步：深拷贝与浅拷贝对比

展示深拷贝和浅拷贝的区别：

```javascript
// 深拷贝与浅拷贝对比
console.log("\n=== Deep Copy vs Shallow Copy ===");

// 创建包含复杂属性的原型
class ComplexShape extends Shape {
  constructor(x, y, color, metadata) {
    super(x, y, color);
    this.metadata = metadata || { 
      createdBy: 'user', 
      tags: ['shape', 'prototype'],
      properties: {
        visible: true,
        opacity: 1.0
      }
    };
  }
  
  clone() {
    // 浅拷贝
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    // 深拷贝
    const clone = new this.constructor();
    clone.x = this.x;
    clone.y = this.y;
    clone.color = this.color;
    clone.id = Date.now() + Math.random();
    
    // 深拷贝复杂属性
    clone.metadata = JSON.parse(JSON.stringify(this.metadata));
    
    return clone;
  }
  
  getType() {
    return 'ComplexShape';
  }
  
  draw() {
    console.log(`Drawing complex ${this.color} shape at (${this.x}, ${this.y})`);
  }
}

// 注册复杂原型
prototypeManager.registerPrototype('complex', new ComplexShape(0, 0, 'purple', {
  createdBy: 'admin',
  tags: ['complex', 'advanced'],
  properties: {
    visible: true,
    opacity: 0.8,
    animations: ['fade', 'rotate']
  }
}));

// 浅拷贝示例
console.log("\n--- Shallow Copy Example ---");
const shallowClone = prototypeManager.createClone('complex');
console.log("Original metadata tags:", shallowClone.metadata.tags);
console.log("Original metadata properties:", shallowClone.metadata.properties);

// 修改克隆对象的引用属性
shallowClone.metadata.tags.push('modified');
shallowClone.metadata.properties.newProperty = 'test';

console.log("After modification:");
console.log("Clone metadata tags:", shallowClone.metadata.tags);
console.log("Clone metadata properties:", shallowClone.metadata.properties);

// 检查原型是否也被修改（浅拷贝的问题）
const originalPrototype = prototypeManager.getPrototype('complex');
console.log("Original prototype metadata tags:", originalPrototype.metadata.tags);
console.log("Original prototype metadata properties:", originalPrototype.metadata.properties);

// 深拷贝示例
console.log("\n--- Deep Copy Example ---");
const deepClone = prototypeManager.createDeepClone('complex');
console.log("Original metadata tags:", deepClone.metadata.tags);
console.log("Original metadata properties:", deepClone.metadata.properties);

// 修改深克隆对象的引用属性
deepClone.metadata.tags.push('deep-modified');
deepClone.metadata.properties.deepProperty = 'deep-test';

console.log("After modification:");
console.log("Clone metadata tags:", deepClone.metadata.tags);
console.log("Clone metadata properties:", deepClone.metadata.properties);

// 检查原型是否保持不变（深拷贝的优势）
const originalPrototype2 = prototypeManager.getPrototype('complex');
console.log("Original prototype metadata tags:", originalPrototype2.metadata.tags);
console.log("Original prototype metadata properties:", originalPrototype2.metadata.properties);
```

### 第六步：原型注册与动态创建

实现动态原型注册和创建：

```javascript
// 动态原型注册和创建
console.log("\n=== Dynamic Prototype Registration ===");

// 自定义原型类
class Star extends Shape {
  constructor(x, y, color, points) {
    super(x, y, color);
    this.points = points || 5;
  }
  
  clone() {
    const clone = super.clone();
    clone.points = this.points;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.points = this.points;
    return clone;
  }
  
  getType() {
    return 'Star';
  }
  
  draw() {
    console.log(`Drawing ${this.color} star at (${this.x}, ${this.y}) with ${this.points} points`);
  }
}

class Heart extends Shape {
  constructor(x, y, color, size) {
    super(x, y, color);
    this.size = size || 10;
  }
  
  clone() {
    const clone = super.clone();
    clone.size = this.size;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.size = this.size;
    return clone;
  }
  
  getType() {
    return 'Heart';
  }
  
  draw() {
    console.log(`Drawing ${this.color} heart at (${this.x}, ${this.y}) with size ${this.size}`);
  }
}

// 动态注册新原型
prototypeManager.registerPrototype('star', new Star(0, 0, 'yellow', 5));
prototypeManager.registerPrototype('heart', new Heart(0, 0, 'pink', 15));

// 批量创建对象
console.log("\n--- Batch Creation ---");
function createBatch(prototypeName, count, xOffset = 0, yOffset = 0) {
  const clones = [];
  for (let i = 0; i < count; i++) {
    const clone = prototypeManager.createClone(prototypeName);
    clone.move(xOffset + i * 20, yOffset + i * 20);
    clones.push(clone);
  }
  return clones;
}

// 创建多个圆形
const circles = createBatch('circle', 3, 0, 0);
circles.forEach(circle => {
  circle.draw();
  console.log(`Circle info: ${circle.getInfo().type} at (${circle.x}, ${circle.y})`);
});

// 创建多个星星
const stars = createBatch('star', 2, 100, 100);
stars.forEach(star => {
  star.draw();
  console.log(`Star info: ${star.getInfo().type} at (${star.x}, ${star.y})`);
});
```

### 第七步：原型模式与工厂模式结合

将原型模式与工厂模式结合使用：

```javascript
// 原型工厂 - 结合原型模式和工厂模式
class ShapeFactory {
  constructor() {
    this.prototypeManager = new PrototypeManager();
  }
  
  createShape(type, options = {}) {
    try {
      const {
        x = 0,
        y = 0,
        color = 'black',
        cloneDeep = false,
        ...additionalOptions
      } = options;
      
      let shape;
      if (cloneDeep) {
        shape = this.prototypeManager.createDeepClone(type);
      } else {
        shape = this.prototypeManager.createClone(type);
      }
      
      // 应用位置和颜色
      shape.x = x;
      shape.y = y;
      shape.color = color;
      
      // 应用额外选项（如果支持）
      if (additionalOptions.radius && shape instanceof Circle) {
        shape.radius = additionalOptions.radius;
      }
      
      if (additionalOptions.width && additionalOptions.height && shape instanceof Rectangle) {
        shape.width = additionalOptions.width;
        shape.height = additionalOptions.height;
      }
      
      if (additionalOptions.points && shape instanceof Star) {
        shape.points = additionalOptions.points;
      }
      
      return shape;
    } catch (error) {
      console.error(`Error creating shape: ${error.message}`);
      return null;
    }
  }
  
  registerPrototype(name, prototype) {
    this.prototypeManager.registerPrototype(name, prototype);
  }
  
  getAvailableShapes() {
    return this.prototypeManager.listPrototypes();
  }
}

// 使用原型工厂
console.log("\n=== Using Shape Factory ===");
const shapeFactory = new ShapeFactory();

// 创建各种形状
const shapes = [
  shapeFactory.createShape('circle', { x: 10, y: 20, color: 'red', radius: 15 }),
  shapeFactory.createShape('rectangle', { x: 30, y: 40, color: 'blue', width: 25, height: 30 }),
  shapeFactory.createShape('triangle', { x: 50, y: 60, color: 'green' }),
  shapeFactory.createShape('star', { x: 70, y: 80, color: 'yellow', points: 6 }),
  shapeFactory.createShape('heart', { x: 90, y: 100, color: 'pink', size: 20 })
];

// 绘制所有形状
shapes.forEach(shape => {
  if (shape) {
    shape.draw();
    console.log(`Created: ${shape.getInfo().type} at (${shape.x}, ${shape.y})`);
  }
});

// 注册自定义原型并创建
class CustomShape extends Shape {
  constructor(x, y, color, pattern) {
    super(x, y, color);
    this.pattern = pattern || 'solid';
  }
  
  clone() {
    const clone = super.clone();
    clone.pattern = this.pattern;
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.pattern = this.pattern;
    return clone;
  }
  
  getType() {
    return 'CustomShape';
  }
  
  draw() {
    console.log(`Drawing ${this.color} custom shape with ${this.pattern} pattern at (${this.x}, ${this.y})`);
  }
}

shapeFactory.registerPrototype('custom', new CustomShape(0, 0, 'orange', 'dashed'));
const customShape = shapeFactory.createShape('custom', { x: 110, y: 120, color: 'purple', pattern: 'dotted' });
if (customShape) {
  customShape.draw();
  console.log(`Created: ${customShape.getInfo().type} at (${customShape.x}, ${customShape.y})`);
}
```

## 优势

1. **运行时增加和删除产品**：可以在运行时通过注册原型来增加或删除产品
2. **改变值以指定新对象**：通过修改实例的值来指定新对象
3. **减少子类的构造**：用类动态配置应用
4. **用类动态配置应用**：运行时通过装配部件来定义新类
5. **性能优势**：对于创建成本较高的对象，克隆比重新初始化更高效

## 最佳实践

1. **实现深拷贝和浅拷贝**：根据需求选择合适的克隆方式
2. **使用原型管理器**：集中管理原型对象，便于维护
3. **初始化原型对象**：为原型对象设置合适的默认值
4. **处理循环引用**：在深拷贝时注意处理对象间的循环引用
5. **文档化原型**：清楚地记录每个原型的用途和配置选项
6. **考虑内存管理**：避免创建过多的原型对象导致内存浪费

## 常见误区

1. **克隆复杂对象困难**：循环引用和深层嵌套对象的克隆可能很复杂
2. **深拷贝性能问题**：深拷贝大型对象可能影响性能
3. **原型污染**：不正确的原型使用可能导致安全问题
4. **忽略引用共享**：浅拷贝可能导致意外的引用共享问题
5. **过度使用**：对于简单对象也使用原型模式，增加了不必要的复杂性

## 实际应用示例

### 游戏角色原型系统

```javascript
// 游戏角色原型系统
class GameCharacter {
  constructor(name, level, health, attack, defense) {
    this.name = name;
    this.level = level || 1;
    this.health = health || 100;
    this.maxHealth = this.health;
    this.attack = attack || 10;
    this.defense = defense || 5;
    this.experience = 0;
    this.inventory = [];
    this.skills = [];
    this.id = Date.now() + Math.random();
  }
  
  clone() {
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    const clone = new this.constructor();
    clone.name = this.name;
    clone.level = this.level;
    clone.health = this.health;
    clone.maxHealth = this.maxHealth;
    clone.attack = this.attack;
    clone.defense = this.defense;
    clone.experience = this.experience;
    clone.inventory = [...this.inventory];
    clone.skills = [...this.skills];
    clone.id = Date.now() + Math.random();
    return clone;
  }
  
  getType() {
    return 'GameCharacter';
  }
  
  getInfo() {
    return {
      name: this.name,
      level: this.level,
      health: this.health,
      maxHealth: this.maxHealth,
      attack: this.attack,
      defense: this.defense,
      experience: this.experience,
      inventory: [...this.inventory],
      skills: [...this.skills]
    };
  }
  
  levelUp() {
    this.level++;
    this.maxHealth += 10;
    this.health = this.maxHealth;
    this.attack += 2;
    this.defense += 1;
    console.log(`${this.name} leveled up to level ${this.level}!`);
  }
  
  gainExperience(exp) {
    this.experience += exp;
    console.log(`${this.name} gained ${exp} experience points.`);
    
    // 检查是否升级
    const expNeeded = this.level * 100;
    if (this.experience >= expNeeded) {
      this.experience -= expNeeded;
      this.levelUp();
    }
  }
  
  addItem(item) {
    this.inventory.push(item);
    console.log(`${this.name} picked up ${item}.`);
  }
  
  learnSkill(skill) {
    if (!this.skills.includes(skill)) {
      this.skills.push(skill);
      console.log(`${this.name} learned ${skill}.`);
    }
  }
}

// 具体角色原型
class Warrior extends GameCharacter {
  constructor(name) {
    super(name || 'Warrior', 1, 120, 15, 10);
    this.class = 'Warrior';
    this.skills = ['Slash', 'Block'];
  }
  
  clone() {
    const clone = super.clone();
    clone.class = this.class;
    clone.skills = [...this.skills];
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.class = this.class;
    return clone;
  }
  
  getType() {
    return 'Warrior';
  }
  
  berserkerRage() {
    console.log(`${this.name} enters berserker rage! Attack increased.`);
    this.attack += 5;
  }
}

class Mage extends GameCharacter {
  constructor(name) {
    super(name || 'Mage', 1, 80, 20, 3);
    this.class = 'Mage';
    this.mana = 100;
    this.maxMana = 100;
    this.skills = ['Fireball', 'Ice Shard'];
  }
  
  clone() {
    const clone = super.clone();
    clone.class = this.class;
    clone.mana = this.mana;
    clone.maxMana = this.maxMana;
    clone.skills = [...this.skills];
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.class = this.class;
    clone.mana = this.mana;
    clone.maxMana = this.maxMana;
    return clone;
  }
  
  getType() {
    return 'Mage';
  }
  
  castSpell(spellName, manaCost) {
    if (this.mana >= manaCost) {
      this.mana -= manaCost;
      console.log(`${this.name} casts ${spellName}!`);
    } else {
      console.log(`${this.name} doesn't have enough mana!`);
    }
  }
  
  restoreMana(amount) {
    this.mana = Math.min(this.maxMana, this.mana + amount);
    console.log(`${this.name} restored ${amount} mana.`);
  }
}

class Archer extends GameCharacter {
  constructor(name) {
    super(name || 'Archer', 1, 90, 18, 6);
    this.class = 'Archer';
    this.arrows = 20;
    this.skills = ['Shoot', 'Aimed Shot'];
  }
  
  clone() {
    const clone = super.clone();
    clone.class = this.class;
    clone.arrows = this.arrows;
    clone.skills = [...this.skills];
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    clone.class = this.class;
    clone.arrows = this.arrows;
    return clone;
  }
  
  getType() {
    return 'Archer';
  }
  
  shoot() {
    if (this.arrows > 0) {
      this.arrows--;
      console.log(`${this.name} shoots an arrow! Arrows left: ${this.arrows}`);
    } else {
      console.log(`${this.name} is out of arrows!`);
    }
  }
  
  reloadArrows(amount) {
    this.arrows += amount;
    console.log(`${this.name} reloaded ${amount} arrows. Total: ${this.arrows}`);
  }
}

// 游戏角色原型管理器
class CharacterPrototypeManager {
  constructor() {
    this.prototypes = new Map();
    this.initializePrototypes();
  }
  
  initializePrototypes() {
    this.prototypes.set('warrior', new Warrior());
    this.prototypes.set('mage', new Mage());
    this.prototypes.set('archer', new Archer());
  }
  
  registerPrototype(name, prototype) {
    this.prototypes.set(name, prototype);
  }
  
  createCharacter(className, name) {
    const prototype = this.prototypes.get(className.toLowerCase());
    if (!prototype) {
      throw new Error(`Unknown character class: ${className}`);
    }
    
    const character = prototype.cloneDeep();
    character.name = name || `${className} ${Math.floor(Math.random() * 1000)}`;
    return character;
  }
  
  getAvailableClasses() {
    return Array.from(this.prototypes.keys());
  }
}

// 使用游戏角色原型系统
console.log("\n=== Game Character Prototype System ===");

const characterManager = new CharacterPrototypeManager();

// 创建游戏角色
const warrior = characterManager.createCharacter('warrior', 'Conan');
const mage = characterManager.createCharacter('mage', 'Gandalf');
const archer = characterManager.createCharacter('archer', 'Legolas');

// 显示角色信息
console.log("Created characters:");
console.log(`${warrior.name} (${warrior.class}): Level ${warrior.level}, HP ${warrior.health}/${warrior.maxHealth}`);
console.log(`${mage.name} (${mage.class}): Level ${mage.level}, HP ${mage.health}/${mage.maxHealth}, Mana ${mage.mana}/${mage.maxMana}`);
console.log(`${archer.name} (${archer.class}): Level ${archer.level}, HP ${archer.health}/${archer.maxHealth}, Arrows ${archer.arrows}`);

// 角色互动
console.log("\n--- Character Interactions ---");
warrior.gainExperience(150);
mage.castSpell('Fireball', 30);
archer.shoot();
archer.shoot();

// 学习新技能
warrior.learnSkill('Whirlwind');
mage.learnSkill('Lightning Bolt');
archer.learnSkill('Multishot');

// 显示更新后的信息
console.log("\n--- After Interactions ---");
console.log(`${warrior.name} skills: ${warrior.skills.join(', ')}`);
console.log(`${mage.name} mana: ${mage.mana}/${mage.maxMana}`);
console.log(`${archer.name} arrows: ${archer.arrows}`);
```

### 文档模板原型系统

```javascript
// 文档模板原型系统
class DocumentTemplate {
  constructor(title, content, metadata) {
    this.title = title || 'Untitled Document';
    this.content = content || '';
    this.metadata = metadata || {
      author: 'Unknown',
      created: new Date(),
      tags: [],
      version: '1.0'
    };
    this.styles = {
      font: 'Arial',
      fontSize: 12,
      color: 'black'
    };
    this.id = Date.now() + Math.random();
  }
  
  clone() {
    const clone = Object.create(Object.getPrototypeOf(this));
    return Object.assign(clone, this);
  }
  
  cloneDeep() {
    const clone = new this.constructor();
    clone.title = this.title;
    clone.content = this.content;
    clone.metadata = JSON.parse(JSON.stringify(this.metadata));
    clone.styles = JSON.parse(JSON.stringify(this.styles));
    clone.id = Date.now() + Math.random();
    return clone;
  }
  
  getType() {
    return 'DocumentTemplate';
  }
  
  setTitle(title) {
    this.title = title;
    return this;
  }
  
  setContent(content) {
    this.content = content;
    return this;
  }
  
  setAuthor(author) {
    this.metadata.author = author;
    return this;
  }
  
  addTag(tag) {
    if (!this.metadata.tags.includes(tag)) {
      this.metadata.tags.push(tag);
    }
    return this;
  }
  
  setStyle(property, value) {
    this.styles[property] = value;
    return this;
  }
  
  render() {
    console.log(`=== ${this.title} ===`);
    console.log(`Author: ${this.metadata.author}`);
    console.log(`Tags: ${this.metadata.tags.join(', ')}`);
    console.log(`Style: ${this.styles.font} ${this.styles.fontSize}pt`);
    console.log(`\n${this.content}`);
  }
  
  getInfo() {
    return {
      title: this.title,
      author: this.metadata.author,
      tags: [...this.metadata.tags],
      style: { ...this.styles },
      wordCount: this.content.split(/\s+/).length
    };
  }
}

// 具体文档模板
class ReportTemplate extends DocumentTemplate {
  constructor() {
    super('Monthly Report', '## Executive Summary\n\n## Key Metrics\n\n## Recommendations\n\n## Conclusion');
    this.metadata.tags = ['report', 'business'];
    this.styles.font = 'Times New Roman';
    this.styles.fontSize = 11;
  }
  
  clone() {
    const clone = super.clone();
    clone.metadata.tags = [...this.metadata.tags];
    clone.styles = { ...this.styles };
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    return clone;
  }
  
  getType() {
    return 'ReportTemplate';
  }
  
  addSection(sectionName) {
    this.content += `\n\n## ${sectionName}\n\n`;
    return this;
  }
}

class LetterTemplate extends DocumentTemplate {
  constructor() {
    super('Business Letter', '[Your Name]\n[Your Address]\n[City, State ZIP]\n\n[Date]\n\n[Recipient Name]\n[Recipient Address]\n[City, State ZIP]\n\nDear [Recipient Name],\n\n[Letter content]\n\nSincerely,\n[Your Name]');
    this.metadata.tags = ['letter', 'correspondence'];
    this.styles.font = 'Georgia';
    this.styles.fontSize = 12;
  }
  
  clone() {
    const clone = super.clone();
    clone.metadata.tags = [...this.metadata.tags];
    clone.styles = { ...this.styles };
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    return clone;
  }
  
  getType() {
    return 'LetterTemplate';
  }
  
  setAddress(address) {
    // 替换地址占位符
    this.content = this.content.replace('[Your Address]', address);
    return this;
  }
}

class ResumeTemplate extends DocumentTemplate {
  constructor() {
    super('Professional Resume', '# [Your Name]\n\n[Contact Information]\n\n## Summary\n\n## Experience\n\n## Education\n\n## Skills');
    this.metadata.tags = ['resume', 'cv', 'career'];
    this.styles.font = 'Calibri';
    this.styles.fontSize = 11;
  }
  
  clone() {
    const clone = super.clone();
    clone.metadata.tags = [...this.metadata.tags];
    clone.styles = { ...this.styles };
    return clone;
  }
  
  cloneDeep() {
    const clone = super.cloneDeep();
    return clone;
  }
  
  getType() {
    return 'ResumeTemplate';
  }
  
  addExperience(jobTitle, company, dates, description) {
    const experienceSection = `\n### ${jobTitle}\n**${company}** | ${dates}\n\n${description}\n`;
    this.content = this.content.replace('## Experience', `## Experience${experienceSection}`);
    return this;
  }
}

// 文档模板管理器
class DocumentTemplateManager {
  constructor() {
    this.templates = new Map();
    this.initializeTemplates();
  }
  
  initializeTemplates() {
    this.templates.set('report', new ReportTemplate());
    this.templates.set('letter', new LetterTemplate());
    this.templates.set('resume', new ResumeTemplate());
  }
  
  registerTemplate(name, template) {
    this.templates.set(name, template);
  }
  
  createDocument(templateName, overrides = {}) {
    const template = this.templates.get(templateName);
    if (!template) {
      throw new Error(`Template '${templateName}' not found`);
    }
    
    const document = template.cloneDeep();
    
    // 应用覆盖项
    if (overrides.title) document.setTitle(overrides.title);
    if (overrides.author) document.setAuthor(overrides.author);
    if (overrides.content) document.setContent(overrides.content);
    
    return document;
  }
  
  getAvailableTemplates() {
    return Array.from(this.templates.keys());
  }
  
  getTemplateInfo(templateName) {
    const template = this.templates.get(templateName);
    if (template) {
      return template.getInfo();
    }
    return null;
  }
}

// 使用文档模板系统
console.log("\n=== Document Template System ===");

const templateManager = new DocumentTemplateManager();

// 查看可用模板
console.log("Available templates:", templateManager.getAvailableTemplates());

// 创建文档
const report = templateManager.createDocument('report', {
  title: 'Q3 Sales Report',
  author: 'John Smith'
});

const letter = templateManager.createDocument('letter', {
  title: 'Job Application Letter',
  author: 'Jane Doe'
});

const resume = templateManager.createDocument('resume', {
  title: 'Software Engineer Resume',
  author: 'Alice Johnson'
});

// 自定义文档
report
  .addSection('Market Analysis')
  .addSection('Competitor Review')
  .addTag('Q3-2023')
  .setStyle('color', 'darkblue');

letter
  .setAddress('123 Main St, Anytown, ST 12345')
  .addTag('application')
  .setStyle('font', 'Arial');

resume
  .addExperience('Senior Developer', 'Tech Corp', '2020-Present', 'Led development team of 5 engineers')
  .addTag('software')
  .setStyle('fontSize', 10);

// 渲染文档
console.log("\n--- Generated Documents ---");
report.render();
console.log("\n" + "=".repeat(50) + "\n");
letter.render();
console.log("\n" + "=".repeat(50) + "\n");
resume.render();

// 显示文档信息
console.log("\n--- Document Information ---");
console.log("Report info:", report.getInfo());
console.log("Letter info:", letter.getInfo());
console.log("Resume info:", resume.getInfo());
```

## 总结

原型模式是一种强大的创建型设计模式，它通过复制现有对象来创建新对象，避免了复杂的初始化过程。这种模式特别适用于以下场景：

1. **创建成本较高的对象**：当对象初始化过程复杂或耗时时
2. **需要创建大量相似对象**：通过克隆避免重复的初始化过程
3. **运行时动态配置对象**：可以在运行时通过注册原型来动态配置系统
4. **避免子类化**：通过原型克隆避免创建大量的工厂类

原型模式的主要优势包括：

1. **性能优势**：对于复杂对象，克隆通常比重新创建更高效
2. **运行时灵活性**：可以在运行时添加和删除原型
3. **简化对象创建**：避免了复杂的初始化过程
4. **动态配置**：支持运行时动态配置应用

但也要注意其局限性：

1. **克隆复杂性**：深拷贝复杂对象可能很困难
2. **循环引用问题**：需要特殊处理对象间的循环引用
3. **安全性考虑**：不正确的原型使用可能导致安全问题

正确使用原型模式可以大大提高系统的性能和灵活性，特别是在需要创建大量相似对象或对象初始化成本较高的场景中。它是面向对象设计中处理对象创建问题的重要工具。