---
title: Creational Patterns
date: 2018-11-7 17:15:25
tags: 设计模式
categories: 设计模式
---

## Creational Patterns

A collection of design patterns and idioms in Python.
<!-- more -->

### abstract factory （抽象工厂）

在 Java 和其他语言中，抽象工厂模式用于提供接口去创建相关对象，而不需要显式指定它们的类。
在 Python 中，在正常情况下我们可以简单地使用类本身。如果有多种不同类型对象需要创建，使用抽象工厂模式。以实现一个宠物店的例子说明，在一个抽象工厂类里实现多个关联对象的创建。


```python
import random

class PetShop(object):
    """A pet shop"""
    
    def __init__(self, animal_factory=None):
        """pet_factory is our abstract factory. We can set it at will."""
        
        self.pet_factory = animal_factory
        
    def show_pet(self):
        """Creates and shows a pet using the abstract factory"""
        
        pet = self.pet_factory()
        print("We have a lovely {}".format(pet))
        print("It says {}".format(pet.speak()))

class Dog(object):
    def speak(self):
        return "woof"
    
    def __str__(self):
        return "Dog"
        
class Cat(object):
    def speak(self):
        return "meow"
    
    def __str__(self):
        return "Cat"
        
#  Additional factories
def random_animal():
    """Let's be dynamic!"""
    return random.choice([Dog, Cat])()

# Show pets with various factories
if __name__ == "__main__":

    # A Shop that sells only cats
    cat_shop = PetShop(Cat)
    cat_shop.show_pet()
    print("")
    
    # A Shop that sells random animals
    shop = PetShop(random_animal)
    for i in range(3):
        shop.show_pet()
        print("=" * 20)

### OUTPUT ###
# We have a lovely Cat
# It says meow
#
# We have a lovely Dog
# It says woof
# ====================
# We have a lovely Cat
# It says meow
# ====================
# We have a lovely Cat
# It says meow
# ====================
```

## borg (实例中具有共享状态的单例)

Borg 模式（也称为 Monostate 模式）是一种方式实现单例行为，但不是只有一个实例。在一个类中，有多个实例共享相同的状态。在 Python 中，实例属性存储在一个属性字典名为 \_\_dict\_\_。通常，每个实例都会有它自己的字典，但 Borg 模式修改了这一切实例具有相同的字典。在此示例中，__shared_state 属性将是字典在所有实例之间共享，这通过指定来确保初始化新的 \_\_shared_state 到 \_\_dict\_\_ 变量实例（即 \_\_init\_\_ 方法）

```python
class Bord(object):
    __shared_state = {}
    
    def __init__(self):
        self.__dict__ = self.__shared_state
        self.state = 'init'
    
    def __str__(self):
        return self.state

class YourBord(Borg):
    pass
    
if __name__ == '__main__':
    rm1 = Borg()
    rm2 = Borg()
    
    rm1.state = 'Idle'
    rm2.state = 'Running'
    
    print('rm1: {0}'.format(rm1))
    print('rm2: {0}'.format(rm2))

    rm2.state = 'Zombie'
    
    print('rm1: {0}'.format(rm1))
    print('rm2: {0}'.format(rm2))
    
    print('rm1 id: {0}'.format(id(rm1)))
    print('rm2 id: {0}'.format(id(rm2)))

    rm3 = YourBorg()

    print('rm1: {0}'.format(rm1))
    print('rm2: {0}'.format(rm2))
    print('rm3: {0}'.format(rm3))
    
### OUTPUT ###

# rm1: Running
# rm2: Running
# rm1: Zombie
# rm2: Zombie
# rm1 id: 140732837899224
# rm2 id: 140732837899296
# rm1: Init
# rm2: Init
# rm3: Init
```

## builder 建造者模式

当一个对象必须经过多个步骤来创建，并且要求不同的参数可以产生不同的表现的时候， 我们可以使用建造者模式。

```python
# Abstract Building
class Building(object):
    def __init__(self):
        self.build_floor()
        self.build_size()
    
    def build_floor(self):
        raise NotImplementedError
    
    def build_size(self):
        raise NotImplementedError
        
    def __repe__(self):
        return 'Floor: {0.floor} | Size: {0.size}'.format(self)
        
# Concrete Buildings
class House(Building):
    def build_floor(self):
        self.floor = 'One'
    
    def build_size(self):
        self.size = 'Big'
        
class Flat(Building):
    def build_floor(self):
        self.floor = 'More than one'
    
    def build_size(self):
        self.size = 'Small'
     
 class ComplexHouse(ComplexBuilding):
     def build_floor(self):
         self.floor = 'One'
         
     def build_size(self):
         self.size = 'Big and fancy'
         
def construct_building(cls):
    building = cls()
    building.build_floor()
    building.build_size()
    return building # 返回一个实例
   
# Client
if __name__ == "__main__":
    house = House()
    print(house)
    flat = Flat()
    print(flat)
    
    # Using a external constructor function:     
    complex_house = constructor_building(ComplexHouse)
    print(complex_house)
    
### OUTPUT ###
# Floor: One | Size: Big
# Floor: More than One | Size: Small
# Floor: One | Size: Big and fancy
```

## factory method 工厂方法

工厂方法模式能够封装具体类型的实例化，抽象的 get\_localizer 提供了一个创建对象的工厂方法。

```python
class ChineseGetter(object):
    """A simple localizer a la gettext"""
    
    def __init__(self):
        self.trans = dict(dog="狗", cat="猫")
        
    def get(self, msgid):
        """如果没有对应翻译，就提供默认值"""
        return self.trans.get(msgid, str(msgid))
        
class EnglishGetter(object):
    """Simply echoes the msg ids"""
   
    def get(self, msgid):
        return str(msgid)
        
def get_localizer(language="English"):
    """The factory method"""
    languages = dict(English=EnglishGetter, Chinese=ChineseGetter)
    return languages[language]()
    
if __name__ == '__main__':
    # Create our localizers
    e, g = get_localizer(language="English"), get_localizer(language="Chinese")
    # Localize some text
    for msgid in "dog parrot cat bear".split():
        print(e.get(msgid), g.get(msgid))
        
### OUTPUT ###
# dog 狗
# parrot parrot
# cat 猫
# bear bear
```

