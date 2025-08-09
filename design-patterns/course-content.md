# 设计模式全面学习指南

## 1. 课程介绍

### 1.1 课程目标
- 为初学者提供设计模式的基础知识和实践方法。
- 为进阶者提供深入理解和应用设计模式的技巧和案例。

### 1.2 适用人群
- 初学者：对设计模式感兴趣，希望系统学习设计模式的开发者。
- 进阶者：有一定编程基础，希望深入理解和灵活应用设计模式的开发者。

## 2. 课程内容

### 2.1 第一部分：设计模式相关介绍

#### 2.1.1 设计模式的概述
- **定义**：设计模式是在软件设计过程中，针对常见问题的可重用解决方案。
- **起源**：设计模式的概念最早由"设计模式四人组"（Gang of Four, GoF）在《设计模式：可复用面向对象软件的基础》一书中提出。
- **重要性**：设计模式可以提高代码的可读性、可维护性和可扩展性。

#### 2.1.2 UML图
- **UML简介**：UML（Unified Modeling Language，统一建模语言）是一种标准化的建模语言，用于软件系统的可视化建模。
- **UML图类型**：
  - **类图**：展示系统的静态结构，包括类、接口、协作及其关系。
  - **顺序图**：描述对象之间的交互顺序。
  - **状态图**：展示一个对象在其生命周期中的状态变化。
  - **用例图**：描述系统功能和外部参与者之间的交互。

#### 2.1.3 软件设计原则
- **单一职责原则（SRP）**：一个类应该只有一个引起它变化的原因。
- **开放封闭原则（OCP）**：软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。
- **里氏替换原则（LSP）**：子类型必须能够替换其基类型。
- **依赖倒置原则（DIP）**：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。
- **接口隔离原则（ISP）**：客户端不应该被迫依赖它不需要的接口。
- **迪米特法则（LoD）**：一个对象应当对其他对象有最少的了解。

### 2.2 第二部分：设计模式的学习

#### 2.2.1 创建者模式
- **工厂方法模式**：
  - **定义**：定义一个用于创建对象的接口，让子类决定实例化哪一个类。
  - **应用场景**：需要创建的对象具有共同的父类。
  - **代码示例**：
    ```java
    // 工厂接口
    interface Creator {
        Product createProduct();
    }

    // 具体工厂
    class ConcreteCreator implements Creator {
        @Override
        public Product createProduct() {
            return new ConcreteProduct();
        }
    }

    // 产品接口
    interface Product {
        void operation();
    }

    // 具体产品
    class ConcreteProduct implements Product {
        @Override
        public void operation() {
            System.out.println("ConcreteProduct operation");
        }
    }
    ```
- **抽象工厂模式**：
  - **定义**：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
  - **应用场景**：需要创建一组相关或相互依赖的对象。
  - **代码示例**：
    ```java
    // 抽象工厂接口
    interface AbstractFactory {
        ProductA createProductA();
        ProductB createProductB();
    }

    // 具体工厂
    class ConcreteFactory1 implements AbstractFactory {
        @Override
        public ProductA createProductA() {
            return new ProductA1();
        }

        @Override
        public ProductB createProductB() {
            return new ProductB1();
        }
    }

    // 产品A接口
    interface ProductA {
        void operationA();
    }

    // 具体产品A1
    class ProductA1 implements ProductA {
        @Override
        public void operationA() {
            System.out.println("ProductA1 operationA");
        }
    }

    // 产品B接口
    interface ProductB {
        void operationB();
    }

    // 具体产品B1
    class ProductB1 implements ProductB {
        @Override
        public void operationB() {
            System.out.println("ProductB1 operationB");
        }
    }
    ```
- **建造者模式**：
  - **定义**：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。
  - **应用场景**：需要逐步构建一个复杂的对象。
  - **代码示例**：
    ```java
    // 产品类
    class Product {
        private List<String> parts = new ArrayList<>();

        public void add(String part) {
            parts.add(part);
        }

        public void show() {
            System.out.println("Product parts: " + String.join(", ", parts));
        }
    }

    // 建造者接口
    interface Builder {
        void buildPartA();
        void buildPartB();
        Product getResult();
    }

    // 具体建造者
    class ConcreteBuilder implements Builder {
        private Product product = new Product();

        @Override
        public void buildPartA() {
            product.add("PartA");
        }

        @Override
        public void buildPartB() {
            product.add("PartB");
        }

        @Override
        public Product getResult() {
            return product;
        }
    }

    // 导演类
    class Director {
        private Builder builder;

        public Director(Builder builder) {
            this.builder = builder;
        }

        public void construct() {
            builder.buildPartA();
            builder.buildPartB();
        }
    }
    ```
- **原型模式**：
  - **定义**：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
  - **应用场景**：需要创建大量相似对象时。
  - **代码示例**：
    ```java
    // 原型接口
    interface Prototype {
        Prototype clone();
    }

    // 具体原型
    class ConcretePrototype implements Prototype {
        private String field;

        public ConcretePrototype(String field) {
            this.field = field;
        }

        @Override
        public Prototype clone() {
            return new ConcretePrototype(this.field);
        }

        public String getField() {
            return field;
        }

        public void setField(String field) {
            this.field = field;
        }
    }
    ```

#### 2.2.2 结构型模式
- **适配器模式**：
  - **定义**：将一个类的接口转换成客户希望的另一个接口。
  - **应用场景**：希望复用一些现存的类，但接口又与复用环境要求不一致。
  - **代码示例**：
    ```java
    // 目标接口
    interface Target {
        void request();
    }

    // 已有的类
    class Adaptee {
        public void specificRequest() {
            System.out.println("Adaptee specificRequest");
        }
    }

    // 适配器
    class Adapter implements Target {
        private Adaptee adaptee;

        public Adapter(Adaptee adaptee) {
            this.adaptee = adaptee;
        }

        @Override
        public void request() {
            adaptee.specificRequest();
        }
    }
    ```
- **装饰器模式**：
  - **定义**：动态地给一个对象添加一些额外的职责。
  - **应用场景**：需要增加由一些基本功能的组合而产生的非常丰富的功能。
  - **代码示例**：
    ```java
    // 组件接口
    interface Component {
        void operation();
    }

    // 具体组件
    class ConcreteComponent implements Component {
        @Override
        public void operation() {
            System.out.println("ConcreteComponent operation");
        }
    }

    // 装饰器抽象类
    abstract class Decorator implements Component {
        protected Component component;

        public Decorator(Component component) {
            this.component = component;
        }

        @Override
        public void operation() {
            component.operation();
        }
    }

    // 具体装饰器
    class ConcreteDecorator extends Decorator {
        public ConcreteDecorator(Component component) {
            super(component);
        }

        @Override
        public void operation() {
            super.operation();
            addedBehavior();
        }

        private void addedBehavior() {
            System.out.println("ConcreteDecorator addedBehavior");
        }
    }
    ```
- **代理模式**：
  - **定义**：为其他对象提供一种代理以控制对这个对象的访问。
  - **应用场景**：需要在访问对象时加入额外的操作。
  - **代码示例**：
    ```java
    // 真实主题
    class RealSubject implements Subject {
        @Override
        public void request() {
            System.out.println("RealSubject request");
        }
    }

    // 代理类
    class Proxy implements Subject {
        private RealSubject realSubject;

        @Override
        public void request() {
            if (realSubject == null) {
                realSubject = new RealSubject();
            }
            preRequest();
            realSubject.request();
            postRequest();
        }

        private void preRequest() {
            System.out.println("Proxy preRequest");
        }

        private void postRequest() {
            System.out.println("Proxy postRequest");
        }
    }
    ```
- **外观模式**：
  - **定义**：为子系统中的一组接口提供一个一致的界面。
  - **应用场景**：需要简化复杂系统的接口。
  - **代码示例**：
    ```java
    // 子系统类
    class SubSystemOne {
        public void methodOne() {
            System.out.println("SubSystemOne methodOne");
        }
    }

    class SubSystemTwo {
        public void methodTwo() {
            System.out.println("SubSystemTwo methodTwo");
        }
    }

    class SubSystemThree {
        public void methodThree() {
            System.out.println("SubSystemThree methodThree");
        }
    }

    // 外观类
    class Facade {
        private SubSystemOne one;
        private SubSystemTwo two;
        private SubSystemThree three;

        public Facade() {
            this.one = new SubSystemOne();
            this.two = new SubSystemTwo();
            this.three = new SubSystemThree();
        }

        public void methodA() {
            System.out.println("Facade methodA");
            one.methodOne();
            two.methodTwo();
        }

        public void methodB() {
            System.out.println("Facade methodB");
            two.methodTwo();
            three.methodThree();
        }
    }
    ```
- **桥接模式**：
  - **定义**：将抽象部分与它的实现部分分离，使它们都可以独立地变化。
  - **应用场景**：不希望在抽象和它的实现部分之间有一个固定的绑定关系。
  - **代码示例**：
    ```java
    // 实现类接口
    interface Implementor {
        void operationImpl();
    }

    // 具体实现类
    class ConcreteImplementorA implements Implementor {
        @Override
        public void operationImpl() {
            System.out.println("ConcreteImplementorA operationImpl");
        }
    }

    class ConcreteImplementorB implements Implementor {
        @Override
        public void operationImpl() {
            System.out.println("ConcreteImplementorB operationImpl");
        }
    }

    // 抽象类
    abstract class Abstraction {
        protected Implementor implementor;

        public Abstraction(Implementor implementor) {
            this.implementor = implementor;
        }

        public abstract void operation();
    }

    // 扩充抽象类
    class RefinedAbstraction extends Abstraction {
        public RefinedAbstraction(Implementor implementor) {
            super(implementor);
        }

        @Override
        public void operation() {
            implementor.operationImpl();
        }
    }
    ```
- **组合模式**：
  - **定义**：将对象组合成树形结构以表示"部分-整体"的层次结构。
  - **应用场景**：需求是体现部分与整体层次的结构。
  - **代码示例**：
    ```java
    // 组件抽象类
    abstract class Component {
        protected String name;

        public Component(String name) {
            this.name = name;
        }

        public abstract void add(Component c);
        public abstract void remove(Component c);
        public abstract void display(int depth);
    }

    // 叶子类
    class Leaf extends Component {
        public Leaf(String name) {
            super(name);
        }

        @Override
        public void add(Component c) {
            System.out.println("Cannot add to a leaf");
        }

        @Override
        public void remove(Component c) {
            System.out.println("Cannot remove from a leaf");
        }

        @Override
        public void display(int depth) {
            System.out.println("-".repeat(depth) + name);
        }
    }

    // 容器类
    class Composite extends Component {
        private List<Component> children = new ArrayList<>();

        public Composite(String name) {
            super(name);
        }

        @Override
        public void add(Component c) {
            children.add(c);
        }

        @Override
        public void remove(Component c) {
            children.remove(c);
        }

        @Override
        public void display(int depth) {
            System.out.println("-".repeat(depth) + name);
            for (Component component : children) {
                component.display(depth + 2);
            }
        }
    }
    ```
- **享元模式**：
  - **定义**：运用共享技术有效地支持大量细粒度的对象。
  - **应用场景**：系统中有大量相似对象，需要缓冲池的场景。
  - **代码示例**：
    ```java
    // 享元抽象类
    abstract class Flyweight {
        public abstract void operation(int extrinsicState);
    }

    // 具体享元类
    class ConcreteFlyweight extends Flyweight {
        @Override
        public void operation(int extrinsicState) {
            System.out.println("ConcreteFlyweight: " + extrinsicState);
        }
    }

    // 享元工厂类
    class FlyweightFactory {
        private Map<String, Flyweight> flyweights = new HashMap<>();

        public Flyweight getFlyweight(String key) {
            if (!flyweights.containsKey(key)) {
                flyweights.put(key, new ConcreteFlyweight());
            }
            return flyweights.get(key);
        }
    }
    ```

#### 2.2.3 行为型模式
- **策略模式**：
  - **定义**：定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。
  - **应用场景**：需要在几种算法中选择一种。
  - **代码示例**：
    ```java
    // 策略接口
    interface Strategy {
        void algorithmInterface();
    }

    // 具体策略类
    class ConcreteStrategyA implements Strategy {
        @Override
        public void algorithmInterface() {
            System.out.println("ConcreteStrategyA algorithmInterface");
        }
    }

    class ConcreteStrategyB implements Strategy {
        @Override
        public void algorithmInterface() {
            System.out.println("ConcreteStrategyB algorithmInterface");
        }
    }

    // 上下文类
    class Context {
        private Strategy strategy;

        public Context(Strategy strategy) {
            this.strategy = strategy;
        }

        public void contextInterface() {
            strategy.algorithmInterface();
        }
    }
    ```
- **模板方法模式**：
  - **定义**：定义一个操作中的算法骨架，而将一些步骤延迟到子类中。
  - **应用场景**：一次性实现一个算法的不变部分，并将可变的行为留给子类来实现。
  - **代码示例**：
    ```java
    // 抽象模板类
    abstract class AbstractClass {
        public abstract void primitiveOperation1();
        public abstract void primitiveOperation2();

        public void templateMethod() {
            primitiveOperation1();
            primitiveOperation2();
        }
    }

    // 具体模板类
    class ConcreteClassA extends AbstractClass {
        @Override
        public void primitiveOperation1() {
            System.out.println("ConcreteClassA primitiveOperation1");
        }

        @Override
        public void primitiveOperation2() {
            System.out.println("ConcreteClassA primitiveOperation2");
        }
    }

    class ConcreteClassB extends AbstractClass {
        @Override
        public void primitiveOperation1() {
            System.out.println("ConcreteClassB primitiveOperation1");
        }

        @Override
        public void primitiveOperation2() {
            System.out.println("ConcreteClassB primitiveOperation2");
        }
    }
    ```
- **观察者模式**：
  - **定义**：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
  - **应用场景**：当一个对象的改变需要同时改变其他对象，而不知道具体有多少对象有待改变。
  - **代码示例**：
    ```java
    // 抽象观察者
    interface Observer {
        void update();
    }

    // 具体观察者
    class ConcreteObserver implements Observer {
        private String name;
        private String observerState;
        private ConcreteSubject subject;

        public ConcreteObserver(ConcreteSubject subject, String name) {
            this.subject = subject;
            this.name = name;
        }

        @Override
        public void update() {
            observerState = subject.getSubjectState();
            System.out.println("Observer " + name + "'s new state is " + observerState);
        }
    }

    // 抽象主题
    abstract class Subject {
        private List<Observer> observers = new ArrayList<>();

        public void attach(Observer observer) {
            observers.add(observer);
        }

        public void detach(Observer observer) {
            observers.remove(observer);
        }

        public void notifyObservers() {
            for (Observer observer : observers) {
                observer.update();
            }
        }
    }

    // 具体主题
    class ConcreteSubject extends Subject {
        private String subjectState;

        public String getSubjectState() {
            return subjectState;
        }

        public void setSubjectState(String subjectState) {
            this.subjectState = subjectState;
            notifyObservers();
        }
    }
    ```
- **迭代器模式**：
  - **定义**：提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。
  - **应用场景**：需要访问一个聚合对象，而不管这些对象是什么都需要遍历的时候。
  - **代码示例**：
    ```java
    // 迭代器接口
    interface Iterator<T> {
        T first();
        T next();
        boolean isDone();
        T currentItem();
    }

    // 聚集接口
    interface Aggregate<T> {
        Iterator<T> createIterator();
    }

    // 具体迭代器类
    class ConcreteIterator<T> implements Iterator<T> {
        private ConcreteAggregate<T> aggregate;
        private int current = 0;

        public ConcreteIterator(ConcreteAggregate<T> aggregate) {
            this.aggregate = aggregate;
        }

        @Override
        public T first() {
            return aggregate.get(0);
        }

        @Override
        public T next() {
            T ret = null;
            current++;
            if (current < aggregate.getCount()) {
                ret = aggregate.get(current);
            }
            return ret;
        }

        @Override
        public boolean isDone() {
            return current >= aggregate.getCount();
        }

        @Override
        public T currentItem() {
            return aggregate.get(current);
        }
    }

    // 具体聚集类
    class ConcreteAggregate<T> implements Aggregate<T> {
        private List<T> items = new ArrayList<>();

        @Override
        public Iterator<T> createIterator() {
            return new ConcreteIterator<>(this);
        }

        public int getCount() {
            return items.size();
        }

        public T get(int index) {
            return items.get(index);
        }

        public void set(int index, T value) {
            items.add(index, value);
        }
    }
    ```
- **责任链模式**：
  - **定义**：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。
  - **应用场景**：有多个对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定。
  - **代码示例**：
    ```java
    // 处理者抽象类
    abstract class Handler {
        protected Handler successor;

        public void setSuccessor(Handler successor) {
            this.successor = successor;
        }

        public abstract void handleRequest(int request);
    }

    // 具体处理者类
    class ConcreteHandler1 extends Handler {
        @Override
        public void handleRequest(int request) {
            if (request >= 0 && request < 10) {
                System.out.println("ConcreteHandler1 handled request " + request);
            } else if (successor != null) {
                successor.handleRequest(request);
            }
        }
    }

    class ConcreteHandler2 extends Handler {
        @Override
        public void handleRequest(int request) {
            if (request >= 10 && request < 20) {
                System.out.println("ConcreteHandler2 handled request " + request);
            } else if (successor != null) {
                successor.handleRequest(request);
            }
        }
    }

    class ConcreteHandler3 extends Handler {
        @Override
        public void handleRequest(int request) {
            if (request >= 20 && request < 30) {
                System.out.println("ConcreteHandler3 handled request " + request);
            } else if (successor != null) {
                successor.handleRequest(request);
            }
        }
    }
    ```
- **命令模式**：
  - **定义**：将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化。
  - **应用场景**：需要抽象出待执行的动作以参数化某对象。
  - **代码示例**：
    ```java
    // 命令接口
    interface Command {
        void execute();
    }

    // 具体命令类
    class ConcreteCommand implements Command {
        private Receiver receiver;

        public ConcreteCommand(Receiver receiver) {
            this.receiver = receiver;
        }

        @Override
        public void execute() {
            receiver.action();
        }
    }

    // 接收者类
    class Receiver {
        public void action() {
            System.out.println("Receiver action");
        }
    }

    // 调用者类
    class Invoker {
        private Command command;

        public void setCommand(Command command) {
            this.command = command;
        }

        public void executeCommand() {
            command.execute();
        }
    }
    ```
- **备忘录模式**：
  - **定义**：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。
  - **应用场景**：必须保存一个对象在某一个时刻的(部分)状态，这样以后需要时它才能恢复到先前的状态。
  - **代码示例**：
    ```java
    // 备忘录类
    class Memento {
        private String state;

        public Memento(String state) {
            this.state = state;
        }

        public String getState() {
            return state;
        }
    }

    // 发起人类
    class Originator {
        private String state;

        public String getState() {
            return state;
        }

        public void setState(String state) {
            this.state = state;
        }

        public Memento createMemento() {
            return new Memento(state);
        }

        public void setMemento(Memento memento) {
            state = memento.getState();
        }

        public void show() {
            System.out.println("State: " + state);
        }
    }

    // 管理者类
    class Caretaker {
        private Memento memento;

        public Memento getMemento() {
            return memento;
        }

        public void setMemento(Memento memento) {
            this.memento = memento;
        }
    }
    ```
- **状态模式**：
  - **定义**：允许一个对象在其内部状态改变时改变它的行为。
  - **应用场景**：一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为。
  - **代码示例**：
    ```java
    // 状态抽象类
    abstract class State {
        public abstract void handle(Context context);
    }

    // 具体状态类
    class ConcreteStateA extends State {
        @Override
        public void handle(Context context) {
            context.setState(new ConcreteStateB());
        }
    }

    class ConcreteStateB extends State {
        @Override
        public void handle(Context context) {
            context.setState(new ConcreteStateA());
        }
    }

    // 上下文类
    class Context {
        private State state;

        public Context(State state) {
            this.state = state;
        }

        public State getState() {
            return state;
        }

        public void setState(State state) {
            this.state = state;
            System.out.println("State changed to " + state.getClass().getSimpleName());
        }

        public void request() {
            state.handle(this);
        }
    }
    ```
- **访问者模式**：
  - **定义**：表示一个作用于某对象结构中的各元素的操作。
  - **应用场景**：一个对象结构包含很多类对象，它们有不同的接口，而想对这些对象实施一些依赖于其具体类的操作。
  - **代码示例**：
    ```java
    // 访问者接口
    interface Visitor {
        void visitConcreteElementA(ConcreteElementA element);
        void visitConcreteElementB(ConcreteElementB element);
    }

    // 元素接口
    interface Element {
        void accept(Visitor visitor);
    }

    // 具体访问者类
    class ConcreteVisitor1 implements Visitor {
        @Override
        public void visitConcreteElementA(ConcreteElementA element) {
            System.out.println("ConcreteVisitor1 visits ConcreteElementA");
        }

        @Override
        public void visitConcreteElementB(ConcreteElementB element) {
            System.out.println("ConcreteVisitor1 visits ConcreteElementB");
        }
    }

    class ConcreteVisitor2 implements Visitor {
        @Override
        public void visitConcreteElementA(ConcreteElementA element) {
            System.out.println("ConcreteVisitor2 visits ConcreteElementA");
        }

        @Override
        public void visitConcreteElementB(ConcreteElementB element) {
            System.out.println("ConcreteVisitor2 visits ConcreteElementB");
        }
    }

    // 具体元素类
    class ConcreteElementA implements Element {
        @Override
        public void accept(Visitor visitor) {
            visitor.visitConcreteElementA(this);
        }

        public void operationA() {
            System.out.println("ConcreteElementA operationA");
        }
    }

    class ConcreteElementB implements Element {
        @Override
        public void accept(Visitor visitor) {
            visitor.visitConcreteElementB(this);
        }

        public void operationB() {
            System.out.println("ConcreteElementB operationB");
        }
    }
    ```
- **中介者模式**：
  - **定义**：用一个中介对象来封装一系列的对象交互。
  - **应用场景**：一组定义良好的对象，现在要进行复杂的通信。
  - **代码示例**：
    ```java
    // 中介者抽象类
    abstract class Mediator {
        public abstract void send(String message, Colleague colleague);
    }

    // 同事抽象类
    abstract class Colleague {
        protected Mediator mediator;

        public Colleague(Mediator mediator) {
            this.mediator = mediator;
        }
    }

    // 具体同事类
    class ConcreteColleague1 extends Colleague {
        public ConcreteColleague1(Mediator mediator) {
            super(mediator);
        }

        public void send(String message) {
            mediator.send(message, this);
        }

        public void notify(String message) {
            System.out.println("Colleague1 gets message: " + message);
        }
    }

    class ConcreteColleague2 extends Colleague {
        public ConcreteColleague2(Mediator mediator) {
            super(mediator);
        }

        public void send(String message) {
            mediator.send(message, this);
        }

        public void notify(String message) {
            System.out.println("Colleague2 gets message: " + message);
        }
    }

    // 具体中介者类
    class ConcreteMediator extends Mediator {
        private ConcreteColleague1 colleague1;
        private ConcreteColleague2 colleague2;

        public void setColleague1(ConcreteColleague1 colleague1) {
            this.colleague1 = colleague1;
        }

        public void setColleague2(ConcreteColleague2 colleague2) {
            this.colleague2 = colleague2;
        }

        @Override
        public void send(String message, Colleague colleague) {
            if (colleague == colleague1) {
                colleague2.notify(message);
            } else {
                colleague1.notify(message);
            }
        }
    }
    ```
- **解释器模式**：
  - **定义**：给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。
  - **应用场景**：如果某一特定类型的问题发生的频率足够高，那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。
  - **代码示例**：
    ```java
    // 抽象表达式类
    abstract class AbstractExpression {
        public abstract void interpret(Context context);
    }

    // 终结符表达式类
    class TerminalExpression extends AbstractExpression {
        @Override
        public void interpret(Context context) {
            System.out.println("Terminal expression");
        }
    }

    // 非终结符表达式类
    class NonterminalExpression extends AbstractExpression {
        @Override
        public void interpret(Context context) {
            System.out.println("Nonterminal expression");
        }
    }

    // 环境类
    class Context {
        private String input;
        private String output;

        public Context(String input) {
            this.input = input;
        }

        public String getInput() {
            return input;
        }

        public void setInput(String input) {
            this.input = input;
        }

        public String getOutput() {
            return output;
        }

        public void setOutput(String output) {
            this.output = output;
        }
    }
    ```