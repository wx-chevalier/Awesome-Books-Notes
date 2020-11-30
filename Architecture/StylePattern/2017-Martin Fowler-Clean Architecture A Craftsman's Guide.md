# Clean Architecture

# 1.What is design and architecture

The word “architecture” is often used in the context of something at a high level that is divorced from the  lower-level details,whereas "design" more often seems to imply structures and decisions at a lower level.

The goal of software architecture is to minimize the human resources required to build and maintain the required system.

架构设计的目的：减少系统构建和维护的成本。引入例子对比不同开发思路下维护软件的成本，说明架构的重要性。

# 2.A tale of two values

Every software system provides two different values to the stakeholders: behavior and structure. Software developers are responsible for ensuring that both values remain high.

The first value of software--behavior--is urgent but not always particularly important. The second value of software--architecture--is important but never particularly urgent.

只注重功能开发不注重架构的系统是无法延续的。引入 Esenhower Matrix，把要做的事按重要程度和紧急程度划分在四个象限，并且排优先级。架构设计不紧急但是重要，应该放在较高的优先级。

# 3.Paradigm overview

- Structured programming imposes discipline of direct transfer of control.

- Object-oriented programming imposes discipline of indirect transfer of control.

- Functional programming imposes discipline upon assignment.

Each of the paradigms removes capabilities from programmer. None of them add new capabilities. The three paradigms together remove goto statements, function pointers, and assignment.

# 4.Structured programming

Structured programming allows modules to be recursively decomposed into provable units, which in turn means that modules can be functionally decomposed.That is, you can take a large-scale problem statement and decompose it into high-level functions. Each of those functions can then be decomposed into lower-level functions, ad infinitum. Moreover, each of those decomposed functions can be represented using the restricted control structures of structured programming.

Dijkstra once said , "Testing shows the presence ,not the absence ,of bugs." In other words, a program can be proven incorrect by a test ,but it cannot be proven correct. Structured programming forces us to recursively decompose a program into a set of small provable functions. We can then use tests to try to prove those small provable functions incorrect. If such tests fail to prove incorrectness, then we deem the functions to be correct enough for our purposes.

# 5.Object-oriented programming

The fact that OO languages provide safe and convenient polymorphism means that any source code dependency ,no matter where it is, can be inverted.

To the software architect, OO is the ability, through the use of polymorphism, to gain absolute control over every source code dependency in the system. It allows the architect to create a plugin architecture, in which modules that contain high-level policies are independent of modules that contain low-level details.The low-level details are relegated to plugin modules that can be deployed and developed independently from the modules that contain high-level policies.

# 6.Functional programming

Variables in functional languages do not vary. All race conditions , deadlock conditions, and concurrent update problems are due to mutable variables.

Design principles. The SOLID principles tell us how to arrange our functions and data structures into classes, and how those classes should be interconnected.

The goal of the principles is the creation of mid-level software structures that:

1. Tolerate change,
2. Are easy to understand ,and
3. Are the basis of components that can be used in many software systems.

# 7.SIP: The single responsibility principle

SIP: A module should be responsible to one ,and only one, actor.

Now, what do we mean by the word "module" ? The simplest definition is just a source file. Most of the time that definition works fine. Some languages and development environment, though ,don't use source files to contain their code. In those cases a module is just a cohesive set of functions and data structures.

The word "cohesive" implies the SRP. Cohesion is the force that binds together the code responsible to a single actor.

# 8.OCP: The open-closed principle

OCP: A software artifact should be open for extension but closed for modification.

The OCP is one of the driving forces behind the architecture of systems. The goal is to make the system easy to extend without incurring a high impact of change. This goal is accomplished by partitioning the system into components, and arranging those components into a dependency hierarchy that protects higher-level components from changes in lower-level components.

# 9.LSP：The liskov substitution principle

LSP: What is wanted here is something like the following substitution property: If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T.

# 10.ISP: The interface segregation principle

The lesson here is that depending on something that carries baggage that you don't need can cause you troubles that you didn't expect.

# 11.DIP: The dependency inversion principle

The DIP tells us that the most flexible systems are those in which source code dependencies refer only to abstractions ,not to concretions. We tend to ignore the stable background of operating system and platform facilities when it comes to DIP. We tolerate those concrete dependencies because we know we can rely on them not to change.

It is the volatile concrete elements of our system that we want to avoid depending on . Those are the modules that we are actively developing ,and that are undergoing frequent change. Stable architectures are those that avoid depending on volatile concretions, and that favor the use of stable abstract interfaces. This implication boils down to a set of very specific coding practices:

Don't refer to volatile concrete classes, refer to abstract interfaces instead. It also puts severe constraints on the creation of objects and generally enforces the use of Abstract Factories. Don't derive from volatile concrete classes. Don't override concrete functions. Never mention the name of anything concrete and volatile.

![Use](https://i.postimg.cc/htx9dGBj/image.png)

Components principles：If the SOLID principles tell us how to arrange the bricks into walls and rooms, then the component principles tell us how to arrange the rooms into buildings.

# 12.Components

Components are the units of deployment. They are the smallest entities that can be deployed as part of a system.

Components can be linked together into a single executable.

# 13.Components cohesion

REP: The reuse/release equivalence principle ,the granule of reuse is the granule of release. For a software design and architecture point of view, this principle means that the classes and modules that are formed into a component must belong to a cohesive group. The component cannot simply consist of a random hodgepodge of classes and modules; instead ,there must be some overarching theme or purpose that those modules all share.

CCP:The common closure principle: Gather into components those classes that change for the same reasons and at same times. Separate into different components those classes that change at different times and for different reasons. The SRP tells us to separate methods into different classes, if they change for different reasons. The CCP tells us to separate classes into different components, if they change for different reasons.

CRP: The common reuse principle. Don't force users of a component to depend on things they don't need. The CRP is the generic version of the ISP. The ISP advises us not depend on classes that have methods we don't use. The CRP advises us not to depend on components that have classes we don't use.
