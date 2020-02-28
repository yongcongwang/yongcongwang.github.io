---
title: Design Patterns in C++
date: 2019-06-05 20:44:42
categories: algorithm
---

# Strategy Pattern
## Intent
- Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from the clients that use it.
- Capture the abstraction in an interface, bury implementation details in derived classes.

```puml
@startuml
class Context {
- strategy_ : Strategy *
+ contextInterface() : void
}

class Strategy {
+ strategyInterface() : void
}

class ConcreteStrategyA {
+ strategyInterface() : void
}

class ConcreteStrategyB {
+ strategyInterface() : void
}

Context *-- Strategy
Strategy <|-- ConcreteStrategyA
Strategy <|-- ConcreteStrategyB
@enduml
```

```C++
// By yongcong.wang @ 09/06/2019
#include <iostream>

class Strategy {
 public:
  explicit Strategy() {}
  virtual ~Strategy() {}

  virtual void exec() = 0;

 private:
  Strategy(const Strategy &rhs);
  Strategy &operator=(const Strategy &rhs);
};

class ConcreteStrategyA : public Strategy {
 public:
  explicit ConcreteStrategyA () : Strategy() {}
  ~ConcreteStrategyA() {}

  void exec() {
    std::cout << "StrategyA: exec()" << std::endl;
  }

 private:
  ConcreteStrategyA(const ConcreteStrategyA &rhs);
  ConcreteStrategyA &operator=(const ConcreteStrategyA &rhs);
};

class ConcreteStrategyB : public Strategy {
 public:
  explicit ConcreteStrategyB () : Strategy() {}
  ~ConcreteStrategyB() {}

  void exec() {
    std::cout << "StrategyB: exec()" << std::endl;
  }

 private:
  ConcreteStrategyB(const ConcreteStrategyB &rhs);
  ConcreteStrategyB &operator=(const ConcreteStrategyB &rhs);
};

class Context {
 public:
  explicit Context(Strategy *strategy) : strategy_(strategy) {}
  ~Context() {}

  void setStrategy(Strategy *strategy) {
    strategy_ = strategy;
  }

  void exec() {
    strategy_->exec();
  }

 private:
  Context(const Context &rhs);
  Context &operator=(const Context &rhs);

  Strategy *strategy_;
};

int main() {
  ConcreteStrategyA str_a;
  ConcreteStrategyB str_b;

  std::cout << "set a strategy" << std::endl;
  Context cont(&str_a);
  cont.exec();
  std::cout << "set b strategy" << std::endl;
  cont.setStrategy(&str_b);
  cont.exec();

  return 0;
}

```
output: 

```shell
set a strategy
StrategyA: exec()
set b strategy
StrategyB: exec()
```

# Observer Pattern
Observer pattern is used when there is one-to-many relationship between objects such as if one object is modified, its dependent objects are to be notified automatically.

```puml

@startuml
class Subject {
+ registerObserver(Observer *) : void
+ removeObserver(Observer *) : void
+ notifyObserver() : void
}

class ConcreteSubjectA {
- observer_list : vector<Observer *> 
+ registryObserver(Observer *) : void
+ removeObserver(Observer *) : void
+ nofityObserver() : void
}

class Observer {
+ updata() : void  
}

class ConcreteObserverA {
+ update() : void
}

class ConcreteObserverB {
+ update() : void
}
  
Subject <|-- ConcreteSubjectA
Subject --> Observer
Observer <|-- ConcreteObserverA
ConcreteSubject <-- ConcreteObserverA
Observer <|-- ConcreteObserverB
ConcreteSubject <-- ConcreteObserverB
@enduml

```

```C++
// By yongcong.wang @ 09/06/2019
#include <iostream>
#include <vector>

class Observer {
 public:
  explicit Observer() {}
  virtual ~Observer() {}

  virtual void update() = 0;

 private:
  Observer(const Observer &rhs);
  Observer &operator=(const Observer &rhs);
};

class ConcreteObserverA : public Observer {
 public:
  explicit ConcreteObserverA() : Observer() {}
  virtual ~ConcreteObserverA() {}

  void update() {
    std::cout << "observer A heard" << std::endl;
  }

 private:
  ConcreteObserverA(const ConcreteObserverA &rhs);
  ConcreteObserverA &operator=(const ConcreteObserverA &rhs);
};

class ConcreteObserverB : public Observer {
 public:
  explicit ConcreteObserverB() : Observer() {}
  virtual ~ConcreteObserverB() {}

  void update() {
    std::cout << "observer B heard" << std::endl;
  }

 private:
  ConcreteObserverB(const ConcreteObserverB &rhs);
  ConcreteObserverB &operator=(const ConcreteObserverB &rhs);
};

class Subject {
 public:
  explicit Subject() {}
  virtual ~Subject() {}

  virtual void registerObserver(Observer *observer) = 0;
  virtual void removeObserver(Observer *observer) = 0;
  virtual void notifyObserver() = 0;

 private:
  Subject(const Subject &rhs);
  Subject &operator=(const Subject &rhs);
};

class ConcreteSubjectA : public Subject {
 public:
  explicit ConcreteSubjectA() : Subject() {}
  virtual ~ConcreteSubjectA() {}

  void registerObserver(Observer *observer) {
    observer_list_.push_back(observer);
  }

  void removeObserver(Observer *observer) {
    for (auto it = observer_list_.begin(); it != observer_list_.end(); ++it) {
      if (*it == observer) {
        observer_list_.erase(it);
        return;
      }
    }
  }

  void notifyObserver() {
    for (auto it = observer_list_.begin(); it != observer_list_.end(); ++it) {
      (*it)->update();
    }
  }

 private:
  ConcreteSubjectA(const ConcreteSubjectA &rhs);
  ConcreteSubjectA &operator=(const ConcreteSubjectA &rhs);

  std::vector<Observer *> observer_list_;
};

int main() {
  ConcreteObserverA obsr_a;
  ConcreteObserverB obsr_b;
  ConcreteSubjectA subj;
  std::cout << "add two observer and update" << std::endl;
  subj.registerObserver(&obsr_a);
  subj.registerObserver(&obsr_b);
  subj.notifyObserver();
  std::cout << "remove ovserver A and update" << std::endl;
  subj.removeObserver(&obsr_a);
  subj.notifyObserver();

  return 0;
}

```

output:

```shell
add two observer and update
observer A heard
observer B heard
remove ovserver A and update
observer B heard
```

# Decorator Pattern
Decorator pattern allows a user to add new functionality to an existing object without altering its structure. This type of design pattern comes under structural pattern as this pattern acts as a wrapper to existing class.
This pattern creates a decorator class which wraps the original class and provides additional functionality keeping class methods signature inact;

```puml

class Component {
+ methodA() = 0 : virtual void
+ methodB() = 0: virtual void
}

class ConcreteComponent {
+ methodA() : void
+ methodB() : void
}

class Decorator {
+ methodA() : void
+ methodB() : void
- Component wrappedObj;
}

class ConcreteDecoratorA {
+ methodA() : void
+ methodB() : void
}

class ConcreteDecoratorB {
+ methodA() : void
+ methodB() : void
}

Component <|-- ConcreteComponent
Component <|-- Decorator
Component <-- Decorator
ConcreteDecoratorA --|> Decorator
ConcreteDecoratorB --|> Decorator

```

```C++
// By yongcong.wang @ 28/07/2019
#include <iostream>
#include <string>
#include <memory>

class Component {
 public:
  explicit Component() {};
  virtual ~Component() {};
  Component(const Component &rhs) = delete;
  Component &operator=(const Component &rhs) = delete;

  virtual std::string methodA() = 0;
  virtual std::string methodB() = 0;

 private:
};

class ConcreteComponentA : public Component {
 public:
  ConcreteComponentA() : Component() {};
  ~ConcreteComponentA() {};
  ConcreteComponentA(const ConcreteComponentA &rhs) = delete;
  ConcreteComponentA &operator=(const ConcreteComponentA &rhs) = delete;

  std::string methodA() {
    return "ConcreteComponentA methodA ";
  };

  std::string methodB() {
    return "ConcreteComponentA methodB ";
  };

 private:
};

class Decorator : public Component {
 public:
  Decorator(std::shared_ptr<Component> component) : component_(component) {};
  ~Decorator() {};
  Decorator(const Decorator &rhs) = delete;
  Decorator &operator=(const Decorator &rhs) = delete;

  std::string methodA() {
    return component_->methodA();
  };

  std::string methodB() {
    return component_->methodB();
  };

 protected:
  std::shared_ptr<Component> component_;
};

class ConcreteDecoratorA : public Decorator {
 public:
  ConcreteDecoratorA(std::shared_ptr<Component> component) : Decorator(component) {};
  ~ConcreteDecoratorA() {};
  ConcreteDecoratorA(const ConcreteDecoratorA &rhs) = delete;
  ConcreteDecoratorA &operator=(const ConcreteDecoratorA &rhs) = delete;

  std::string methodA() {
    return component_->methodA() + "ConcreteDecoratorA methodA ";
  }
  std::string methodB() {
    return component_->methodB() + "ConcreteDecoratorB methodB ";
  }

 private:
};

class ConcreteDecoratorB : public Decorator {
 public:
  ConcreteDecoratorB(std::shared_ptr<Component> component) : Decorator(component) {};
  ~ConcreteDecoratorB() {};
  ConcreteDecoratorB(const ConcreteDecoratorB &rhs) = delete;
  ConcreteDecoratorB &operator=(const ConcreteDecoratorB &rhs) = delete;

  std::string methodA() {
    return component_->methodA() + "ConcreteDecoratorB methodA ";
  }
  std::string methodB() {
    return component_->methodB() + "ConcreteDecoratorB methodB ";
  }

 private:
};

int main() {
  std::shared_ptr<ConcreteComponentA> ptr_component_a = std::make_shared<ConcreteComponentA>();
  std::cout << ptr_component_a->methodA() << ", " << ptr_component_a->methodB() << std::endl;
  
  std::shared_ptr<ConcreteDecoratorA> ptr_decorator_a = std::make_shared<ConcreteDecoratorA>(
      ptr_component_a);
  std::cout << ptr_decorator_a->methodA() << ", " << ptr_decorator_a->methodB() << std::endl;

  std::shared_ptr<ConcreteDecoratorB> ptr_decorator_b = std::make_shared<ConcreteDecoratorB>(
      ptr_component_a);
  std::cout << ptr_decorator_b->methodA() << ", " << ptr_decorator_b->methodB() << std::endl;

  std::shared_ptr<ConcreteDecoratorB> ptr_decorator_a_b = std::make_shared<ConcreteDecoratorB>(
      ptr_decorator_a);
  std::cout << ptr_decorator_a_b->methodA() << ", " << ptr_decorator_a_b->methodB() << std::endl;

  return 0;
}
```

output:
```Shell
ConcreteComponentA methodA , ConcreteComponentA methodB 
ConcreteComponentA methodA ConcreteDecoratorA methodA , ConcreteComponentA methodB ConcreteDecoratorB methodB 
ConcreteComponentA methodA ConcreteDecoratorB methodA , ConcreteComponentA methodB ConcreteDecoratorB methodB 
ConcreteComponentA methodA ConcreteDecoratorA methodA ConcreteDecoratorB methodA , ConcreteComponentA methodB ConcreteDecoratorB methodB ConcreteDecoratorB methodB 
```

# Factory Pattern
定义一个创建对象的接口, 但由子类决定要实例化的类是哪一个. 工厂模式让类把实例化推迟到子类.
按照解决的问题类型, 工厂模式分为三种: 简单工程模式(Simple Factory), 工厂模式(Factory Method), 抽象工厂模式(Abstract Factory).

## 简单工厂模式(Simple Factory)
简单工厂模式主要是为了定义一个用于创建对象的接口, 缺点是对修改不封闭, 新增加产品需要修改工厂代码, 违反了开闭法则(OCP).

```puml

class Factory {
+ createProduct() : Product
}

class Product {
+ operation() = 0 : virtual void 
}

class ConcreteProductA {
+ operation() : void 
}

class ConcreteProductB {
+ operation() : void 
}

Product <|-- ConcreteProductA
Product <|-- ConcreteProductB

Factory ..> ConcreteProductA
Factory ..> ConcreteProductB
```

```C++
// By yongcong.wang @ 30/07/2019
#include <iostream>
#include <string>
#include <memory>

class Product {
 public:
  explicit Product() {}
  virtual ~Product() {}
  Product(const Product &rhs) = delete;
  Product &operator=(const Product &rhs) = delete;

  virtual void operation() = 0;
};

class ConcreteProductA : public Product {
 public:
  explicit ConcreteProductA() : Product() {}
  ~ConcreteProductA() {}
  ConcreteProductA(const ConcreteProductA &rhs) = delete;
  ConcreteProductA &operator=(const ConcreteProductA &rhs) = delete;

  void operation() {
    std::cout << "ConcreteProductA called!" << std::endl;
  }
};

class ConcreteProductB : public Product {
 public:
  explicit ConcreteProductB() : Product() {}
  ~ConcreteProductB() {}
  ConcreteProductB(const ConcreteProductB &rhs) = delete;
  ConcreteProductB &operator=(const ConcreteProductB &rhs) = delete;

  void operation() {
    std::cout << "ConcreteProductB called!" << std::endl;
  }
};

class Factory {
 public:
  explicit Factory() {}
  ~Factory() {}
  Factory(const Factory &rhs) = delete;
  Factory &operator=(const Factory &rhs) = delete;

  std::shared_ptr<Product> createProduct(const std::string &product) {
    if (product == "ProductA") {
      return std::make_shared<ConcreteProductA>();
    }
    if (product == "ProductB") {
      return std::make_shared<ConcreteProductB>();
    }
  }

};

int main() {
  Factory factory;

  std::shared_ptr<Product> product_a = factory.createProduct("ProductA");
  std::shared_ptr<Product> product_b = factory.createProduct("ProductB");

  product_a->operation();
  product_b->operation();

  return 0;
}
```

output:
```Shell
ConcreteProductA called!
ConcreteProductB called!
```

## 工厂方法模式(Factory Method)
简单工厂模式和工厂方法模式的区别之处在于, 工厂方法模式并不是只是为了封装对象的创建, 而是将对象的创建放到子类中实现, `Factory`类中只是提供了对象创建的接口, 实现放在了子类`ConcreteFacotory`中进行.
缺点: 每增加一个工厂对象, 都需要增加一个类, 如果工厂比较多, 工厂对象的类也会非常多.
```puml
class Product {
+ operation() = 0: virtual void
}

class ConcreteProductA {
+ operation() : void
}

class ConcreteProductB {
+ operation() : void
}

class Factory {
+ createProduct() = 0 : virtual Product 
}

class ConcreteFactoryA {
+ createProduct() : Product
}

class ConcreteFactoryB {
+ createProduct() : Product
}

Product <|-- ConcreteProductA
Product <|-- ConcreteProductB

Factory <|-- ConcreteFactoryA
Factory <|-- ConcreteFactoryB

ConcreteFactoryA ..> ConcreteProductA
ConcreteFactoryB ..> ConcreteProductB
```
```C++
// By yongcong.wang @ 30/07/2019
#include <iostream>
#include <string>
#include <memory>

class Product {
 public:
  explicit Product() {}
  virtual ~Product() {}
  Product(const Product &rhs) = delete;
  Product &operator=(const Product &rhs) = delete;

  virtual void operation() = 0;
};

class ConcreteProductA : public Product {
 public:
  explicit ConcreteProductA() : Product() {}
  ~ConcreteProductA() {}
  ConcreteProductA(const ConcreteProductA &rhs) = delete;
  ConcreteProductA &operator=(const ConcreteProductA &rhs) = delete;

  void operation() {
    std::cout << "ConcreteProductA called!" << std::endl;
  }
};

class ConcreteProductB : public Product {
 public:
  explicit ConcreteProductB() : Product() {}
  ~ConcreteProductB() {}
  ConcreteProductB(const ConcreteProductB &rhs) = delete;
  ConcreteProductB &operator=(const ConcreteProductB &rhs) = delete;

  void operation() {
    std::cout << "ConcreteProductB called!" << std::endl;
  }
};

class Factory {
 public:
  explicit Factory() {}
  virtual ~Factory() {}
  Factory(const Factory &rhs) = delete;
  Factory &operator=(const Factory &rhs) = delete;

  virtual std::shared_ptr<Product> createProduct() = 0;
};

class ConcreteFactoryA : public Factory {
 public:
  explicit ConcreteFactoryA() : Factory() {}
  ~ConcreteFactoryA() {}
  ConcreteFactoryA(const ConcreteFactoryA &rhs) = delete;
  ConcreteFactoryA &operator=(const ConcreteFactoryA &rhs) = delete;

  std::shared_ptr<Product> createProduct() {
    return std::make_shared<ConcreteProductA>();
  }
};

class ConcreteFactoryB : public Factory {
 public:
  explicit ConcreteFactoryB() : Factory() {}
  ~ConcreteFactoryB() {}
  ConcreteFactoryB(const ConcreteFactoryB &rhs) = delete;
  ConcreteFactoryB &operator=(const ConcreteFactoryB &rhs) = delete;

  std::shared_ptr<Product> createProduct() {
    return std::make_shared<ConcreteProductB>();
  }
};

int main() {

  auto factory_a = std::make_shared<ConcreteFactoryA>();
  auto factory_b = std::make_shared<ConcreteFactoryB>();

  auto product_a = factory_a->createProduct();
  auto product_b = factory_b->createProduct();

  product_a->operation();
  product_b->operation();

  return 0;
}
```

output:
```Shell
ConcreteProductA called!
ConcreteProductB called!
```


## 抽象工厂模式(Abstract Factory)
抽象工厂模式扩展了工厂子类的产品能力, 它给客户端提供了接口, 可以创建多个产品的产品对象.
```puml
class Factory {
+ createProductA() = 0 : virtual ProductA
+ createProductB() = 0 : virtual ProductB
}

class ConcreteFactoryA {
+ createProductA() : ProductA
+ createProductB() : ProductB
}

class ConcreteFactoryB {
+ createProductA() : ProductA
+ createProductB() : ProductB
}

class ProductA {
+ operation() = 0 : virtual void
}

class ConcreteProductAa {
+ operation() : void
}

class ConcreteProductAb {
+ operation() : void
}

class ProductB {
+ operation() = 0 : virtual void
}

class ConcreteProductBa {
+ operation() : void
}

class ConcreteProductBb {
+ operation() : void
}

Factory <|-- ConcreteFactoryA
Factory <|-- ConcreteFactoryB

ProductA <|-- ConcreteProductAa
ProductA <|-- ConcreteProductAb
ProductB <|-- ConcreteProductBa
ProductB <|-- ConcreteProductBb

ConcreteFactoryA ..> ConcreteProductAa
ConcreteFactoryA ..> ConcreteProductBa

ConcreteFactoryB ..> ConcreteProductAb
ConcreteFactoryB ..> ConcreteProductBb

```

# Singleton Pattern


# OO(Object Orient) Principle

## The Open-Closed Principle: 
Class should be open for extension, but closed for modification.

## 针对接口编程, 而不是针对实现编程: 
函数继承自基类, 或者继承基类接口由子类来实现, 这两种做法都是依赖于`实现`. 实现被绑定在子类上, 不利于更改. 针对接口编程将接口分离出来, 放在分开的类中, 子类不需要知道也不需要负责具体实现.

## 多用组合, 少用继承: 
使用组合(Composition)建立系统具有很大的弹性, 不仅可以将算法族封装成类, 更可以在运行时动态地改变行为, 只要组合的行为对象符合正确的接口标准即可.

## 为交互对象之间的松耦合设计而努力:
当两个对象之间松耦合, 他们依然可以交互, 但是不太清楚彼此的细节. 松耦合设计之所以能让我们建立有弹性的OO系统, 能够应对变化, 是因为对象之间的相互依赖降到最低.

## 类应该对扩展开放, 对修改关闭:
我们的目标是允许类容易扩展, 在不修改现有代码的情况下, 就可以搭配新的行为. 这样的设计具有弹性能够应对改变, 可以接受新的功能来应对改变的需求.

