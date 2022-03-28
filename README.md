## Python Interview Collection Basics 2021

#### Topic 001: How to implement singleton mode in Python.

> **Comment**: The singleton mode means that a class can only create a unique instance. This topic appears very frequently in interviews, because it examines not only the singleton mode, but also the Python language To what extent you have mastered it, I suggest you use decorators and metaclasses to implement the singleton mode, because these two methods are the most versatile, and you can also show your knowledge of the two of decorators and metaclasses by the way. Understanding of key knowledge points.

Method 1: Use decorators to implement singleton mode.

```Python
from functools import wraps


def singleton(cls):
    """Singleton Decorator"""
    instances = {}

    @wraps(cls)
    def wrapper(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return wrapper


@singleton
class President:
    pass
```

> **Extensions**: Decorator is a very distinctive grammar in Python. Use a function to decorate another function or class, adding extra capabilities to it. The functions that are usually implemented through decoration are crosscutting functions, that is, functions that are not necessarily related to normal business logic and can be dynamically added or removed. Decorator can provide services such as caching, proxy, context environment and so on for the code. It is a practice of the proxy mode in the design pattern. When writing decorators, functions with decoration functions (the `wrapper` function in the code above) are usually decorated with `wraps` in the `functools` module. The most important role of this decorator is to be decorated A class or function dynamically adds a `__wrapped__` attribute, which will retain the class or function before being decorated, so that when we don’t need the decoration function, we can use it to cancel the decorator. For example, we can use `President = President .__wrapped__` to cancel the singleton handling of the `President` class. What needs to be reminded is that the above singleton is not thread-safe. If you want to be thread-safe, you need to lock the code that creates the object. In Python, the `RLock` object of the `threading` module can be used to provide locks, and the `acquire` and `release` methods of the lock object can be used to implement locking and unlocking operations. Of course, a more convenient way is to use the `with` context syntax of the lock object to perform implicit lock and unlock operations.

Method 2: Use metaclasses to implement singleton mode.

```Python
class SingletonMeta(type):
    """Custom Singleton Metaclass"""

    def __init__(cls, *args, **kwargs):
        cls.__instance = None
        super().__init__(*args, **kwargs)

    def __call__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = super().__call__(*args, **kwargs)
        return cls.__instance


class President(metaclass=SingletonMeta):
    pass
```

> **Extensions**: Python is an object-oriented programming language. In an object-oriented world, everything is an object. Objects are created through classes, and classes themselves are objects, and objects like classes are created through metaclasses. When we define a class, if no parent is specified for a class, the default parent is `object`, and if no metaclass is specified for a class, then the default metaclass is `type`. Through a custom metaclass, we can change the default behavior of a class, just like in the above code, we changed the constructor of the `President` class through the `__call__` magic method of the metaclass.

> **Supplement**: Regarding the singleton mode, you may be asked about its application scenarios in the interview. Usually the state of an object is shared by other objects, you can design it as a singleton. For example, the database connection pool objects and configuration objects used in the project are usually singletons, so as to ensure that the database connection and data obtained everywhere The configuration information is completely consistent; and because the object has only a unique instance, the time and space overhead caused by repeatedly creating the object is fundamentally avoided, and the multiple occupation of resources is also avoided. For another example, the log operation in the project usually uses the singleton mode. This is because the shared log file is always open and there can only be one instance to operate it, otherwise it will cause confusion when writing the log.

#### Subject 002: Without using intermediate variables, swap the values ​​of the two variables `a` and `b`.

>**Remarks**: A typical problem of giving people a head, usually an intermediate variable is needed to exchange two variables. If the intermediate variable is not allowed, the exclusive OR operation can be used in other programming languages ​​to exchange the two variables. The value of, but there is a simpler and clearer way in Python.

method one:

```Python
a = a ^ b
b = a ^ b
a = a ^ b
```

Method Two:

```Python
a, b = b, a
```

> **Extension**: It should be noted that this approach of `a, b = b, a` is not actually tuple unpacking, although many people think so. There are `ROT_TWO` instructions in Python bytecode instructions to support this operation, and similarly `ROT_THREE`, for more than 3 elements, such as `a, b, c, d = b, c, d, a`, It will be used to create tuples and unpack tuples. If you want to know the bytecode instructions corresponding to your code, you can use the `dis` function of the `dis` module in the Python standard library to disassemble your Python code.

#### Question 003: Write a function to delete duplicate elements in the list, requiring the relative position of the elements to remain unchanged after deduplication.

> **Remarks**: This question often appears in the junior/intermediate Python job interviews. The question originated from the 10th question in the first chapter of the book "Python Cookbook". Many interview questions are actually on this book. So I suggest you have time to study this book.

```Python
def dedup(items):
    no_dup_items = []
    seen = set()
    for item in items:
        if item not in seen:
            no_dup_items.append(item)
            seen.add(item)
    return no_dup_items
```

If you want, you can also transform the above function into a generator, the code is shown below.

```Python
def dedup(items):
    seen = set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)
```

> **Extension**: Since the bottom layer of the collection in Python uses hash storage, the `in` and `not in` member operations of the collection are far better than the list in performance, so the above code we use the collection to save Elements that have already appeared. The elements in the collection must be `hashable` objects, so the above code will be invalid when the list element is not a `hashable` object. To solve this problem, you can add a parameter to the function, which can be designed to return a hash code or a hashable `Function of the object.

#### Subject 004: Assuming that you are using the official CPython, tell the results of the following code.

> **Remarks**: The following program has no meaning for actual development, but it is a big pit in CPython. This question aims to examine how well the interviewer knows about the official Python interpreter.

```Python
a, b, c, d = 1, 1, 1000, 1000
print(a is b, c is d)

def foo():
    e = 1000
    f = 1000
    print(e is f, e is d)
    g = 1
    print(g is a)

foo()
```

operation result:

```
True False
True False
True
```

The result of `a is b` in the above code is `True` but the result of `c is d` is `False`, which is really puzzling. For performance optimization considerations, the CPython interpreter caches frequently used integer objects in an object pool called `small_ints`. The integer value of the `small_ints` cache is set to the range of `[-5, 256]`, that is to say, in any place where these integers are referenced, there is no need to recreate the `int` object, but directly reference the buffer pool In the object. If the integer is not in this range, then even if the values ​​of the two integers are the same, they are different objects.

In order to further improve performance, the bottom layer of CPython has made another setting. For integers whose value is not in the cache range of `small_ints` in the same code block, if there is already an integer object with the same value in the same code block, then directly Refer to the object, otherwise create a new `int` object. It should be noted that this rule applies to numeric types, but for strings, the length of the string needs to be considered. You can prove this by yourself.

> **Extensions**: If you use PyPy (another Python interpreter implementation, support JIT, improve the shortcomings of CPython, it is better than CPython in performance, but the support for third-party libraries is slightly worse) to run the above The code, you will find that all the outputs are True.

#### Topic 005: What is a Lambda function, and its application scenarios are illustrated by examples.

> **Comment**: This topic mainly wants to examine the application scenarios of Lambda functions. The subtext is to ask whether you have used Lambda functions in your project, and in what scenarios will you use Lambda functions to judge you Ability to write code. Because Lambda functions are usually used in higher-order functions, the main function is to finally realize the decoupling of the code by passing the function to the function or letting the function return to the function.

Lambda function is also called anonymous function. It is a small function that can be implemented with a single line of code. The Lambda function in Python can only write one expression, and the execution result of this expression is the return value of the function, without writing the `return` keyword. Because Lambda functions don't have names, they won't have naming conflicts with other functions.

> **Extension**: During the interview, you may be asked to use Lambda functions to implement some functions, that is, use one line of code to implement the function required by the topic, for example: use one line of code to implement the factorial function, and use one line of code to implement Find the function of the greatest common divisor, etc.
>
> ```Python
> fac = lambda x: __import__('functools').reduce(int.__mul__, range(1, x + 1), 1)
> gcd = lambda x, y: y% x and gcd(y% x, x) or x
> ```

In fact, the main purpose of Lambda function is to pass a function into another higher-order function (such as Python built-in `filter`, `map`, etc.) to decouple the function and enhance the flexibility and versatility of the function. The following example uses the `filter` and `map` functions to filter out odd numbers from the list and square them to form a new list. Because higher-order functions are used, the rules for filtering and mapping data are the callers of the function. It is passed in through another function, so the `filter` and `map` functions are not coupled with specific filtering and mapping data rules.

```Python
items = [12, 5, 7, 10, 8, 19]
items = list(map(lambda x: x ** 2, filter(lambda x: x% 2, items)))
print(items) # [25, 49, 361]
```

> **Extension**: It will be simpler and clearer to implement the above code with the generation of the list, the code is as follows.
>
> ```Python
> items = [12, 5, 7, 10, 8, 19]
> items = [x ** 2 for x in items if x% 2]
> print(items) # [25, 49, 361]
> ```

#### Subject 006: Talk about shallow copy and deep copy in Python.

> **Comment**: The frequency of the topic itself is very high, but there is no technical content in terms of the topic. For this kind of interview question, when answering, you must make your answer exceed the interviewer’s expectations, so that you can get a better impression. So the point to answer this question is not only to be able to tell the difference between shallow copy and deep copy, the two major problems that may be encountered during deep copy, but also to tell the Python standard library's support for shallow copy and deep copy, and then you can Talk about how lists and dictionaries implement copy operations and how to implement deep copying through serialization and deserialization. Finally, you can also mention the prototype pattern in the design pattern and its application in the project.

Shallow copy usually only copies the object itself, while deep copy not only copies the object, but also recursively copy the objects associated with the object. Deep copy may encounter two problems: one is that if an object directly or indirectly references itself, it will cause endless recursive copies; the other is that deep copy may also copy data originally designed to be shared by multiple objects. Python implements shallow copy and deep copy operations through the `copy` and `deepcopy` functions in the `copy` module. `deepcopy` can save the copied objects through the `memo` dictionary, thereby avoiding the self Reference recursion; in addition, you can customize the copy behavior of a specified type of object through the `pickle` function of the `copyreg` module.

The essence of the `deepcopy` function is actually one-time serialization and one-time return serialization of the object. In the interview questions, we also tested the deep copy operation of the object with a custom function. Obviously we can use the `dumps` and `of the `pickle` module. loads` to do it, the code is as follows.

```Python
import pickle

my_deep_copy = lambda obj: pickle.loads(pickle.dumps(obj))
```

The slicing operation `[:]` of the list is equivalent to the shallow copy of the list object, and the `copy` method of the dictionary can realize the shallow copy of the dictionary object. Object copy is actually a faster way to create objects. In Python, the creation of an object through the constructor is a two-stage construction, the first is the allocation of memory space, and then the initialization. When creating an object, we can also create a new object based on the "prototype" object. The creation and initialization of the object are completed by copying the prototype object (copy memory). This approach is more efficient. This is the design pattern. Prototype mode. In Python, we can implement the prototype mode through metaclasses. The code is shown below.

```Python
import copy


class PrototypeMeta(type):
    """The metaclass of the prototype pattern"""

    def __init__(cls, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Bind the clone method to the object to realize the object copy
        cls.clone = lambda self, is_deep=True: \
            copy.deepcopy(self) if is_deep else copy.copy(self)


class Person(metaclass=PrototypeMeta):
    pass


p1 = Person()
p2 = p1.clone() # deep copy
p3 = p1.clone(is_deep=False) # shallow copy
```

#### Question 007: How does Python implement memory management?

> **Comment**: When the interviewer asks this question, an opportunity to show oneself is in front of you. You have to ask the interviewer first: "Are you talking about the official CPython interpreter?". This rhetorical question can show that you know the different implementation versions of the Python interpreter, and you also know that the interviewer wants to ask about CPython. Of course, many interviewers have no idea about the differences between the underlying implementations of different Python interpreters. Therefore, **don't think that the interviewer must be better than you**. With this confidence, you can complete the interview better.

Python provides automatic memory management, that is to say, the allocation and release of memory space are automatically performed by the Python interpreter at runtime. The automatic memory management function greatly reduces the programmer's workload and can also help the programmer in To some extent solve the problem of memory leaks. Taking the CPython interpreter as an example, its memory management has three key points: reference counting, tag cleaning, and generational collection.

**Reference Counting**: For the CPython interpreter, every object in Python is actually a `PyObject` structure, which has a reference counter member variable named `ob_refcnt` inside. When the program is running, the value of `ob_refcnt` will be updated to reflect how many variables refer to the object. When the object's reference count value is 0, its memory will be released.

```C
typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
```

The following conditions will cause the reference count to increase by 1:

-Object is created
-Object is referenced
-The object is passed into a function as a parameter
-The object is stored as an element in a container

The following conditions will cause the reference count to decrement by `1`:

-Use `del` statement to display deleted object references
-Object references are reassigned to other objects
-An object leaves its scope
-The container holding the object itself is destroyed
-The container holding the object deletes the object

The reference count of the object can be obtained through the `getrefcount` function of the `sys` module. The memory management method of reference counting will be fatal when it encounters circular references, so other garbage collection algorithms are needed to supplement it.

**Mark cleanup**: CPython uses the "Mark and Sweep" algorithm to solve the circular reference problem that may arise from container types. The algorithm is divided into two phases during garbage collection: marking phase, traversing all objects, if the object is reachable (referenced by other objects), then mark the object as reachable; cleaning phase, traversing the object again, if found If an object is not marked as reachable, it will be recycled. The bottom layer of CPython maintains two double-ended linked lists. One linked list stores the container objects that need to be scanned (let's call it linked list A), and the other linked list stores temporary unreachable objects (let's call it linked list B). In order to implement the "mark-clean" algorithm, in addition to the `ref_count` variable that records the current reference count, each node in the linked list also has a `gc_ref` variable. This `gc_ref` is a copy of `ref_count`, so the initial The value is the size of `ref_count`. When performing garbage collection, it first traverses the nodes in linked list A, and subtracts `1` from the `gc_ref` of all objects referenced by the current object. The main function of this step is to remove the influence of circular references on the reference count. Traverse the nodes in linked list A again, if the node's `gc_ref` value is `0`, then this object is marked as "temporarily unreachable" (`GC_TENTATIVELY_UNREACHABLE`) and moved to linked list B; if the node's `gc_ref` `If ​​it is not `0`, then the object will be marked as "reachable" (`GC_REACHABLE`). For the "reachable" object, the node that can be reached by the node is also recursively marked as "reachable"; The nodes marked as "reachable" in linked list B must be put back into linked list A. After traversing twice, the nodes in the linked list B are the nodes that need to release the memory.

**Generational Collection**: During the recycling of circularly referenced objects, the entire application will be suspended. In order to reduce the pause time of the application, Python improves the efficiency of garbage collection through generational collection (space for time). The basic idea of ​​generational collection is: the longer an object exists, the less likely it is to be garbage, and such objects should not be garbage collected as much as possible. CPython divides objects into three generations and denoted them as `0`, `1`, and `2`. Each new object is in the `0` generation. If the object survives a round of garbage collection scans, then It will be moved to the `1` generation. Objects that exist in the `1` generation will be less scanned by garbage collection; if the garbage collection scan is performed on the `1` generation, this object will survive again , Then it will be moved to the second generation, where it will be scanned fewer times by garbage collection. The threshold of the generational collection scan can be obtained by the `get_threshold` function of the `gc` module. This function returns a triplet indicating how many memory allocation operations will be performed after the generation of the `0` garbage collection, and how many times` After the 0` generation garbage collection, the `1` generation garbage collection will be executed, and how many times the `1` generation garbage collection will be executed after the `2` generation garbage collection. It should be noted that if a `2` generation of garbage collection is performed once, then the younger generation will perform garbage collection. If you want to modify these thresholds, you can do it through the `set_threshold` function of the `gc` module.

#### Question 008: Tell me about your understanding of iterators and generators in Python.

> **Remarks**: Many interviewees will write iterators and generators, but they cannot explain exactly what iterators and generators are. If you have the same confusion, you can refer to the answer below.

Iterators are objects that implement the iterator protocol. Unlike other programming languages, there are no keywords used to define protocols or express conventions in Python. Words such as `interface` and `protocol` are not in the keyword list of the Python language. The Python language expresses the agreement through magic methods, which is what we call the agreement, and the two magic methods `__next__` and `__iter__` represent the iterator protocol. You can use the `for-in` loop to fetch the value from the iterator object, or you can use the `next` function to fetch the next value in the iterator object. Generator is a syntactic upgrade version of iterator, and it can be implemented with simpler code.

> **Extension**: In interviews, I often ask to write iterators that generate Fibonacci numbers. You can refer to the following code.
>
> ```Python
> class Fib(object):
>
> def __init__(self, num):
> self.num = num
> self.a, self.b = 0, 1
> self.idx = 0
>
> def __iter__(self):
> return self
>
> def __next__(self):
> if self.idx <self.num:
> self.a, self.b = self.b, self.a + self.b
> self.idx += 1
> return self.a
> raise StopIteration()
> ```
>
> If you use the generator syntax to rewrite the above code, the code will be much simpler and more elegant.
>
> ```Python
> def fib(num):
> a, b = 0, 1
> for _ in range(num):
> a, b = b, a + b
> yield a
> ```

#### Topic 009: What is the difference between the match method and the search method of regular expressions?

> **Comments**: Regular expressions are an important tool for string processing, so they are also knowledge points frequently examined in interviews. In Python, there are two ways to use regular expressions, one is to directly call the function in the `re` module, passing in the regular expression and the string to be processed; the other is to pass the `compile of the `re` module first `The function creates a regular expression object, and then calls the method through the object and passes in the string to be processed. **If a regular expression is frequently used, we recommend using the `re.compile` function to create a regular expression object, which will reduce the overhead caused by frequently compiling the same regular expression**.

 The `match` method is to perform regular expression matching from the beginning of the string and return a `Match` object or None. The `search` method scans the entire string to find a matching pattern, and also returns a Match object or None.

#### Question 010: What is the execution result of the following code?

```Python
def multiply():
    return [lambda x: i * x for i in range(4)]

print([m(100) for m in multiply()])
```

operation result:

```
[300, 300, 300, 300]
```

The running result of the above code can easily be misjudged as `[0, 100, 200, 300]`. The first thing to note is that the `multiply` function uses generative syntax to return a list. The list stores 4 Lambda functions. These 4 Lambda functions will return the result of multiplying the passed parameter by the `i`. It should be noted that there is a closure phenomenon. The life cycle of the local variable `i` in the `multiply` function is extended. Since the final value of `i` is `3`, it passes `m(100) `When the Lambda function in the list is called, it will return `300`, and this is the case for 4 calls.

If you want to get the result of `[0, 100, 200, 300]`, you can modify the `multiply` function in the following ways.

Method 1: Use a generator to let the function get the current value of `i`.

```Python
def multiply():
    return (lambda x: i * x for i in range(4))

print([m(100) for m in multiply()])
```

or

```Python
def multiply():
    for i in range(4):
        yield lambda x: x * i

print([m(100) for m in multiply()])
```

Method 2: Use partial functions to avoid closures completely.

```Python
from functools import partial
from operator import __mul__

def multiply():
    return [partial(__mul__, i) for i in range(4)]

print([m(100) for m in multiply()])
```

#### Subject 011: Why is there no function overloading in Python?

> **Review**: C++, Java, C# and many other programming languages ​​support function overloading. The so-called function overloading refers to multiple functions with the same name in the same scope, and they have different parameter lists (parameters). The number is different or the parameter type is different or both are different), can be distinguished from each other. Overloading is also a kind of polymorphism, because the number and type of parameters are usually used to determine which overloaded function to call at compile time, so it is also called compile-time polymorphism or pre-binding. The subtext of this question is actually to ask the interviewer whether he has experience in other programming languages, whether he understands that Python is a dynamically typed language, and whether he knows the concepts of variable parameters and keyword parameters of functions in Python.

First of all, Python is an interpreted language, and function overloading usually occurs in compiled languages. Secondly, Python is a dynamically typed language. There are no type constraints on the parameters of a function, so it is impossible to distinguish overloads based on parameter types. In addition, the parameters of functions in Python can have default values, and can use variable and keyword parameters. Therefore, even if there is no function overloading, a function can behave differently according to the parameters passed in by the caller.

#### Topic 012: Use Python code to implement the Python built-in function max.

> **Remarks**: This topic may seem simple, but in fact it is a comparative study of the interviewer's skills. Because Python's built-in `max` function can not only pass in an iterable object to find the largest, but also pass in two or more parameters to find the largest; the most important thing is that you can also specify one by naming the keyword parameter `key` The function used for element comparison can also use the `default` named keyword parameter to specify the default value returned when the iterable is empty.

The following code is for reference only:

```Python
def my_max(*args, key=None, default=None):
    """
    Get the largest element in the iterable object or the largest element in two or more actual parameters
    :param args: an iterable object or multiple elements
    :param key: The function to extract the feature value for element comparison, the default is None
    :param default: If the iterable object is empty, the default value is returned, if no default value is given, a ValueError exception is raised
    :return: returns the largest element of the iterable object or multiple elements
    """
    if len(args) == 1 and len(args[0]) == 0:
        if default:
            return default
        else:
            raise ValueError('max() arg is an empty sequence')
    items = args[0] if len(args) == 1 else args
    max_elem, max_value = items[0], items[0]
    if key:
        max_value = key(max_value)
    for item in items:
        value = item
        if key:
            value = key(item)
        if value> max_value:
            max_elem, max_value = item, value
    return max_elem
```

#### Topic 013: Write a function to count the number of occurrences of each number in the incoming list and return the corresponding dictionary.

> **Comments**: The subject of giving away the head, no explanation.

```Python
def count_letters(items):
    result = {}
    for item in items:
        if isinstance(item, (int, float)):
            result[item] = result.get(item, 0) + 1
    return result
```

You can also directly use the `Counter` class of the `collections` module in the Python standard library to solve this problem. `Counter` is a subclass of `dict`, which uses each element in the incoming sequence as a key, and the element appears The number of times is used as the value to construct the dictionary.

```Python
from collections import Counter

def count_letters(items):
    counter = Counter(items)
    return {key: value for key, value in counter.items() \
            if isinstance(key, (int, float))}
```

#### Topic 014: Use Python code to implement the operation of traversing a folder.

> **Comments**: Basically it is also a question of giving people a head. As long as you have used the `os` module, you should know how to do it.

The `walk` function of the `os` module of the Python standard library provides the function of traversing a folder, and it returns a generator.

```Python
import os

g = os.walk('/Users/Hao/Downloads/')
for path, dir_list, file_list in g:
    for dir_name in dir_list:
        print(os.path.join(path, dir_name))
    for file_name in file_list:
        print(os.path.join(path, file_name))
```

> **Explanation**: The `os.path` module provides many tool functions for path operations, which are often used in project development. If the topic clearly requires that the `os.walk` function cannot be used, then the `os.listdir` function can be used to obtain the files and folders in the specified directory, and then the `os.isdir` function can be used to determine which folders are through the loop traversal , For folders, you can traverse through **recursive call**, which can also realize the operation of traversing a folder.

#### Question 015: There are currently three denominations of 2 yuan, 3 yuan, and 5 yuan. If you need to change 99 yuan, how many ways are there?

> **Comment**: There is also a very similar question: "A kid can walk 1 step, 2 steps or 3 steps at a time when walking up stairs. How many ways are there after 10 steps?" , The idea of ​​these two topics is the same, if you use recursive function to write it is very simple.

```Python
from functools import lru_cache


@lru_cache()
def change_money(total):
    if total == 0:
        return 1
    if total <0:
        return 0
    return change_money(total-2) + change_money(total-3) + \
        change_money(total-5)
```

> **Explanation**: In the above code, we decorate the recursive function `change_money` with the `lru_cache` decorator. If this optimization is not done, the asymptotic time complexity of the above code will be $O(3^ N)$, and if the value of the parameter `total` is `99`, the amount of calculation is very huge. The `lru_cache` decorator caches the execution result of the function, so that it can reduce the overhead caused by repeated operations. This is a space-for-time strategy, and it is also a programming idea of ​​dynamic programming.

#### Question 016: Write a function, given the order of the matrix `n`, output a spiral digital matrix.

> For example: n = 2, return:
>
> ```
> 1 2
> 4 3
> ```
>
> For example: n = 3, return:
>
> ```
> 1 2 3
> 8 9 4
> 7 6 5
> ```

The subject itself is not complicated, the code below is for reference only.

```Python
def show_spiral_matrix(n):
    matrix = [[0] * n for _ in range(n)]
    row, col = 0, 0
    num, direction = 1, 0
    while num <= n ** 2:
        if matrix[row][col] == 0:
            matrix[row][col] = num
            num += 1
        if direction == 0:
            if col <n-1 and matrix[row][col + 1] == 0:
                col += 1
            else:
                direction += 1
        elif direction == 1:
            if row <n-1 and matrix[row + 1][col] == 0:
                row += 1
            else:
                direction += 1
        elif direction == 2:
            if col> 0 and matrix[row][col-1] == 0:
                col -= 1
            else:
                direction += 1
        else:
            if row> 0 and matrix[row-1][col] == 0:
                row -= 1
            else:
                direction += 1
        direction %= 4
    for x in matrix:
        for y in x:
            print(y, end='\t')
        print()
```

#### Question 017: Read the following code and write the results of the program.

```Python
items = [1, 2, 3, 4]
print([i for i in items if i> 2])
print([i for i in items if i% 2])
print([(x, y) for x, y in zip('abcd', (1, 2, 3, 4, 5))])
print({x: f'item{x ** 2}' for x in (2, 4, 6)})
print(len({x for x in'hello world' if x not in'abcdefg'}))
```

> **Review**: **Generative (derivative) is one of the characteristic grammars of Python and is almost a must-test content for interviews**. Python can create lists, sets, and dictionaries through generative literal syntax.

```
[3, 4]
[1, 3]
[('a', 1), ('b', 2), ('c', 3), ('d', 4)]
{2:'item4', 4:'item16', 6:'item36'}
6
```

#### Subject 018: Tell the result of the following code.

```Python
class Parent:
    x = 1

class Child1(Parent):
    pass

class Child2(Parent):
    pass

print(Parent.x, Child1.x, Child2.x)
Child1.x = 2
print(Parent.x, Child1.x, Child2.x)
Parent.x = 3
print(Parent.x, Child1.x, Child2.x)
```

> **Comment**: Running the above code first outputs `1 1 1`, you should have no doubt about this. Next, rebind the property `x` to the class `Child1` through `Child1.x = 2` and assign it to `2`, so `Child1.x` will output `2`, and `Parent` and `Child2 `Not affected. Executing `Parent.x = 3` will re-assign the `x` property of the `Parent` class to `3`. Since the `x` property of `Child2` inherits from `Parent`, the value of `Child2.x` is also `3`; And before we rebind the `x` property for `Child1`, then its `x` property value will not be affected by `Parent.x = 3`, but the previous value `2`.

```
1 1 1
1 2 1
3 2 3
```

#### Topic 19: Tell me which modules in the Python standard library have you used.

> **Comment**: There are many modules in the Python standard library. It is recommended that you introduce the standard library and tripartite libraries you have used based on your past project experience, because these are the ones you are most familiar with and can withstand the interviewer's depth. Dug.

| Module name | Introduction |
| ----------- | ------------ |
| sys | Variables and functions related to the Python interpreter, for example: `sys.version`, `sys.exit()` |
| os | Functions related to the operating system, for example: `os.listdir()`, `os.remove()` |
| re | Functions related to regular expressions, for example: `re.compile()`, `re.search()` |
| math | Functions related to mathematical operations, for example: `math.pi`, `math.e`, `math.cos` |
| logging | Classes and functions related to the logging system, for example: `logging.Logger`, `logging.Handler` |
| json / pickle | A module that implements object serialization and deserialization, for example: `json.loads`, `json.dumps` |
| hashlib | A module that encapsulates a variety of hash digest algorithms, for example: `hashlib.md5`, `hashlib.sha1` |
| urllib | Contains URL-related sub-modules, such as: `urllib.request`, `urllib.parse` |
| itertools | Modules that provide various iterators, for example: `itertools.cycle`, `itertools.product` |
| functools | Function related tool modules, for example: `functools.partial`, `functools.lru_cache` |
| collections / heapq | A module that encapsulates common data structures and algorithms, for example: `collections.deque` |
| threading / multiprocessing | Multithreading/multiprocessing related classes and function modules, for example: `threading.Thread` |
| concurrent.futures / asyncio | Concurrent programming/asynchronous programming related classes and function modules, for example: `ThreadPoolExecutor` |
| base64 | A module that provides BASE-64 encoding related functions, for example: `bas64.encode` |
| csv | Modules related to reading and writing CSV files, for example: `csv.reader`, `csv.writer` |
| profile / cProfile / pstats | Modules related to code performance analysis, for example: `cProfile.run`, `pstats.Stats` |
| unittest | Modules related to unit testing, for example: `unittest.TestCase` |

#### Topic 20: What is the difference between `__init__` and `__new__` methods?

Calling the constructor to create an object in Python is a two-stage construction process. First, the `__new__` method is executed to obtain the memory space needed to save the object, and then the filling of the memory space data (initialization of object attributes) is performed through `__init__`. The return value of the `__new__` method is the created Python object (reference), and the first parameter of the `__init__` method is this object (reference), so the initialization of the object can be completed in `__init__`. `__new__` is a class method, its first parameter is a class, and `__init__` is an object method, and its first parameter is an object.

#### Topic 21: Enter the year, month, and day, and determine whether this date is the day of the year.

Method 1: Do not use the modules and functions in the standard library.

```Python
def is_leap_year(year):
    """Judge whether the specified year is a leap year, return False for normal years, and return True for leap years"""
    return year% 4 == 0 and year% 100 != 0 or year% 400 == 0

def which_day(year, month, date):
    """Calculate the date passed in is the day of the year"""
    # Use nested lists to save the days of each month in normal years and leap years
    days_of_month = [
        [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31],
        [31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
    ]
    days = days_of_month[is_leap_year(year)][:month-1]
    return sum(days) + date
```

Method 2: Use the `datetime` module in the standard library.

```Python
import datetime

def which_day(year, month, date):
    end = datetime.date(year, month, date)
    start = datetime.date(year, 1, 1)
    return (end-start).days + 1
```

#### Topic 22: What tools are used for static code analysis in normal work?

> **Review**: Static code analysis tools can extract various static properties from the code, which allows developers to have a better understanding of the complexity, maintainability and readability of the code, as mentioned here Static properties include:
>
> 1. Does the code comply with coding standards, for example: PEP-8.
> 2. Potential problems in the code, including: syntax errors, indentation problems, missing imports, variable coverage, etc.
> 3. Bad smell in the code.
> 4. The complexity of the code.
> 5. The logic of the code.

[Pylint](<https://www.pylint.org/>) and [Flake8](<https://flake8.pycqa.org/en/latest/>) are mainly used in static code analysis at work. Pylint can check out problems such as code errors, bad smells, and irregular codes. The newer version also provides code complexity statistics and can generate inspection reports. Flake8 encapsulates Pyflakes (check code logic errors), McCabe (check code complexity) and Pycodestyle (check whether the code complies with the PEP-8 specification) tools, which can perform the checks provided by these three tools.

#### Topic 23: Tell me about the magic methods in Python you know.

> **Comment**: Magic method is also called magic method, which is a characteristic grammar in Python, and it is also a high-frequency question in interviews.

| Magic method | Function |
| ------------- | ------- |
| `__new__`, `__init__`, `__del__` | Related to creating and destroying objects |
| `__add__`, `__sub__`, `__mul__`, `__div__`, `__floordiv__`, `__mod__` | Arithmetic operator related |
| `__eq__`, `__ne__`, `__lt__`, `__gt__`, `__le__`, `__ge__` | Relational operator related |
| `__pos__`, `__neg__`, `__invert__` | Unary operator related |
| `__lshift__`, `__rshift__`, `__and__`, `__or__`, `__xor__` | Bitwise operation related |
| `__enter__`, `__exit__` | Context Manager Protocol |
| `__iter__`, `__next__`, `__reversed__` | Iterator Protocol |
| `__int__`, `__long__`, `__float__`, `__oct__`, `__hex__` | Type/base conversion related |
| `__str__`, `__repr__`, `__hash__`, `__dir__` | Object representation related |
| `__len__`, `__getitem__`, `__setitem__`, `__contains__`, `__missing__` | Sequence related |
| `__copy__`, `__deepcopy__` | Object copy related |
| `__call__`, `__setattr__`, `__getattr__`, `__delattr__` | Other magic methods |

#### Topic 24: What do the function parameters `*arg` and `**kwargs` represent?

In Python, function parameters are divided into positional parameters, variable parameters, keyword parameters, and named keyword parameters. `*args` represents variable parameters, which can receive `0` or any number of parameters. When you are not sure how many positional parameters the caller will pass in, you can use variable parameters, which will pack the incoming parameters Into a tuple. `**kwargs` stands for keyword parameters, which can receive parameters passed in as `parameter name=parameter value`, and the passed parameters will be packaged into a dictionary. If you use both `*args` and `**kwargs` when defining a function, then the function can receive any parameters.

#### Topic 25: Write a decorator that records the execution time of a function.

> **Comments**: High-frequency interview questions, also the simplest decorator, what the interviewer **must master**.

Method 1: Use functions to implement decorators.

```Python
from functools import wraps
from time import time


def record_time(func):
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time()
        result = func(*args, **kwargs)
        print(f'{func.__name__} execution time: (time()-start} seconds')
        return result
        
    return wrapper
```

Method 2: Implement decorators with classes. The class has the magic method of `__call__`. The object of this class is a callable object and can be used as a decorator.

```Python
from functools import wraps
from time import time


class Record:
    
    def __call__(self, func):
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time()
            result = func(*args, **kwargs)
            print(f'{func.__name__} execution time: (time()-start} seconds')
            return result
        
        return wrapper
```

> **Description**: The decorator can be used to decorate classes or functions to provide them with additional capabilities. It belongs to the **agent mode** in the design pattern.

> **Extension**: **The decorator itself can also be parameterized**, for example, in the above example, if you don’t want to display the execution time of the function in the terminal but want the caller to decide how to output the execution time of the function , Can be done by parameterizing the decorator, the code is shown below.

```Python
from functools import wraps
from time import time


def record_time(output):
    """Parameterizable decorator"""
To
def decorate(func):
To
@wraps(func)
def wrapper(*args, **kwargs):
start = time()
result = func(*args, **kwargs)
output(func.__name__, time()-start)
return result
            
return wrapper
To
return decorate
```

#### Topic 26: What is duck typing?

Duck type is a method used by dynamic type language to judge whether an object is a certain type, also called duck judgment method. Simply put, duck type refers to judging whether a bird is a duck. We only care about whether it swims like a duck, howls like a duck, and whether it walks like a duck is enough. In other words, if the object's behavior is consistent with our expectations (can receive certain messages), we assume that it is a certain type of object.

In the Python language, there are many bytes-like objects (eg: `bytes`, `bytearray`, `array.array`, `memoryview`), file-like objects (eg: `StringIO`, `BytesIO`, `GzipFile `, `socket`), path-like objects (such as: `str`, `bytes`), among which file-like objects can support `read` and `write` operations, and can be read and written like files. This is the so-called The behavior of a duck can be judged as a duck's judgment method. Another example is the `extend` method of lists in Python. The parameters it needs do not have to be lists, as long as it is an iterable object, there is no problem.

> **Explanation**: The duck type of dynamic language greatly simplifies the application of design patterns.

#### Topic 27: Tell me about the scope of variables in Python.

There are four scopes in Python: local scope (**L**ocal), nested scope (**E**mbedded), global scope (**G**lobal), and built-in scope (**B**uilt-in), when searching for an identifier, the search will be performed in the order of **LEGB**. If the identifier is not found in all scopes, a `NameError` exception will be raised.

#### Topic 28: Tell me about your understanding of closures.

Closure is a technology that implements lexical binding in programming languages ​​(Python, JavaScript, etc.) that support first-class functions. When capturing a closure, its free variables (variables defined outside the function but used inside the function) will be determined at the time of capture, so that it can run as usual even if it is out of the context of the capture. Simply put, a closure can be understood as a function that can read the internal variables of other functions. Under the circumstances, the local variable of the function ends its life cycle after the function call ends, but the **closure makes the life cycle of the local variable extended**. When using closures, you need to pay attention. Closures will prevent the objects created in the function from being garbage collected, which may cause a lot of memory overhead, so **closures must not be abused**.

#### Topic 29: Talk about the application scenarios and advantages and disadvantages of multithreading and multiprocess in Python.

Thread is the basic unit of operating system to allocate CPU, process is the basic unit of operating system to allocate memory. Usually the program we run will contain one or more processes, and each process contains one or more threads. The advantage of multithreading is that multiple threads can share the memory space of the process, so the communication between processes is very easy to achieve; but if the official CPython interpreter is used, multithreading is subject to the GIL (Global Interpreter Lock) and cannot use the CPU. Multi-core features, this is a big problem. The use of multiple processes can make full use of the multi-core characteristics of the CPU, but the communication between processes is relatively troublesome and requires the use of IPC mechanisms (pipes, sockets, etc.).

Multithreading is suitable for I/O-intensive applications that spend a lot of time on I/O operations, but do not have too many parallel computing requirements and do not take up too much memory. Multi-process is suitable for performing computationally intensive tasks (such as video encoding and decoding, data processing, scientific computing, etc.), tasks that can be decomposed into multiple parallel subtasks and the results of subtasks can be combined, and there are no restrictions on memory usage and no Tasks that are strongly dependent on I/O operations.

> **Extension**: There are usually three options for concurrent programming in Python: multithreading, multiprocessing and asynchronous programming. Asynchronous programming achieves cooperative concurrency. Through the user mode switching of multiple cooperating subprograms, efficient use of the CPU is realized. This method is also very suitable for I/O-intensive applications.

#### Topic 30: Tell me about the difference between Python 2 and Python 3.

> **Comments**: Don't memorize the so-called reference answer for this kind of question. It is enough to say something that you are most familiar with.

1. Both `print` and `exec` in Python 2 are keywords, which became functions in Python 3.
2. There is no `long` type in Python 3. Integers are all `int` types.
3. The inequality sign `<>` in Python 2 is discarded in Python 3, and `!=` is used uniformly.
4. The `xrange` function in Python 2 is replaced by the `range` function in Python 3.
5. Python 3 improves the insecure `input` function in Python 2, and discards the `raw_input` function.
6. The `file` function in Python 2 is replaced by the `open` function in Python 3.
7. The `/` operation in Python 2 is for the type of `int` to divide. In Python 3, you need to use `//` to do the division.
8. In Python 3, the code for catching exceptions in Python 2 has been improved. Obviously, Python 3 is written in a more reasonable way.
9. The scope of loop variables in Python 3 productions has been better controlled and will not affect variables with the same name outside the productions.
10. The `round` function in Python 3 can return the `int` or `float` type, and the `round` function in Python 2 returns the `float` type.
11. The `str` type of Python 3 is a Unicode string, and the `str` type of Python 2 is a byte string, which is equivalent to the `bytes` in Python 3.
12. The comparison operators in Python 3 must compare objects of the same kind.
13. Classes defined in Python 3 are all new-style classes. The classes defined in Python 2 are divided into new-style classes (classes that explicitly inherit from `object`) and old-style classes (classic classes). New-style classes and old-style classes are in MRO There is a very significant difference in the problem. The new-style class can use the `__class__` attribute to get its own type, and the new-style class can use the `__slots__` magic.
14. Python 3 has stricter requirements for code indentation. If spaces and tabs are mixed, a `TabError` will be raised.
15. The `keys`, `values`, and `items` methods of the dictionary in Python 3 no longer return the `list` object, but return the `view object`, and the built-in `map`, `filter` and other functions are no longer Return a `list` object, but return an iterator object.
16. The names of some modules in the Python 3 standard library are different from those of Python 2. In terms of tripartite libraries, some tripartite libraries only support Python 2, and some only support Python 3.

#### Topic 31: Talk about your understanding of "monkey patching".

"Monkey patch" is a feature of dynamically typed languages. When the code is running, the methods, attributes, functions, etc. in the code are changed without modifying the source code to achieve the effect of a hot patch. Many system security patches are also implemented through monkey patches, but the use of monkey patches should be avoided in actual development to avoid inconsistent code behavior.

When using the `gevent` library, we will execute `gevent.monkey.patch_all()` at the beginning of the code. The function of this line of code is to replace the `socket` module in the standard library, so that we are using In the case of `socket`, the code can be coroutineized without modifying any code to achieve the purpose of improving performance. This is the application of the monkey patch.

In addition, if you want to replace the `json` in the standard library with the `ujson` tripartite library, you can also use the monkey patch method. The code is shown below.

```Python
import json, ujson

json.__name__ ='ujson'
json.dumps = ujson.dumps
json.loads = ujson.loads
```

The `Mock` technology in unit testing is also an application of monkey patches. The `unittest.mock` module in Python is a module that solves the problem of substituting `Mock` objects for the objects dependent on the tested objects in unit tests.

#### Topic 32: Read the following code to tell the result of the operation.

```Python
class A:
    def who(self):
        print('A', end='')

class B(A):
    def who(self):
        super(B, self).who()
        print('B', end='')

class C(A):
    def who(self):
        super(C, self).who()
        print('C', end='')

class D(B, C):
    def who(self):
        super(D, self).who()
        print('D', end='')

item = D()
item.who()
```

> **Comments**: This question has found two knowledge points:
>
> 1. MRO (Method Resolution Order) in Python. In the absence of multiple inheritance, a message is sent to the object. If the object does not have a corresponding method, the order of upward (parent) search is very clear. If you go back to the `object` class (the parent class of all classes) and no corresponding method is found, then an `AttributeError` exception will be raised. But when there is multiple inheritance, especially when diamond inheritance (diamond inheritance) appears, the MRO must be determined by tracing back to the end to find which method should be found. The classes in Python 3 and the new-style classes in Python 2 use [C3 algorithm](<https://www.jianshu.com/p/a08c61abe895>) to determine MRO, which is a method similar to breadth first search; Old-style classes (classic classes) in Python 2 use depth-first search to determine MRO. If you are not sure about the MRO, you can use the `mro` method or the `__mro__` attribute of the class to get the MRO list of the class.
> 2. Use of `super()` function. When using the `super` function, you can use `super(type, object)` to specify which object to start from which class to search for the parent class method. So the `super(B, self).who()` in the above `B` class code means that starting from class B, search upwards for the `who` method of `self` (object of class D), so you will find `C` The `who` method in the class, because the MRO list of the `D` class object is `D --> B --> C --> A --> object`.

```
ACBD
```

#### Topic 33: Write a function to evaluate inverse Polish expressions. You cannot use Python's built-in functions.

> **Review**: [Reverse Polish expression](<https://baike.baidu.com/item/%E9%80%86%E6%B3%A2%E5%85%B0%E5%BC% 8F/128437>) is also called "postfix expression". Compared with the usual "infix expression", reverse Polish expression does not require parentheses to determine the priority of the operation, such as `5 * (2 + 3 )` The corresponding reverse Polish expression is `5 2 3 + *`. Inverse Polish expression evaluation requires the use of a stack structure. The scan expression will be pushed into the stack when it encounters an operand, and two elements will be popped out of the stack when it encounters an operator, and the result of the operation will be pushed into the stack. After the expression scan is over, there is only one number in the stack, and this number is the final operation result, which can be popped directly from the stack.

```Python
import operator


class Stack:
    """Stack (FILO)"""

    def __init__(self):
        self.elems = []
    
    def push(self, elem):
        """Into the stack"""
        self.elems.append(elem)
    
    def pop(self):
        """Pull"""
        return self.elems.pop()
    
    @property
    def is_empty(self):
        """Check if the stack is empty"""
        return len(self.elems) == 0


def eval_suffix(expr):
    """Reverse Polish expression evaluation"""
    operators = {
        '+': operator.add,
        '-': operator.sub,
        '*': operator.mul,
        '/': operator.truediv
    }
    stack = Stack()
    for item in expr.split():
        if item.isdigit():
            stack.push(float(item))
        else:
            num2 = stack.pop()
            num1 = stack.pop()
            stack.push(operators[item](num1, num2))
    return stack.pop()
```

#### Topic 34: How to implement string replacement operations in Python?

There are roughly two ways to implement string replacement in Python: the `replace` method of strings and the `sub` method of regular expressions.

Method 1: Use the `replace` method of strings.

```Python
message ='hello, world!'
print(message.replace('o','O').replace('l','L').replace('he','HE'))
```

Method 2: Use the `sub` method of regular expressions.

```Python
import re

message ='hello, world!'
pattern = re.compile('[aeiou]')
print(pattern.sub('#', message))
```

> **Extension**: There is also a related interview question. Sort the list of saved file names. The file names are required to be sorted by alphabet and number size. For example, for the list `filenames = ['a12.txt','a8 .txt','b10.txt','b2.txt','b19.txt','a3.txt'] `, the result of sorting is `['a3.txt','a8.txt','a12 .txt','b2.txt','b10.txt','b19.txt']`. As a reminder, you can fill in the file name by replacing the string, and use the `sorted` function to sort the file name after the fill. You can think about how to solve this problem.

#### Topic 35: How to analyze the execution performance of Python code?

Profiling code performance can use the `cProfile` and `pstats` modules in the Python standard library. The `run` function of `cProfile` can execute the code and collect statistics, create a `Stats` object and print a simple profiling report. `Stats` is a class in the `pstats` module, which is a statistical object. Of course, you can also use the tripartite tools `line_profiler` and `memory_profiler` to analyze the time and memory consumed by each line of code. These two tripartite tools will output the profiling structure in a very friendly way. If you use PyCharm, you can use the "Profile" menu item of the "Run" menu to perform performance analysis on the code. In PyCharm, you can use a table or call graph to display the results of the performance analysis.

The following is an example of profiling code performance using `cProfile`.

`example.py`

```Python
import cProfile


def is_prime(num):
    for factor in range(2, int(num ** 0.5) + 1):
        if num% factor == 0:
            return False
    return True


class PrimeIter:

    def __init__(self, total):
        self.counter = 0
        self.current = 1
        self.total = total

    def __iter__(self):
        return self

    def __next__(self):
        if self.counter <self.total:
            self.current += 1
            while not is_prime(self.current):
                self.current += 1
            self.counter += 1
            return self.current
        raise StopIteration()


cProfile.run('list(PrimeIter(10000))')
```

If you use the `line_profiler` tripartite tool, you can directly analyze the performance of each line of code of the `is_prime` function. You need to add a `profiler` decorator to the `is_prime` function. The code is shown below.

```Python
@profiler
def is_prime(num):
    for factor in range(2, int(num ** 0.5) + 1):
        if num% factor == 0:
            return False
    return True
```

Install `line_profiler`.

```Bash
pip install line_profiler
```

Use `line_profiler`.

```Bash
kernprof -lv example.py
```

The result of the operation is shown below.

```
Line # Hits Time Per Hit% Time Line Contents
================================================== ============
     1 @profile
     2 def is_prime(num):
     3 86624 48420.0 0.6 50.5 for factor in range(2, int(num ** 0.5) + 1):
     4 85624 44000.0 0.5 45.9 if num% factor == 0:
     5 6918 3080.0 0.4 3.2 return False
     6 1000 430.0 0.4 0.4 return True
```

#### Question 36: How to use the `random` module to generate random numbers, realize random disorder and random sampling?

> **Comment**: The subject of giving people a head, because the commonly used modules in the Python standard library should be familiar to Python developers. If this question cannot be answered, the entire interview will basically be ruined.

1. The `random.random()` function can generate random floating point numbers between `[0.0, 1.0)`.
2. The `random.uniform(a, b)` function can generate a random floating point number between `[a, b]` or `[b, a]`.
3. The `random.randint(a, b)` function can generate a random integer between `[a, b]` or `[b, a]`.
4. The `random.shuffle(x)` function can realize random shuffle of the sequence `x` in place.
5. The `random.choice(seq)` function can extract a random element from a non-empty sequence.
6. The `random.choices(population, weights=None, *, cum_weights=None, k=1)` function can randomly sample from the population (with replacement sampling) out of a sample of capacity `k` and return a list of samples , You can specify the weight of the individual through the parameter. If the weight is not specified, the probability of the individual being selected is equal.
7. The `random.sample(population, k)` function can randomly sample (sample without replacement) from the population and get a sample with a capacity of `k` and return a list of samples.

> **Extension**: In addition to generating uniformly distributed random numbers, the function provided by the `random` module can also generate random numbers with other distributions. For example, the `random.gauss(mu, sigma)` function can generate Gaussian distribution (positive Random distribution); `random.paretovariate(alpha)` function will generate Pareto distributed random numbers; `random.gammavariate(alpha, beta)` function will generate gamma distributed random numbers.

#### Title 37: Explain the working principle of the thread pool.

> **Review**: Pooling technology is a typical space-for-time strategy. The database connection pools and thread pools we use are all applications of pooling technology. The `ThreadPoolExecutor of the Python standard library `currrent.futures` module `Is the realization of the thread pool, if you want to figure out its working principle, you can refer to the following.

The thread pool is a technology used to reduce the overhead caused by the creation and destruction of the thread itself, which is a typical space-for-time operation. If the application needs to frequently dispatch tasks to threads for execution, the thread pool is a must, because the creation and release of threads involves a large number of low-level system operations, and the overhead is relatively high. If the application can be used during the working period, the threads will be created and released. The operation becomes the pre-creation and borrowing operation, which will greatly reduce the underlying overhead. After the application is started, the thread pool creates a certain number of threads and puts them in the idle queue. These threads are initially blocked and will not consume CPU resources, but will take up a small amount of memory space. When the task arrives, take out an idle thread from the queue, dispatch the task to this thread to run, and mark the thread as occupied. When all threads in the thread pool are occupied, you can choose to automatically create a certain number of new threads for processing more tasks, or you can choose to queue tasks until there are free threads available. After the task is executed, the thread does not exit and ends, but continues to remain in the pool waiting for the next task. When the system is relatively idle, when most threads are idle for a long time, the thread pool can automatically destroy some threads and reclaim system resources. Based on this pre-creation technology, the thread pool allocates the overhead caused by thread creation and destruction to each specific task. The more the number of executions, the smaller the thread overhead shared by each task.

Generally, thread pools must have the following components:

1. Thread pool manager: used to create and manage thread pools.
2. Work threads and thread queues: the threads actually executed in the thread pool and the containers that hold these threads.
3. Task interface: abstract the tasks performed by threads to form a task interface to ensure that the thread pool has nothing to do with specific tasks.
4. Task queue: A container in the thread pool that holds tasks waiting to be executed.

#### Question 38: Give examples of situations where `KeyError`, `TypeError`, and `ValueError` will occur.

To give a simple example, the variable `a` is a dictionary, and performing the operation of `int(a['x'])` may cause the above three types of exceptions. If there is no key `x` in the dictionary, a `KeyError` will be raised; if the value corresponding to the key `x` is not `str`, `float`, `int`, `bool` and `bytes-like` type, call ` When the int` function constructs an object of type `int`, it will raise a `TypeError`; if `a[x]` is a string or byte string, and the corresponding content cannot be processed into `int`, it will raise ` ValueError`.

#### Subject 39: Tell the results of the following code.

```Python
def extend_list(val, items=[]):
    items.append(val)
    return items

list1 = extend_list(10)
list2 = extend_list(123, [])
list3 = extend_list('a')
print(list1)
print(list2)
print(list3)
```

> **Review**: When a Python function is defined, the value of the default parameter `items` is calculated, that is, `[]`. Because the default parameter `items` refers to the object `[]`, every time you call this function, if you operate on the list referenced by `items`, the next time you call, the default parameter will still refer to the previous list instead of reassigning `[]`, so there will be previously added elements in the list. If you re-assign `items` by passing parameters, then `items` will refer to the new list object instead of the default list object. This question is often asked in interviews. It is usually not recommended to use the default parameters of the container type. Code inspection tools like PyLint will also question and warn this code.

```
[10,'a']
[123]
[10,'a']
```

#### Question 40: How to read a large file, for example, the memory is only 4G, how to read a file with a size of 8G?

Obviously, it is unrealistic for 4G memory to load 8G files at one time. In this case, you must consider multiple reading and batch processing. To read a file in Python, you can first obtain the file object through the `open` function. When reading the file, you can specify the read size through the `size` parameter of the `read` method, or you can use the `offset` of the `seek` method The `parameter specifies the position to read, so that you can control the number of bytes and total bytes of data read at a time. In addition, you can use the built-in function `iter` to process file objects into iterator objects, and only read a small amount of data for processing each time. The code is roughly written as follows.

```Python
with open('...','rb') as file:
    for data in iter(lambda: file.read(2097152), b''):
        pass
```

On Linux systems, large files can be cut into small pieces by the `split` command, and then the data can be processed by reading the cut small files. For example, the following command cuts a large file named `filename` into multiple files with a size of 512M.

```Bash
split -b 512m filename
```

If you want, you can also cut the file named `filename` into 10 files, the command is as follows.

```Bash
split -n 10 filename
```

> **Extended**: The external sorting is very similar to the above situation, because the processed data cannot be loaded into the memory at one time, and can only be placed on the external memory (usually a hard disk) with slower reading and writing. "**Sort-Merge Algorithm**" is a commonly used external sorting strategy. In the sorting phase, first read in the amount of data that can be placed in the memory, sort and output it to a temporary file, and proceed accordingly, organize the data to be sorted into multiple ordered temporary files, and then merge these temporary files in the merge phase The files are combined into a large ordered file, and this large ordered file is the result of sorting.

#### Topic 41: Tell me about your understanding of modules and packages in Python.

Each Python file is a module, and the folder where these files are saved is a package, but this folder as a Python package must have a file named `__init__.py`, otherwise the package cannot be imported. Usually, a folder can also have subfolders, which means that a package can also have subpackages. The `__init__.py` in the subpackage is not necessary. Modules and packages solve the problem of naming conflicts in Python. Different packages can have modules with the same name, and different modules can have variables, functions, or classes with the same name. In Python, you can use `import` or `from ... import ...` to import packages and modules. When importing, you can also use the `as` keyword to alias packages, modules, classes, functions, variables, etc. , So as to completely solve the problem of naming conflicts in programming, especially when developing multi-person collaborative teams.

#### Topic 42: Tell me about the Python coding standards you know.

> **Review**: The company’s Python coding standards basically refer to [PEP-8](<https://www.python.org/dev/peps/pep-0008/>) or [Google Open Source Project Style Guide ](<https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/>), the latter also mentioned that the Lint tool can be used to check the degree of standardization of the code. Interview When encountering this kind of problem, you can first talk about these two reference standards, and then focus on the precautions for Python coding.

1. Use of spaces
   -Use spaces to indicate indentation and do not use tabs (Tab).
   -Each level of indentation related to grammar is represented by 4 spaces.
   -The number of characters in each line should not exceed 79 characters. If the expression is too long and occupies multiple lines, all lines except the first line should be indented with 4 spaces in addition to the normal width.
   -The definition of functions and classes must be separated by two blank lines before and after the code.
   -In the same class, each method should be separated by a blank line.
   -There should be one space on the left and right sides of the binary operator, and only one space is fine.
2. Identifier naming
   -Variables, functions, and properties should be spelled in lowercase letters, and if there are multiple words, use underscores to connect them.
   -The protected instance attributes in the class should start with an underscore.
   -Private instance attributes in the class should start with two underscores.
   -The names of classes and exceptions should be capitalized.
   -Module-level constants should be in all capital letters, and if there are multiple words, use underscores to connect them.
   -For instance methods of the class, the first parameter should be named `self` to represent the object itself.
   -For the class method of the class, the first parameter should be named `cls` to indicate the class itself.
3. Expressions and statements
   -Use the inline form of the negative word instead of putting the negative word in front of the entire expression. For example: `if a is not b` is easier to understand than `if not a is b`.
   -Don't use the method of checking the length to determine whether a string, list, etc. is `None` or has no elements, you should use `if not x` to check it.
   -Even if there is only one line of code in `if` branch, `for` loop, `except` exception capture, etc., do not write the code together with `if`, `for`, `except`, etc., and write the code separately clearer.
   -The `import` statement is always placed at the beginning of the file.
   -When importing modules, `from math import sqrt` is better than `import math`.
   -If there are multiple `import` statements, they should be divided into three parts, from top to bottom are Python **standard module**, **third-party module** and **custom module**, each The parts inside should be arranged according to the **alphabetical order** of the module name.

#### Question 43: If you run the following code, will it report an error? If you report an error, please explain what is wrong. If you don't report an error, please tell the code execution result.

```Python
class A:
    def __init__(self, value):
        self.__value = value

    @property
    def value(self):
        return self.__value

obj = A(1)
obj.__value = 2
print(obj.value)
print(obj.__value)
```

> **Comment**: There are two points of investigation in this question, one point is the understanding of the object property access permissions at the beginning of `_` and `__` and the `@property` decorator, and the other is The understanding of dynamic language does not require too much explanation.

```
1
2
```

> **Extension**: If you don't want to dynamically add new attributes to the object when the code is running, you can use the `__slots__` magic when defining the class. For example, we can add a line `__slots__ = ('__value', )` to the above `A`, and run the above code again, and an `AttributeError` error will be generated at the original line 10.

#### Topic 44: Sort the keys of the dictionary given below in descending order of value.

```Python
prices = {
    'AAPL': 191.88,
    'GOOG': 1186.96,
    'IBM': 149.24,
    'ORCL': 48.44,
    'ACN': 166.89,
    'FB': 208.09,
    'SYMC': 21.29
}
```

> **Review**: The high-level usage of the `sorted` function often appears in interviews. The `key` parameter can be passed in a function name or a Lambda function, and the return value of the function represents the comparison of elements when sorting in accordance with.

```Python
sorted(prices, key=lambda x: prices[x], reverse=True)
```

#### Topic 45: Tell me about the usage and function of `namedtuple`.

> **Remarks**: The `collections` module of the Python standard library provides a lot of useful data structures, which are not clear to every developer. For example, the question called `namedtuple`, in the interview I participated in Among them, 90% of interviewers cannot accurately state its role and application scenarios. In addition, `deque` is also a very useful but often overlooked class, as well as `Counter`, `OrderedDict`, `defaultdict`, `UserDict` and other classes. Do you know their usage?

When using an object-oriented programming language, defining a class is the most common thing. Sometimes, we will use a class with only attributes but no methods. The objects of this class are usually only used to organize data and cannot receive messages. So we call this kind of data class or degenerate class, just like the structure in the C language. We don't recommend using this degenerate class. You can use `namedtuple` (named tuple) to replace this class in Python.

```Python
from collections import namedtuple

Card = namedtuple('Card', ('suite','face'))
card1 = Card('heart', 13)
card2 = Card('Grass Flower', 5)
print(f'{card1.suite}{card1.face}')
print(f'{card2.suite}{card2.face}')
```

Named tuples are immutable containers like ordinary tuples. Once the data is stored in the top-level attribute of `namedtuple`, the data can no longer be modified, which means that all attributes on the object follow "write once, multiple The principle of “read twice”. Unlike ordinary tuples, the data in named tuples has access names, and the saved data can be obtained by name instead of index. Not only is it easier to operate, but the code is more readable.

The essence of a named tuple is a class, so it can also be used as a parent class to create subclasses. In addition, named tuples have a series of built-in methods. For example, you can use the `_asdict` method to process the named tuples into a dictionary, or you can use the `_replace` method to create a shallow copy of the named tuple object.

```Python
class MyCard(Card):
    
    def show(self):
        faces = ['','A', '2', '3', '4', '5', '6', '7', '8', '9', '10','J', 'Q','K']
        return f'{self.suite}{faces[self.face]}'


print(Card) # <class'__main__.Card'>
card3 = MyCard('square', 12)
print(card3.show()) # Block Q
print(dict(card1._asdict())) # {'suite':'红桃','face': 13}
print(card2._replace(suite='block')) # Card(suite='block', face=5)
```

All in all, named tuples can better organize the data structure and make the code clearer and readable. In many scenarios, it is a substitute for tuples, dictionaries, and data classes. When you need to create immutable classes that take up less space, named tuples are a good choice.

#### Topic 46: Write the corresponding function according to the requirements of the topic.

> **Requirement**: Write a function, pass in a list of several integers, and return this element if the number of occurrences of an element in the list exceeds 50%.

```Python
def more_than_half(items):
    temp, times = None, 0
    for item in items:
        if times == 0:
            temp = item
            times += 1
        else:
            if item == temp:
                times += 1
            else:
                times -= 1
    return temp
```

> **Comment**: The question on LeetCode, which appeared in the Python interview, uses the feature that the number of occurrences of the element exceeds 50%. The occurrence of the same element as `temp` will increase the count value by 1, and appear with `temp` `Different elements will decrement the count value by 1. If the count value is `0`, it means that the previous element has no effect on the final result. Use `temp` to write down the current element and set the count value to `1`. Eventually, the element that appears more than 50% will be assigned to the variable `temp`.

#### Topic 47: Write the corresponding function according to the requirements of the topic.

> **Requirement**: Write a function, the incoming parameter is a list (the elements in the list may also be a list), and return the maximum nesting depth of the list. For example: the nesting depth of the list `[1, 2, 3]` is `1`, and the nesting depth of the list `[[1], [2, [3]]]` is `3`.

```Python
def list_depth(items):
    if isinstance(items, list):
        max_depth = 1
        for item in items:
            max_depth = max(list_depth(item) + 1, max_depth)
        return max_depth
    return 0
```

> **Review**: Seeing the title, it should be natural to think of using recursive methods to check each element in the list.

#### Topic 48: Write the corresponding decorator according to the requirements of the topic.

> **Requirement**: There is a function to obtain data through the network (may be abnormal due to network reasons), write a decorator so that this function can retry the specified number of times when the specified exception occurs, and retry each time Before random delay for a period of time, the longest delay time can be controlled by parameters.

method one:

```Python
from functools import wraps
from random import random
from time import sleep


def retry(*, retry_times=3, max_wait_secs=5, errors=(Exception, )):

    def decorate(func):

        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(retry_times):
                try:
                    return func(*args, **kwargs)
                except errors:
                    sleep(random() * max_wait_secs)
            return None

        return wrapper

    return decorate
```

Method Two:

```Python
from functools import wraps
from random import random
from time import sleep


class Retry(object):

    def __init__(self, *, retry_times=3, max_wait_secs=5, errors=(Exception, )):
        self.retry_times = retry_times
        self.max_wait_secs = max_wait_secs
        self.errors = errors

    def __call__(self, func):

        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(self.retry_times):
                try:
                    return func(*args, **kwargs)
                except self.errors:
                    sleep(random() * self.max_wait_secs)
            return None

        return wrapper
```

> **Remarks**: We have emphasized more than once that decorators are almost a must-question content for Python interviews. This topic is slightly more complicated than the previous one. What it needs is a parameterized decorator.

#### Topic 49: Write a function to achieve string reversal, and write as many methods as you know.

> **Comment**: The problem of bad street is basically a problem of giving away heads.

**Method One**: Reverse Slicing

```Python
def reverse_string(content):
    return content[::-1]
```

**Method Two**: Reverse stitching

```Python
def reverse_string(content):
    return''.join(reversed(content))
```

**Method Three**: Recursive call

```Python
def reverse_string(content):
    if len(content) <= 1:
        return content
    return reverse_string(content[1:]) + content[0]
```

**Method Four**: Deque

```Python
from collections import deque

def reverse_string(content):
    q = deque()
    q.extendleft(content)
    return''.join(q)
```

**Method 5**: Reverse assembly

```Python
from io import StringIO

def reverse_string(content):
    buffer = StringIO()
    for i in range(len(content)-1, -1, -1):
        buffer.write(content[i])
    return buffer.getvalue()
```

**Method 6**: Reverse stitching

```Python
def reverse_string(content):
    return''.join([content[i] for i in range(len(content)-1, -1, -1)])
```

**Method Seven**: Half Exchange

```Python
def reverse_string(content):
    length, content = len(content), list(content)
    for i in range(length // 2):
        content[i], content[length-1-i] = content[length-1-i], content[i]
    return''.join(content)
```

**Method 8**: Counterpoint exchange

```Python
def reverse_string(content):
    length, content = len(content), list(content)
    for i, j in zip(range(length // 2), range(length-1, length // 2-1, -1)):
        content[i], content[j] = content[j], content[i]
    return''.join(content)
```

> **Extension**: These methods are actually the same, and it is enough to give several representative ones during the interview. Let me leave a question for everyone. Which of the above methods have better performance? We have mentioned the methods of analyzing code performance before. You can use these methods to check whether the answers you give are correct.

#### Topic 50: Write the corresponding function according to the requirements of the topic.

> **Requirement**: There are `1000000` elements in the list, and the value range is `[1000, 10000)`. Design a function to find the repeated elements in the list.

```Python
def find_dup(items: list):
    dups = [0] * 9000
    for item in items:
        dups[item-1000] += 1
    for idx, val in enumerate(dups):
        if val> 1:
            yield idx + 1000
```

> **Review**: The solution of this question is the same as the principle of [Counting Sort](<https://www.runoob.com/w3cnote/counting-sort.html>), although the number of elements is very large, but The value range `[1000, 10000)` is not very large, there are only 9000 possible values, so you can use a `dups` list that can store 9000 elements to record the number of occurrences of each element, `dups` list The initial value of all elements is `0`, through the traversal of the elements in the `items` list, when an element appears, the value of the corresponding position in the `dups` list is increased by 1, and finally the value in the `dups` list is greater than 1. The elements of corresponds to the repeated elements in the `items` list.

To view more Python interview questions, please move to my Zhihu column ["Python Interview Collection"](https://zhuanlan.zhihu.com/c_1228980105135497216).

### CREDITS

Original Credits: [骆昊](https://github.com/jackfrued).

English Credits: [Jishan Shaikh](https://github.com/jishanshaikh4)
