---
title: Python Object-Oriented Programming (OOP)
date: 2025-03-25
order: 6
categories: [Backend, Python]
tags: [Python, OOP, Object-Oriented Programming]
author: kai
---

# 🚀 Python Object-Oriented Programming (OOP)
Python is a **fully object-oriented programming language**, allowing you to define your own classes, encapsulate data and behavior, and build scalable, modular code.  

## 📦 Basic Class Structure

```python
class MyClass:
    """This is a class-level docstring."""

    # Class attribute (shared by all instances, similar to static in Jave)
    name = "Kai"

    # Private class attribute (cannot be accessed directly outside)
    __weight = 0

    def __init__(self, arg1, arg2):  
        """Constructor: called when a new object is created"""
        # attr1, and attr2 are instance-specific attributes, need not defined outside the Constructor
        self.attr1 = arg1
        self.attr2 = arg2

    def instance_method(self):
        """Instance method: operates on instance-level data"""
        return self.attr1

    @classmethod
    def class_method(cls):
        """Class method: operates on class-level data"""
        return cls.name

    @staticmethod
    def static_method():
        """Static method: no self, no cls, doesn't access instance or class data"""
        return "This is a static utility"
```

## 🔑 Key Components
**1. __init__ Constructor**
- `obj = MyClass("value1", "value2")`
- Can accept parameters for initialization. 
- Called **automatically when a new object is instantiated**.
- If you don’t define an __init__ method, Python provides a default no-argument constructor.


**2. self: Reference to the Instance of the class**
- Required in all instance methods
- Allows access to instance attributes

**3. Class vs. Instance Attributes**
- Class attribute: Shared across all instances. Access via `ClassName.attr` or `self.attr`.
- Instance attribute: Unique to each instance. Access via `self.attr` only.

| Feature | Class Attribute | Instance Attribute |
|---------|-----------------|--------------------|
| Storage | Defined inside the **class** | Defined inside `__init__()` using self |
| Shared or Not | Shared by **all instances** | Each instance has its **own copy** |
| Effect of Change | Affects **all instances** (unless overridden) | Affects **only the modified instance** |
| Memory Use | **One copy** shared across class | **Separate copy** per instance | 
| Typical Usage | Constants, counters, global config | Per-object custom data | 
| Access via | `MyClass.name` or `obj.name` | `obj.attr1` |
| Best For | Constants, Configuration, Counters | Custom per-object behavior |

### Example: Access and Modify Attributes 
```python
class Dog:
    # Class attribute (shared by all instances)
    species = "Canis familiaris"

    def __init__(self, name, age):
        # Instance attributes (unique to each object)
        self.name = name

dog1 = Dog("Buddy")
dog2 = Dog("Max")

# 1. Access Instance Attributes
print(dog1.name)  # Buddy
print(dog2.name)  # Max

# 2. Shared Class Attribute
print(dog1.species)  # Canis familiaris
print(dog2.species)  # Canis familiaris

# 3. Modify Class Attribute - using Classname.att to modify the value of class-level attribute ✅
Dog.species = "Canis lupus"
print(dog1.species)  # Canis lupus
print(dog2.species)  # Canis lupus

# 4. Shadowing with Instance Attribute - using instance.att to update the value of class-level attribute ‼️
dog1.species = "Custom Species" # Actually create an instance attriute! The value of the class-level attribute will not change.

print(dog1.species)  # Custom Species (instance-level)
print(dog2.species)  # Canis lupus (still class-level)
print(Dog.species)   # Canis lupus (class-level, not change)

# 5. Modify instance attribute
dog1.name = "Coco"
print(dog1.name)  # Coco
print(dog2.name)  # Max, will not change

# 6. Add instance attribute dynamically
dog1.age = 3
print(dog1.age)  # 3
print(dog2.age)  # AttributeError

```

## 🪡 complete class example
a **complete class example** using a `Student` class, including:
- **Class attributes** -  Shared by all instances. Can be changed using a @classmethod.
- **Instance attributes** - Defined in __init__(). Unique for each instance/student.
- **Private attributes & methods** -  Cannot be accessed directly outside the class. Only accessible from within the class
. Convention-based privacy using __ (name mangling).
- **Class methods** - Operates at the class level. Can modify class attributes.
- **Static methods** - No access to self or cls. Use it for utility functions related to the class

```python
class Student:
    school = "University of Pennsylvania"  # Class attribute

    def __init__(self, name, age):
        self.name = name          # Instance attribute
        self.age = age
        self.__secret = "Failed"  # Private instance attribute. 

    def study(self, subject):
        print(f"{self.name} is studying {subject}.")
        self.__update_record()   # Call private method internally

    def __update_record(self):   # Private instance method.
        print("Update the class record!")

    @classmethod 
    def change_school(cls, new_school):
        cls.school = new_school
        print(f"Update school to {new_school}.")

    @staticmethod
    def get_school_year():
        return "Year of 2023-2024"

stu1 = Student("Kai", 20)

stu1.study("Python")
# Kai is studying Python.
# Update the class record!

Student.change_school("University of California, Los Angeles")
# Update school to University of California, Los Angeles.

print(Student.get_school_year())
# Year of 2023-2024
```

## 🧶 Types of Methods

| Method Type | Decorator | First Param | Access Scope | Use Case |
|-------------|-----------|-------------|--------------|----------|
| Instance Method | (none) | self | Instance variables / methods | Access Object's data |
| Class Method | @classmethod | cls | Class variables / methods | Factory methods, class-level operations |
| Static Method | @staticmethod | (none) | General utilities | Independent utilities that don’t touch object or class state |

### Example: Class Method
Two practical and elegant use cases of class methods:

- ✅ **Factory methods** for alternative constructors  
    - Converts external input (like strings, files, etc.) into structured class instances
    - `cls() `will invoke `__init__()` / `constructor` to create object 
    - `d1 = Date.from_string('2023-09-15')` will return an object. 
- ✅ **Utility class logic** that needs access to class context
    - Returns a new instance without needing external values
    - Keeps class logic centralized
	- Can be reused or overridden in child classes

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_str):
        """Factory method: creates Date from 'YYYY-MM-DD' string"""
        year, month, day = map(int, date_str.split('-'))
        return cls(year, month, day) # invoke constructor 

    @classmethod
    def get_current_date(cls):
        """Utility method: returns current date as Date instance"""
        import datetime
        now = datetime.datetime.now()
        return cls(now.year, now.month, now.day) # invoke constructor 

d1 = Date.from_string('2023-09-15')
d2 = Date.get_current_date()
print(f"{d2.year}-{d2.month}-{d2.day}")  # e.g., 2025-03-25
```

### Example: Static Method
defining **utility functions** that are **logically grouped** within a class but don’t need access to instance (`self`) or class (`cls`) context.
- Does not receive self or cls as a parameter
- Cannot access instance or class attributes
- Belongs to the class logically, but not structurally


```python
class MathUtils:

    @staticmethod
    def is_prime(num):
        """checking if a number is a prime（静utility functions）"""
        if num < 2:
            return False
        for i in range(2, int(num**0.5) + 1):
            if num % i == 0:
                return False
        return True

print(MathUtils.is_prime(17))  # Output: True
```

### Example: Using `cls` to Instantiate an Object
In Python, `self` refers to an **instance of a class**, whereas `cls` refers to the **class itself**.  
We typically encounter `cls` inside methods decorated with `@classmethod`.

```python
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
        print('self:', self)

    @classmethod
    def build(cls):
        # cls is equivalent to Person here
        p = cls("Kai", 18)  # cls("Kai", 18) = Person("Kai", 18) -> invoke __init__("Kai", 18)
        print('cls:', cls) 
        return p

person = Person.build() 
print(person, person.name, person.age)  

# Output:
# self: <__main__.Person object at 0x102da9a90> -> person = Person.build() invoke __init__, then print
# cls: <class '__main__.Person'> -> print('cls:', cls)
# <__main__.Person object at 0x102da9a90> Kai 18 -> print(person, person.name, person.age)
```

### Example: The StudentManager Class
build a **mini student management system** in Python that covers key object-oriented principles and practical features:

- ✅ Class and instance attributes  
- ✅ Static and class methods  
- ✅ Input validation  
- ✅ `__str__` and `__repr__` magic methods for clean display  


```python
class StudentManager:
    __students = []  # Private class attribute to hold all student records

    def __init__(self, name):
        self.manager_name = name

    def add_student(self, student): 
        """Adds a Student object to the list"""
        self.__students.append(student)
        print(f"{student} is added!")

    @classmethod
    def get_student_count(cls):
        """Class method to return total count"""
        return len(cls.__students)

    @staticmethod
    def validate_age(age):
        """Static method to enforce age limits"""
        return 15 <= age <= 60

    def __str__(self):
        """Readable object display for the manager"""
        return f"Manager {self.manager_name}，manage the number of students：{len(self.__students)}."


class Student:
    def __init__(self, name, age):
        """Validates age via StudentManager.validate_age() (decoupled logic)"""
        if not StudentManager.validate_age(age):
            raise ValueError("Invalid age")
        self.name = name
        self.age = age

    def __repr__(self):
        """Uses __repr__() for developer-friendly display (e.g., in logs and lists)"""
        return f"<Student {self.name}>"


mgr = StudentManager("Teacher Wang")

try:
    s1 = Student("Kai", 18)
    mgr.add_student(s1)  # Output: <Student Kai> is added!

    s2 = Student("Jo", 12)  # Raises ValueError due to invalid age
except ValueError as e:
    print(e) # Output: Invalid age 


print(mgr) # Output: Manager Teacher Wang，manage the number of students：1.
print(StudentManager.get_student_count()) # Output: 1
```

## 🎩 Magic Methods
In Python, **magic methods** (also known as **dunder methods**, short for *double underscore*) are special functions that give your objects powerful, intuitive behaviors — like printing nicely, comparing, adding, iterating, and more.

They’re the backbone of Python's object model and make your custom classes behave like built-ins.

#### What Are Magic Methods?

- They are **predefined special methods** surrounded by double underscores (e.g., `__init__`, `__str__`, `__len__`)
- Python internally calls them in response to **language syntax or built-in function usage**
- You can override them to **customize your class behavior**

#### Most Common Magic Methods

| Method         | Triggered By                     | Purpose                                |
|----------------|----------------------------------|----------------------------------------|
| `__init__`     | `obj = Class()`                  | Constructor (initializes instance)     |
| `__str__`      | `print(obj)` or `str(obj)`       | Human-readable display                 |
| `__repr__`     | `repr(obj)`, shell/console       | Developer/debug representation         |
| `__len__`      | `len(obj)`                       | Return custom length                   |
| `__eq__`       | `obj1 == obj2`                   | Equality check                         |
| `__lt__`       | `obj1 < obj2`                    | Less-than comparison                   |
| `__add__`      | `obj1 + obj2`                    | Custom addition logic                  |
| `__getitem__`  | `obj[key]`                       | Indexing access                        |
| `__setitem__`  | `obj[key] = value`               | Indexing assignment                    |
| `__iter__`     | `for item in obj:`               | Make object iterable                   |

#### Example  `__str__` and `__repr__`

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __str__(self):
        return f"《{self.title}》 by {self.author}"

    def __repr__(self):
        return f"Book(title='{self.title}', author='{self.author}')"

b = Book("1984", "George Orwell")
print(str(b))     # 《1984》 by George Orwell
print(b)          # 《1984》 by George Orwell
print(repr(b))    # Book(title='1984', author='George Orwell')
```


<br>



---

🚀 Stay tuned for more insights into Python!