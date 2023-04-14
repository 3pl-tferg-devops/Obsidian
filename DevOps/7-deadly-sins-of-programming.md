## Not using programming standards
[[coding-standards]]

Standards give us rules to follow, like whitespacing, file structure and other philosophies that we rely on to make our code consistent. This is especially useful when working in teams because it allows others to have shared expectations when veiwing your code.

If you don't keep standards, it's like switching fonts all the time. You'll understand what's being said but it takes additonal brainpower to figure out what you're looking at. 

#todo : Investigate what standards are available.

## Not learning programming design principles
These are like general guides on how to become a better programmer, they're the raw philosophies of code. 

See below for an example of one such principle:

### SOLID
#### Single Responsibility
We should aim to break our code down into modules of one responsibility each. Doing so will allow you to identify a modules purpose, reuse the module and test it more cleanly.

#### Open/Closed
Design your modules in a way that allows you to add new functionality without having to actually make changes to them. Instead we should extend a module to add to it. 

Once a module is in use, it's locked and shouldn't be modified to add additional features. 

#### Liskov Substitution
We should only extend modules if we're absolutely sure that it's still the same type at heart. 

For example, you have a module that deals with hexagons. We could extend the module to also handle a six pointed star seeing as it's still technically a hexagon. However, if we want to have our module handle pentagons, or a 7 pointed star, we'll want to consider extending something that fits its design or become it's own type instead. 

#### Interface Segregation
Our modules shouldn't need to know about functionality that they don't use. We should split our modules into smaller abstractions, like interfaces, which we can then compose to form an exact set of functionality that the module requires. 

This becomes useful in testing because it allows us to mock out, only the functionality that the module needs. 

#### Dependency Inversion
Instead of talking to other parts of your code directly, you should always communicate abstractly. Typically by the interfaces we define. 

This breaks down any relationships between our code and isolates our modules completely from one another. Meaning we can swap out parts as we need to. 

Since they're communicating with interfaces now, they don't need to know what implementation they are getting, only that they take certain inputs and return a valid output. 

If we combine all these principles together, it ends up decoupling our code. This gives us modules that are completely independant of each other. It makes our code more maintainable, scalable, reusable and testable. 

## Not learning programming design patterns
Patterns give us solutions to our code problems but they aren't fixed implimentations. We use them to architect our software systems, matching the right shapes to fit the needs our sfotware has. 

### Creational Patterns
There to help us make and control new object instances, such as the factory method pattern, which turns a bunch of requirements into different modules that follow the same interface, but aren't necessarily the same type. 

#### Additional Creational Patterns: 
Abstract Factory, Builder, Dependency Injection, Lazy Instantiation, Multiton, Object Pool, Prototype, RAII, Singleton

### Structural Patterns
Concerned with how we organize and manipulate our objects, such as the adaptor pattern, to wrap a module and adapt its interface to one that another module needs.

#### Additional Structural Patterns
Adapter, Bridge, Composite, Decorator, Delegation, Extension Object, Facade, Flyweight, Front Controller, Marker, Module, Proxy, Translator, Twin

### Behavioral Patterns
These focus on how code functions and how it handles communication with other parts of the code, such as the observer pattern, to publish and subscribe to a stream of messages in an event-based architecture. 

#### Additional Behavioral Patterns
Blackboard, Chain of Responsibility, Command, Fluent Interface, Interpreter, Iterator, Memento, Null Object, Observer, Servant, Specification, State, Strategy, Template Method, Visitor.

#todo : Investigate patterns


## Not using descriptive names

Avoid unecessary encodings, such as type information. It makes code harder to read in a natural way. Instead of partList, you could just have parts. 

Expand abbreviations and acronyms to their full names to avoid any miscommunications. 

Use clear distinctions, we shoud aim to use names that more accurately represent the nuances of the code we're working with. 

No "magic" values, we want to replace any instances of hard coded variables with named constants. 

Be descriptive with your names, idealy you want to be descriptive enough that someone reading it will understand what it does but not so verbose that it becomes an information overload. 

	If your code isn't easy to read, it's not good code.

## Not testing your code
There are a couple of different kinds of tests, there is end-to-end testing, unit tests and intergation tests. 

End-to-end testing simulates how an end user will interact with the code. It never actually touches the code itself, only what it delivers. These can be tricky to setup due to the need to always have a fully functional application running, but are suprisingly valuable.

Unit tests verify the operation of our modules in isolation. 

Integration tests, examine the interaction between those modules. 

These provide validation that our code is doing what we expect it to, rather than focusing more on behavior like end-to-end tests. 

	If the thought of writing tests seems overwhelming right now,  try applying the
	solid principles. You might be suprised at how much easier it becomes when your
	code is decoupled. 


## Not giving yourself enough time
When we're giving an estimate on how long we think something should take, triple it. This additional time will give you room for problems you encounter along the way. You will also have room for writing documentation, and tests.



> Any fool can write code that a computer can understand.
> Good programmers write code that humans can understand.
> ~ Martin Fowler

