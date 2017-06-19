Effective Java

C H A P T E R 1:
Creating and Destroying Objects 

Item 1: Consider static factory methods instead of constructors

Advantages:
1.	Unlike constructors, they have names.
2.	Unlike constructors, they are not required to create a new object each time they’re invoked.
3.	Unlike constructors, they can return an object of any subtype of their return type.
4.	They reduce the verbosity of creating parameterized type instances.
Disadvantages:
1.	Classes without public or protected constructors cannot be sub-classed.
2.	They are not readily distinguishable from other static methods.

Item 2: Consider a builder when faced with many constructor parameters
The telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it. JavaBeans Pattern - allows inconsistency, mandates mutability. JavaBean may be in an inconsistent state partway through its construction. The JavaBeans pattern precludes the possibility of making a class immutable. The Builder pattern simulates named optional parameters.

Item 3: Enforce the singleton property with a private constructor or an enum type
A single-element enum type is the best way to implement a singleton.

public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ...}
}

This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks.

Item 4: Enforce noninstantiability with a private constructor
Utility classes were not designed to be instantiated: an instance would be nonsensical. Attempting to enforce noninstantiability by making a class abstract does not work. The class can be subclassed and the subclass instantiated. A default constructor is generated only if a class contains no explicit constructors, so a class can be made noninstantiable by including a private constructor:
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}
Item 5: Avoid creating unnecessary objects
It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is immutable

String s = new String("stringette"); // DON'T DO THIS!

The static factory method Boolean.valueOf(String) is almost always preferable to the constructor Boolean(String). The constructor creates a new object each time it’s called, while the static factory method is never required to do so and won’t in practice. There are other situations where it is less obvious. Consider the case of adapters, also known as views. An adapter is an object that delegates to a backing object, providing an alternative interface to the backing object. Because an adapter has no state beyond that of its backing object, there’s no need to create more than one instance of a given adapter to a given object.

Prefer primitives to boxed primitives, and watch out for unintentional autoboxing.

Item 6: Eliminate obsolete object references
Memory leaks in garbage-collected languages are insidious. If an object reference is unintentionally retained, not only is that object excluded from garbage collection, but so too are any objects referenced by that objects, and so on. Even if only a few object references are unintentionally retained, many, many objects may be prevented from being garbage collected, with potentially large effects on performance.
The fix for this sort of problem is simple: null out references once they become obsolete. Nulling out object references should be the exception rather than the norm. Whenever a class manages its own memory, the programmer should be alert for memory leaks. Whenever an element is freed, any object references contained in the element should be nulled out. Another common source of memory leaks is caches. A third common source of memory leaks is listeners and other callbacks.

Item 7: Avoid finalizers
Finalizers are unpredictable, often dangerous, and generally unnecessary. It is entirely possible, even likely, that a program terminates without executing finalizers on some objects that are no longer reachable. As a consequence, you should never depend on a finalizer to update critical persistent state.
There is a severe performance penalty for using finalizers. On my machine, the time to create and destroy a simple object is about 5.6 ns. Adding a finalizer increases the time to 2,400 ns. In other words, it is about 430 times slower to create and destroy objects with finalizers. So what should you do instead of writing a finalizer for a class whose objects encapsulate resources that require termination, such as files or threads?
Just provide an explicit termination method, and require clients of the class to invoke this method on each instance when it is no longer needed. Typical examples of explicit termination methods are the close methods on InputStream, OutputStream and java.sql.Connection.
Explicit termination methods are typically used in combination with the try-finally construct to ensure termination. Invoking the explicit termination method inside the finally clause ensures that it will get executed even if an exception is thrown while the object is being used:
// try-finally block guarantees execution of termination method
Foo foo = new Foo(...);
try {
// Do   what must be done with foo
} finally {
    foo.terminate(); // Explicit termination method
}
C H A P T E R 2:
Methods Common to All Objects

Item 8: Obey the general contract when overriding equals
When not to override the equals method?
1.	Each instance of the class is inherently unique. You don’t care whether the class provides a “logical equality” test.
2.	A superclass has already overridden equals, and the superclass behavior is appropriate for this class. For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.
3.	The class is private or package-private, and you are certain that its equals method will never be invoked.
So when it is appropriate to override Object equals method?
1.	When a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals to implement the desired behavior. This is generally the case for value classes.

When you override the equals method, you must adhere to its general contract. The equals method implements an equivalence relation if it is:
• Reflexive: For any non-null reference value x, x.equals(x) must return true.
• Symmetric: For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.
• Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.
• Consistent: For any non-null reference values x and y, multiple invocations of  x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.
• For any non-null reference value x, x.equals(null) must return false.

There are some classes in the Java platform libraries that do extend an instantiable class and add a value component. For example, java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed.
Note that you can add a value component to a subclass of an abstract class without violating the equals contract. Problems of the sort shown above won’t occur so long as it is impossible to create a superclass instance directly.

Item 9: Always override hashCode when you override equals
Contract hashCode:
• Whenever it is invoked on the same object more than once during an execution of an application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.
• If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.
• It is not required that if two objects are unequal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.
The key provision that is violated when you fail to override hashCode is the second one: equal objects must have equal hash codes.

@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + areaCode;
    result = 31 * result + prefix;
    result = 31 * result + lineNumber;
    return result;
}

// Lazily initialized, cached hashCode
private volatile int hashCode; // (See Item 71)
@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = 17;
        result = 31 * result + areaCode;
        result = 31 * result + prefix;
        result = 31 * result + lineNumber;
        hashCode = result;
    }
    return result;
}

Item 10: Always override toString
While java.lang.Object provides an implementation of the toString method, the string that it returns is generally not what the user of your class wants to see. It consists of the class name followed by an “at” sign (@) and the unsigned hexadecimal representation of the hash code, for example, “PhoneNumber@163b91.” The general contract for toString says that the returned string should be “a concise but informative representation that is easy for a person to read”.

Item 11: Override clone judiciously
If you override the clone method in a nonfinal class, you should return an object obtained by invoking super.clone. In practice, a class that implements Cloneable is expected to provide a properly functioning public clone method. It is not, in general, possible to do so unless all of the class’s superclasses provide a well-behaved clone implementation, whether public or protected.
In effect, the clone method functions as another constructor; you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone. In order for the clone method on Stack to work properly, it must copy the internals of the stack. The easiest way to do this is to call clone recursively on the elements array:

@Override 
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
Note also that the above solution would not work if the elements field were final, because clone would be prohibited from assigning a new value to the field. This is a fundamental problem: the clone architecture is incompatible with normal use of final fields referring to mutable objects, except in cases where the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove final modifiers from some fields.
Object’s clone method is declared to throw CloneNotSupportedException, but overriding clone methods can omit this declaration. Public clone methods should omit it because methods that don’t throw checked exceptions are easier to use.
A fine approach to object copying is to provide a copy constructor or copy factory. A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example,
public Yum(Yum yum);
A copy factory is the static factory analog of a copy constructor:
public static Yum newInstance(Yum yum);
The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.

Item 12: Override Consider implementing Comparable
	Contract compareTo:
1.	Compares this object with the specified object for order.
2.	Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object.
3.	Throws ClassCastException if the specified object’s type prevents it from being compared to this object.

Comparable interface is parameterized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointerException, and it will, as soon as the method attempts to access its members.

C H A P T E R 4:
Classes and Interfaces

Item 13: Minimize the accessibility of classes and members

A well-designed module hides all of its implementation details, cleanly separating its API from its implementation. Modules then communicate only through their APIs and are oblivious to each others’ inner workings. This concept, known as information hiding or encapsulation, is one of the fundamental tenets of software design
The rule of thumb is simple: make each class or member as inaccessible as possible. In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

Item 14: In public classes, use accessor methods, not public fields

If a class is accessible outside its package, provide accessor methods.
If a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields. 
In general public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable

Item 15: Minimize mutability

To make a class immutable, follow these five rules:
1.	Don’t provide any methods that modify the object’s state.
2.	Ensure that the class can’t be extended.
3.	Make all fields final.
4.	Make all fields private.
5.	Ensure exclusive access to any mutable components.

// Accessors with no corresponding mutators
public double realPart() {
    return re;
}
public double imaginaryPart() {
    return im;
}
public Complex add(Complex c) {
    return new Complex(re + c.re, im + c.im);
}

Immutable objects are inherently thread-safe; they require no synchronization. Therefore, immutable objects can be shared freely. The only real disadvantage of immutable classes is that they require a separate object for each distinct value. If a class cannot be made immutable, limit its mutability as much as possible. Therefore, make every field final unless there is a compelling reason to make it nonfinal.

Item 16: Favor composition over inheritance

It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension.
Unlike method invocation, inheritance violates encapsulation. Inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

Item 17: Design and document for inheritance or else prohibit it

Constructors must not invoke overridable methods, directly or indirectly. If you violate this rule, program failure will result. The superclass constructor runs before the subclass constructor, so the overriding method in the subclass will get invoked before the subclass constructor has run. If the overriding method depends on any initialization performed by the subclass constructor, the method will not behave as expected.
There are some situations where it is clearly the right thing to do, such as abstract classes, including skeletal implementations of interfaces. There are other situations where it is clearly the wrong thing to do, such as immutable classes.
Each time a change is made in such a class, there is a chance that client classes that extend the class will break. This is not just a theoretical problem. It is not uncommon to receive subclassing-related bug reports after modifying the internals of a nonfinal concrete class that was not designed and documented for inheritance.
The best solution to this problem is to prohibit subclassing in classes that are not designed and documented to be safely subclassed. There are two ways to prohibit subclassing. The easier of the two is to declare the class final. The alternative is to make all the constructors private or package-private and to add public static factories in place of the constructors.
You can eliminate a class’s self-use of overridable methods mechanically, without changing its behavior. Move the body of each overridable method to a private “helper method” and have each overridable method invoke its private helper method. Then replace each self-use of an overridable method with a direct invocation of the overridable method’s private helper method.

Item 18: Prefer interfaces to abstract classes
Interfaces are ideal for defining mixins.
1.	It allows the construction of nonhierarchical type frameworks.
2.	It enables safe, powerful functionality enhancements via the wrapper class idiom. 
You can combine the virtues of interfaces and abstract classes by providing an abstract skeletal implementation class to go with each nontrivial interface that you export.
Public interfaces, therefore, must be designed carefully. Once an interface is released and widely implemented, it is almost impossible to change.

Item 19: Use interfaces only to define types

The constant interface pattern is a poor use of interfaces. Interfaces should be used only to define types. They should not be used to export constants.

Item 20: Prefer class hierarchies to tagged classes

Tagged classes are verbose, error-prone, and inefficient. To transform a tagged class into a class hierarchy, first define an abstract class containing an abstract method for each method in the tagged class whose behavior depends on the tag value. If there are any methods whose behavior does not depend on the value of the tag, put them in this class. Similarly, if there are any data fields used by all the flavors, put them in this class. There are no such flavor-independent methods or fields in the Figure class.
Tagged classes are seldom appropriate. If you’re tempted to write a class with an explicit tag field, think about whether the tag could be eliminated and the class replaced by a hierarchy. When you encounter an existing class with a tag field, consider refactoring it into a hierarchy.

Item 21: Use function objects to represent strategies

A primary use of function pointers is to implement the Strategy pattern. To implement this pattern in Java, declare an interface to represent the strategy, and a class that implements this interface for each concrete strategy. When a concrete strategy is designed for repeated use, it is generally implemented as a private static member class and exported in a public static final field whose type is the strategy interface.

Item 22: Favor static member classes over nonstatic

There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes. One common use of a static member class is as a public helper class, useful only in conjunction with its outer class.
One common use of a nonstatic member class is to define an Adapter. If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of the member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

C H A P T E R 5:
Generics

Item 23: Don’t use raw types in new code

It is still legal to use collection types and other generic types without supplying type parameters, but you should not do it. If you use raw types, you lose all the safety and expressiveness benefits of generics As a consequence, you lose type safety if you use a raw type like List, but not if you use a parameterized type like List<Object>. Because generic type information is erased at runtime, it is illegal to use the instanceof operator on parameterized types other than unbounded wildcard types.
This is the preferred way to use the instanceof operator with generic types:
// Legitimate use of raw type - instanceof operator
if(o instanceof Set) { // Raw type
    Set<?> m = (Set<?>) o; // Wildcard type
}

Item 24: Eliminate unchecked warnings

Eliminate every unchecked warning that you can. If you eliminate all warnings, you are assured that your code is typesafe it means that you won’t get a ClassCastException at runtime. If you can’t eliminate a warning, and you can prove that the code that provoked the warning is typesafe, then suppress the warning with @SuppressWarnings("unchecked") annotation. Every time you use @SuppressWarnings("unchecked") annotation, add a comment saying why it’s safe to do so.

Item 25: Prefer lists to arrays

Arrays differ from generic types in two important ways. First, arrays are covariant. This scary-sounding word means simply that if Sub is a subtype of Super, then the array type Sub[] is a subtype of Super[]. Generics, by contrast, are invariant: for any two distinct types Type1 and Type2, List<Type1> is neither a subtype nor a supertype of List<Type2>
Types such as E, List<E>, and List<String> are technically known as non-referable types. Intuitively speaking, a non-referable type is one whose runtime representation contains less information than its compile-time representation.
The only parameterized types that are referable are unbounded wildcard types such as List<?> and Map<?,?>. It is legal, though infrequently useful, to create arrays of unbounded wildcard types. When you get a generic array creation error, the best solution is often to use the collection type List<E> in preference to the array type E[]. You might sacrifice some performance or conciseness, but in exchange you get better type safety
and interoperability. Arrays are covariant and reified; generics are invariant and erased. As a consequence, arrays provide runtime type safety but not compile-time type safety and vice versa for generics. Generally speaking, arrays and generics don’t mix well.

Item 26: Favor generic types
Generic types are safer and easier to use than types that require casts in client code. When you design new types, make sure that they can be used without such casts. This will often mean making the types generic. Generify your existing types as time permits.

Item 27: Favor generic methods

Writing generic methods is similar to writing generic types. The type parameter list, which declares the type parameter, goes between the method’s modifiers and its return type.
You can make the method more flexible by using bounded wildcard types.

// Using a recursive type bound to express mutual comparability
public static <T extends Comparable<T>> T max(List<T> list) {...}

The type bound <T extends Comparable<T>> may be read as “for every type T that can be compared to itself,” which corresponds more or less exactly to the notion of mutual comparability.

// Returns the maximum value in a list - uses recursive type bound
public static <T extends Comparable<T>> T max(List<T> list) {
    Iterator<T> i = list.iterator();
    T result = i.next();
    while (i.hasNext()) {
        T t = i.next();
        if (t.compareTo(result) > 0)
            result = t;
    }
    return result;
}

Item 28: Use bounded wildcards to increase API flexibility

If the element type of the Iterable src exactly matches that of the stack, it works fine. But suppose you have a Stack<Number> and you invoke push(intVal), where intVal is of type Integer. This works, because Integer is a subtype of Number The language provides a special kind of parameterized type call a bounded wildcard type to deal with situations like this. The type of the input parameter to pushAll should not be “Iterable of E” but “Iterable of some subtype of E,” and there is a wildcard type that means precisely that: Iterable<?
extends E>.
For maximum flexibility, use wildcard types on input parameters that represent producers or consumers. If an input parameter is both a producer and a consumer, then wildcard types will do you no good: you need an exact type match, which is what you get without any wildcards. Here is a mnemonic to help you remember which wildcard type to use: PECS stands for producer-extends, consumer-super. In other words, if a parameterized type represents a T producer, use <? extends T>; if it represents a T consumer, use <? super T>. In our Stack example, pushAll’s src parameter produces E instances for use by the Stack, so the appropriate type for src is Iterable<? extends E>; popAll’s dst parameter consumes E instances from the Stack, so the appropriate type for dst is Collection<? super E>.
Do not use wildcard types as return types. Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code.




Item 29: typesafe heterogeneous containers

The normal use of generics, exemplified by the collections APIs, restricts you to a fixed number of type parameters per container. You can get around this restriction by placing the type parameter on the key rather than the container. You can use Class objects as keys for such typesafe heterogeneous containers. A Class object used in this fashion is called a type token. You can also use a custom key type. For example, you could have a DatabaseRow type representing a database row (the container), and a generic type Column<T> as its key.

C H A P T E R 6:
Enums and Annotations

Item 30: Use enums instead of int constants

An enumerated type is a type whose legal values consist of a fixed set of constants. The basic idea behind Java’s enum types is simple: they are classes that export one instance for each enumeration constant via a public static final field. Enum types are effectively final, by virtue of having no accessible constructors. If switch statements on enums are not a good choice for implementing constant- specific behavior on enums, what are they good for? Switches on enums are good for augmenting external enum types with constant-specific behavior. So when should you use enums? Anytime you need a fixed set of constants. Of course, this includes “natural enumerated types,” such as the planets, the days of the week, and the chess pieces

Item 31: Use instance fields instead of ordinals

Never derive a value associated with an enum from its ordinal; store it in an instance field instead:

public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    private final int numberOfMusicians;
    Ensemble(int size) {
        this.numberOfMusicians = size;
    }
    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}

The Enum specification has this to say about ordinal: “Most programmers will have no use for this method. It is designed for use by general-purpose enumbased data structures such as EnumSet and EnumMap.” Unless you are writing such a data structure, you are best off avoiding the ordinal method entirely.


Item 32: Use EnumSet instead of bit fields

The java.util package provides the EnumSet class to efficiently represent sets of values drawn from a single enum type. This class implements the Set interface, providing all of the richness, type safety, and interoperability you get with any other Set implementation. But internally, each EnumSet is represented as a bit vector. If the underlying enum type has sixty-four or fewer elements—and most do—the entire EnumSet is represented with a single long, so its performance is comparable to that of a bit field. Bulk operations, such as removeAll and retainAll, are implemented using bitwise arithmetic, just as you’d do manually for bit fields. But you are insulated from the ugliness and error-proneness of manual bit twiddling: the EnumSet does the hard work for you. Just because an enumerated type will be used in sets, there is no reason to represent it with bit fields.

Item 33: Use EnumMap instead of ordinal indexing 

The reason that EnumMap is comparable in speed to an ordinal-indexed array is that EnumMap uses such an array internally. But it hides this implementation detail from the programmer, combining the richness and type safety of a Map with the speed of an array. Note that the EnumMap constructor takes the Class object of the key type: this is a bounded type token, which provides runtime generic type information it is rarely appropriate to use ordinals to index arrays: use EnumMap instead. If the relationship that you are representing is multidimensional use EnumMap<..., EnumMap<...>>.

Item 34: extensible enums with interfaces 

It is rarely appropriate to use ordinals to index arrays: use EnumMap instead. If the relationship that you are representing is multidimensional, use EnumMap<..., EnumMap<...>>. This is a special case of the general principle that application programmers should rarely, if ever, use Enum.ordinal.
The basic idea is to take advantage of the fact that enum types can implement arbitrary interfaces by defining an interface for the opcode type and an enum that is the standard implementation of the interface.

// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}


Now suppose you want to define an extension to the operation type above, consisting of the exponentiation and remainder operations. All you have to do is write an enum type that implements the Operation interface:

// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}

A minor disadvantage of the use of interfaces to emulate extensible enums is that implementations cannot be inherited from one enum type to another. In the case of our Operation example, the logic to store and retrieve the symbol associated with an operation is duplicated in BasicOperation and ExtendedOperation. In this case it doesn’t matter because very little code is duplicated. If there were a larger amount of shared functionality, you could encapsulate it in a helper class or a static helper method to eliminate the code duplication.

Item 35: extensible Prefer annotations to naming patterns 

There is simply no reason to use naming patterns now that we have annotations. All programmers should, however, use the predefined annotation types provided by the Java platform. Also, consider using any annotations provided by your IDE or static analysis tools. Such annotations can improve the quality of the diagnostic information provided by these tools. Note, however, that these annotations have yet to be standardized, so you will have some work to do if you switch tools, or if a standard emerges.

Item 36: Consistently use the Override annotation

Override annotation on every method declaration that you believe to override a superclass declaration. There is one minor exception to this rule. If you are writing a class that is not labeled abstract, and you believe that it overrides an abstract method, you needn’t bother putting the Override annotation on that method. In a class that is not declared abstract, the compiler will emit an error message if you fail to override an abstract superclass method. However, you might wish to draw attention to all of the methods in your class that override superclass methods, in which case you should feel free to annotate these methods too. The compiler can protect you from a great many errors if you use the Override annotation on every method declaration that you believe to override a supertype declaration, with one exception. In concrete classes, you need not annotate methods that you believe to override abstract method declarations (though it is not harmful to do so).


Item 37: Use marker interfaces to define types

Marker interfaces have two advantages over marker annotations. First and foremost, marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not. The existence of this type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation.
Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely. If an annotation type is declared with target ElementType. TYPE, it can be applied to any class or interface. Suppose you have a marker that is applicable only to implementations of a particular interface. If you define it as a marker interface, you can have it extend the sole interface to which it is applicable, guaranteeing that all marked types are also subtypes of the sole interface to which it is applicable.
The chief advantage of marker annotations over marker interfaces is that it is possible to add more information to an annotation type after it is already in use, by adding one or more annotation type elements with defaults marker interfaces and marker annotations both have their uses. If you want to define a type that does not have any new methods associated with it, a marker interface is the way to go. If you want to mark program elements other than classes and interfaces, to allow for the possibility of adding more information to the marker in the future, or to fit the marker into a framework that already makes heavy use of annotation types, then a marker annotation is the correct choice. If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type, or whether a marker interface would be more appropriate.


C H A P T E R 7:
Methods

Item 38: Check parameters for validity

Most methods and constructors have some restrictions on what values may be passed into their parameters. For example, it is not uncommon that index values must be non-negative and object references must be non-null. If an invalid parameter value is passed to a method and the method checks its parameters before execution, it will fail quickly and cleanly with an appropriate exception. If the method fails to check its parameters, several things could happen. The method could fail with a confusing exception in the midst of processing. Worse, the method could return normally but silently compute the wrong result.
Worst of all, the method could return normally but leave some object in a compromised state, causing an error at some unrelated point in the code at some undetermined time in the future. Each time you write a method or constructor, you should think about what restrictions exist on its parameters. You should document these restrictions and enforce them with explicit checks at the beginning of the method body. It is important to get into the habit of doing this. The modest work that it entails will be paid back with interest the first time a validity check fails.

Item 39: Make defensive copies when needed

Defensive copies are made before checking the validity of the parameters and the validity check is performed on the copies rather than on the originals. While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another thread during the “window of vulnerability” between the time the parameters are checked and the time they are copied. Do not use the clone method to make a defensive copy of a parameter whose type is subclassable by untrusted parties. 
If a class has mutable components that it gets from or returns to its clients, the class must defensively copy these components. If the cost of the copy would be prohibitive and the class trusts its clients not to modify the components inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components.

Item 40: Design method signatures carefully

Don’t go overboard in providing convenience methods. Avoid long parameter lists. Long sequences of identically typed parameters are especially harmful. There are three techniques for shortening overly long parameter lists.
1.	Break the method up into multiple methods, each of which requires only a subset of the parameters.
2.	A second technique for shortening long parameter lists is to create helper classes to hold groups of parameters. Typically these helper classes are static member classes.
3.	A third technique that combines aspects of the first two is to adapt the Builder pattern from object construction to method invocation.
.
For parameter types, favor interfaces over classes.
Prefer two-element enum types to Boolean parameters.

public enum TemperatureScale { FAHRENHEIT, CELSIUS }

Not only does Thermometer.newInstance(TemperatureScale.CELSIUS) make a lot more sense than Thermometer.newInstance(true), but you can add KELVIN to  emperatureScale in a future release without having to add a new static factory to  Thermometer.

Item 41: Use overloading judiciously

The choice of which overloading to invoke is made at compile time. The behavior of this program is counterintuitive because selection among an overloaded method is static, while selection among overridden methods is dynamic. The correct version of an overridden method is chosen at runtime.
A safe, conservative policy is never to export two overloading methods with the same number of parameters. If a method uses varargs, a conservative policy is not to overload it at all just because you can overload methods doesn’t mean you should. You should generally refrain from overloading methods with multiple signatures that have the same number of parameters.
In some cases, especially where constructors are involved, it may be impossible to follow this advice. In that case, you should at least avoid situations where the same set of parameters can be passed to different overloading methods by the addition of casts. If such a situation cannot be avoided, for example, because you are retrofitting an existing class to implement a new interface, you should ensure that all overloading methods behave identically when passed the same parameters. If you fail to do this, programmers will be hard pressed to make effective use of the overloaded method or constructor, and they won’t understand why it doesn’t work.








Item 42: Use varargs judiciously

// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}

This solution has several problems. The most serious is that if the client invokes this method with no arguments, it fails at runtime rather than compile time. Another problem is that it is ugly. You have to include an explicit validity check on args, and you can’t use a for-each loop unless you initialize min which is also ugly.
Varargs were designed for printf, which was added to the platform in release 1.5, and for the core reflection facility (Item 53), which was retrofitted to take advantage of varargs in that release. Both printf and reflection benefit enormously from varargs. Don’t retrofit every method that has a final array parameter; use varargs only when a call really operates on a variable-length sequence of values.
Exercise care when using the varargs facility in performance-critical situations. Every invocation of a varargs method causes an array allocation and initialization. If you have determined empirically that you can’t afford this cost but you need the flexibility of varargs, there is a pattern that lets you have your cake and eat it too. Like most performance optimizations, this technique usually isn’t appropriate, but when it is, it’s a lifesaver.

Item 43: Return empty arrays or collections, not nulls

This sort of circumlocution is required in nearly every use of a method that returns null in place of an empty (zero-length) array or collection. It is error prone, because the programmer writing the client might forget to write the special case code to handle a null return. It is sometimes argued that a null return value is preferable to an empty array because it avoids the expense of allocating the array. This argument fails on two counts. First, it is inadvisable to worry about performance at this level unless profiling has shown that the method in question is a real contributor to performance problems. Second, it is possible to return the same zero-length array.

Item 44: Write doc comments for all exposed API elements

To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment. The doc comment for a method should describe succinctly the contract between the method and its client.

/**
 * An object that maps keys to values. A map cannot contain
 * duplicate keys; each key can map to at most one value.
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> {
... // Remainder omitted
}
It is no longer necessary to use the HTML <code> or <tt> tags in doc comments: the Javadoc {@code} tag is preferable because it eliminates the need to escape HTML metacharacters. No two members or constructors in a class or interface should have the same summary description. When documenting a generic type or method, be sure to document all type parameters:
When documenting an enum type, be sure to document the constants as well as the type and any public methods. When documenting an annotation type, be sure to document any members as well as the type itself.

C H A P T E R 8:
General Programming 

Item 45: Consider Minimize the scope of local variables

By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error. The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.
Declaring a local variable prematurely can cause its scope not only to extend too early, but also to end too late. If a variable is declared outside of the block in which it is used, it remains visible after the program exits that block. If a variable is used accidentally before or after its region of intended use, the consequences can be disastrous.
Nearly every local variable declaration should contain an initializer. Prefer for loops to while loops, assuming the contents of the loop variable aren’t needed after the loop terminates. A final technique to minimize the scope of local variables is to keep methods small and focused.

Item 46: Consider Prefer for-each loops to traditional for loops

Prior to release 1.5, this was the preferred idiom for iterating over a collection:

// No longer the preferred idiom to iterate over a collection!
for (Iterator i = c.iterator(); i.hasNext(); ) {
    doSomething((Element) i.next()); // (No generics before 1.5)
}

This was the preferred idiom for iterating over an array:

// No longer the preferred idiom to iterate over an array!
for (int i = 0; i < a.length; i++) {
    doSomething(a[i]);
}

The for-each loop, introduced in release 1.5, gets rid of the clutter and the opportunity for error by hiding the iterator or index variable completely. The resulting idiom applies equally to collections and arrays: 

// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
    doSomething(e);
}

When you see the colon (:), read it as “in.” Thus, the loop above reads as “for each element e in elements.” the for-each loop provides compelling advantages over the traditional for loop in clarity and bug prevention, with no performance penalty. You should use it wherever you can. 
Unfortunately, there are three common situations where you can’t use a for-each loop:
1.	 Filtering—If you need to traverse a collection and remove selected elements, then you need to use an explicit iterator so that you can call its remove method.
2.	Transforming—If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to set the value of an element.
3.	Parallel iteration—If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable, so that all iterators or index variables can be advanced in lockstep (as demonstrated unintentionally in the buggy card and dice examples above).
If you find yourself in any of these situations, use an ordinary for loop, be wary of the traps mentioned in this item, and know that you’re doing the best you can.

Item 47: Know and use the libraries

No flaws have yet been found in the method, but if a flaw were to be discovered, it would be fixed in the next release. By using a standard library
1.	You take advantage of the knowledge of the experts who wrote it and the experience of those who used it before you.
2.	You don’t have to waste your time writing ad hoc solutions to problems that are only marginally related to your work. If you are like most programmers, you’d rather spend your time working on your application than on the underlying plumbing.
3.	Their performance tends to improve over time, with no effort on your part. Because many people use them and because they’re used in industry-standard benchmarks, the organizations that supply these libraries have a strong incentive to make them run faster. Many of the Java platform libraries have been rewritten over the years, sometimes repeatedly, resulting in dramatic performance improvements.

The libraries are too big to study all the documentation, but every programmer should be familiar with the contents of java.lang, java.util, and, to a lesser extent, java.io. Knowledge of other libraries can be acquired on an as-needed basis.

Item 48: Avoid float and double if exact answers are required

The float and double types are designed primarily for scientific and engineering calculations. They perform binary floating-point arithmetic, which was carefully designed to furnish accurate approximations quickly over a broad range of magnitudes. The float and double types are particularly illsuited for monetary calculations because it is impossible to represent 0.1 (or any other negative power of ten) as a float or double exactly. Use BigDecimal if you want the system to keep track of the decimal point and you don’t mind the inconvenience and cost of not using a primitive type.
Using BigDecimal has the added advantage that it gives you full control over rounding, letting you select from eight rounding modes whenever an operation that entails rounding is performed. This comes in handy if you’re performing business calculations with legally mandated rounding behavior.
If performance is of the essence, you don’t mind keeping track of the decimal point yourself, and the quantities aren’t too big, use int or long. If the quantities don’t exceed nine decimal digits, you can use int; if they don’t exceed eighteen digits, you can use long. If the quantities might exceed eighteen digits, you must use BigDecimal.



Item 49: Prefer primitive types to boxed primitives

There are three major differences between primitives and boxed primitives.
1.	First, primitives have only their values, whereas boxed primitives have identities distinct from their values. In other words, two boxed primitive instances can have the same value and different identities.
2.	Second, primitive types have only fully functional values, whereas each boxed primitive type has one nonfunctional value, which is null, in addition to all of the functional values of its corresponding primitive type.
3.	Last, primitives are generally more time- and space-efficient than boxed primitives.

All three of these differences can get you into real trouble if you aren’t careful. Applying the == operator to boxed primitives is almost always wrong. In nearly every case when you mix primitives and boxed primitives in a single operation, the boxed primitive is autounboxed, and this case is no exception. If a null object reference is auto-unboxed, you get a NullPointerException. 
So when should you use boxed primitives? They have several legitimate uses. The first is as elements, keys, and values in collections. You can’t put primitives in collections, so you’re forced to use boxed primitives. This is a special case of a more general one. You must use boxed primitives as type parameters in parameterized types (Chapter 5), because the language does not permit you to use primitives.

Item 50: strings where other types are more appropriate

Strings are poor substitutes for other value types.
Strings are poor substitutes for enum types.
Strings are poor substitutes for aggregate types
Strings are poor substitutes for capabilities.

Avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

Item 51: Beware the performance of string concatenation

Using the string concatenation operator repeatedly to concatenate n strings requires time quadratic in n. It is an unfortunate consequence of the fact that strings are immutable. When two strings are concatenated, the contents of both are copied.

// Inappropriate use of string concatenation - Performs horribly!
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++)
        result += lineForItem(i); // String concatenation
    return result;
}

Don’t use the string concatenation operator to combine more than a few strings unless performance is irrelevant. Use StringBuilder’s append method instead. Alternatively, use a character array, or process the strings one at a time instead of combining them.




Item 52: Refer to objects by their interfaces

If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types. The only time you really need to refer to an object’s class is when you’re creating it with a constructor.

// Good - uses interface as type 
List<Subscriber> subscribers = new Vector<Subscriber>();

rather than this:

// Bad - uses class as type!
Vector<Subscriber> subscribers = new Vector<Subscriber>();

If you get into the habit of using interfaces as types, your program will be much more flexible. It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists.

Item 53: Prefer interfaces to reflection 

The core reflection facility, java.lang.reflect, offers programmatic access to information about loaded classes. Reflection allows one class to use another, even if the latter class did not exist when the former was compiled. 
This power, however, comes at a price:
1.	 You lose all the benefits of compile-time type checking, including exception checking. If a program attempts to invoke a nonexistent or inaccessible method reflectively, it will fail at runtime unless you’ve taken special precautions.
2.	The code required to perform reflective access is clumsy and verbose. It is tedious to write and difficult to read.
3.	Performance suffers. Reflective method invocation is much slower than normal method invocation. Exactly how much slower is hard to say, because there are so many factors at work. On my machine, the speed difference can be as small as a factor of two or as large as a factor of fifty.

As a rule, objects should not be accessed reflectively in normal applications at runtime. There are a few sophisticated applications that require reflection. Examples include class browsers, object inspectors, code analysis tools, and interpretive embedded systems. Reflection is also appropriate for use in remote procedure call (RPC) systems to eliminate the need for stub compilers. If you have any doubts as to whether your application falls into one of these categories, it probably doesn’t.

Item 54: Use native methods judiciously

It is rarely advisable to use native methods for improved performance. The use of native methods has serious disadvantages. Because native languages are not safe, applications using native methods are no longer immune to memory corruption errors. Because native languages are platform dependent, applications using native methods are far less portable. Applications using native code are far more difficult to debug. There is a fixed cost associated with going into and out of native code, so native methods can decrease performance if they do only a small amount of work. Finally, native methods require “glue code” that is difficult to read and tedious to write.




Item 55: Optimize judiciously

Don’t sacrifice sound architectural principles for performance. Strive to write good programs rather than fast ones. Luckily, it is generally the case that good API design is consistent with good performance. It is a very bad idea to warp an API to achieve good performance. The performance issue that caused you to warp the API may go away in a future release of the platform or other underlying software, but the warped API and the support headaches that come with it will be with you for life. Once you’ve carefully designed your program and produced a clear, concise, and well-structured implementation, then it may be time to consider optimization, assuming you’re not already satisfied with the performance of the program. Do not strive to write fast programs—strive to write good ones; speed will follow. Do think about performance issues while you’re designing systems and especially while you’re designing APIs, wire-level protocols, and persistent data formats. 
When you’ve finished building the system, measure its performance. If it’s fast enough, you’re done. If not, locate the source of the problems with the aid of a profiler, and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of algorithm. Repeat this process as necessary, measuring the performance after every change, until you’re satisfied.

Item 56: Adhere to generally accepted naming conventions 

Internalize the standard naming conventions and learn to use them as second nature. The typographical conventions are straightforward and largely unambiguous; the grammatical conventions are more complex and looser. To quote from The Java Language Specification,”These conventions should not be followed slavishly if long-held conventional usage dictates otherwise.” Use common sense.


C H A P T E R 9:
Exceptions

Item 57: Use exceptions only for exceptional conditions

// Horrible abuse of exceptions. Don't ever do this!
try {
    int i = 0;
    while(true)
        range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e) {
}

The infinite loop terminates by throwing, catching, and ignoring an ArrayIndexOutOfBoundsException when it attempts to access the first array element outside the bounds of the array. Rather use

for (Mountain m : range)
    m.climb();

There are three things wrong with this reasoning:
1.	Because exceptions are designed for exceptional circumstances, there is little incentive for JVM implementors to make them as fast as explicit tests.
2.	Placing code inside a try-catch block inhibits certain optimizations that modern JVM implementations might otherwise perform.
3.	The standard idiom for looping through an array doesn’t necessarily result in redundant checks. Modern JVM implementations optimize them away.

Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow. A well-designed API must not force its clients to use exceptions for ordinary control flow.

Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

Use checked exceptions for conditions from which the caller can reasonably be expected to recover. By throwing a checked exception, you force the caller to handle the exception in a catch clause or to propagate it outward. Each checked exception that a method is declared to throw is therefore a potent indication to the API user that the associated condition is a possible outcome of invoking the method. By confronting the API user with a checked exception, the API designer presents a mandate to recover from the condition. The user can disregard the mandate by catching the exception and ignoring it, but this is usually a bad idea.
Use runtime exceptions to indicate programming errors. The great majority of runtime exceptions indicate precondition violations. While the Java Language Specification does not require it, there is a strong convention that errors are reserved for use by the JVM to indicate resource deficiencies, invariant failures, or other conditions that make it impossible to continue execution. Given the almost universal acceptance of this convention, it’s best not to implement any new Error subclasses. Therefore, all of the unchecked throwables you implement should subclass RuntimeException. Because checked exceptions generally indicate recoverable conditions, it’s especially important for such exceptions to provide methods that furnish information that could help the caller to recover. For example, suppose a checked exception is thrown when an attempt to make a purchase with a gift card fails because the card doesn’t have enough money left on it. The exception should provide an accessor method to query the amount of the shortfall, so the amount can be relayed to the shopper.

Item 59: Avoid unnecessary use of checked exceptions

Checked exceptions are a wonderful feature of the Java programming language. Unlike return codes, they force the programmer to deal with exceptional conditions, greatly enhancing reliability. If the programmer using the API can do no better, an unchecked exception would be more appropriate. The checked nature of the exception provides no benefit to the programmer, but it requires effort and complicates programs.

// Invocation with checked exception
try {
    obj.action(args);
} catch(TheCheckedException e) {
    // Handle exceptional condition     ...
}

// Invocation with state-testing method and unchecked exception
if (obj.actionPermitted(args)) {
    obj.action(args);
} else {
    // Handle exceptional condition     ...
}
This refactoring is not always appropriate, but where it is appropriate, it can make an API more pleasant to use. 

Item 60: Favor the use of standard exceptions

Reusing preexisting exceptions has several benefits. Chief among these, it makes your API easier to learn and use because it matches established conventions with which programmers are already familiar. A close second is that programs using your API are easier to read because they aren’t cluttered with unfamiliar exceptions. Last (and least), fewer exception classes mean a smaller memory footprint and less time spent loading classes.

This table summarizes the most commonly reused exceptions:

Exception	Occasion for Use
IllegalArgumentException	Non-null parameter value is inappropriate
IllegalStateException	Object state is inappropriate for method invocation
NullPointerException	Parameter value is null where prohibited
IndexOutOfBoundsException	Index parameter value is out of range
ConcurrentModificationException	Concurrent modification of an object has been detected where it is prohibited
UnsupportedOperationException	Object does not support method


Item 61: Throw exceptions appropriate to the abstraction

Higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction. This idiom is known as exception translation:

// Exception Translation
try {
// Use lower-level abstraction to do our bidding
...
} catch(LowerLevelException e) {
    throw new HigherLevelException(...);
}

A special form of exception translation called exception chaining is appropriate in cases where the lower-level exception might be helpful to someone debugging the problem that caused the higher-level exception. The lower-level exception (the cause) is passed to the higher-level exception, which provides an accessor method (Throwable.getCause) to retrieve the lower-level exception:

// Exception Chaining
try {
... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}

The higher-level exception’s constructor passes the cause to a chaining-aware superclass constructor, so it is ultimately passed to one of Throwable’s chainingaware constructors, such as Throwable:

// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}

While exception translation is superior to mindless propagation of exceptions from lower layers, it should not be overused. Under these circumstances, it may be appropriate to log the exception using some appropriate logging facility such as java.util.logging. This allows an administrator to investigate the problem.

Item 62: Document all exceptions thrown by each method

Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag. Unchecked exceptions generally represent programming errors (Item 58), and familiarizing programmers with all of the errors they can make helps them avoid making these errors. Use the Javadoc @throws tag to document each unchecked exception that a method can throw, but do not use the throws keyword to include unchecked exceptions in the method declaration. If an exception is thrown by many methods in a class for the same reason, it is acceptable to document the exception in the class’s documentation comment rather than documenting it individually for each method.  A common example is NullPointerException. It is fine for a class’s documentation comment to say, “All methods in this class throw a  NullPointerException if a null object reference is passed in any parameter,” or words to that effect.

Item 63: Include failure-capture information in detail messages

To capture the failure, the detail message of an exception should contain the values of all parameters and fields that “contributed to the exception.” For example, the detail message of an IndexOutOfBoundsException should contain the lower bound, the upper bound, and the index value that failed to lie between the bounds. One way to ensure that exceptions contain adequate failure-capture information in their detail messages is to require this information in their constructors instead of a string detail message. It is rare (although not inconceivable) that a programmer might want programmatic access to the details of an unchecked exception. Even for unchecked exceptions, however, it seems advisable to provide these accessors on general principle.

Item 64: Strive for failure atomicity

Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation. A method with this property is said to be failure atomic.
There are several ways to achieve this effect. The simplest is to design immutable objects. If an object is immutable, failure atomicity is free. If an operation fails, it may prevent a new object from getting created, but it will never leave an existing object in an inconsistent state, because the state of each object is consistent when it is created and can’t be modified thereafter. For methods that operate on mutable objects, the most common way to achieve failure atomicity is to check parameters for validity before performing the operation. This causes any exception to get thrown before object modification commences.

public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}

A third and far less common approach to achieving failure atomicity is to write recovery code that intercepts a failure that occurs in the midst of an operation and causes the object to roll back its state to the point before the operation began. This approach is used mainly for durable (disk-based) data structures. final approach to achieving failure atomicity is to perform the operation on a temporary copy of the object and to replace the contents of the object with the temporary copy once the operation is complete As a rule, any generated exception that is part of a method’s specification should leave the object in the same state it was in prior to the method invocation. Where this rule is violated, the API documentation should clearly indicate what state the object will be left in.

Item 64: Don’t ignore (swallow) exceptions

An empty catch block defeats the purpose of exceptions, which is to force you to handle exceptional conditions. Whenever you see an empty catch block, alarm bells should go off in your head. At the very least, the catch block should contain a comment explaining why it is appropriate to ignore the exception. The advice in this item applies equally to checked and unchecked exceptions. Whether an exception represents a predictable exceptional condition or a programming error, ignoring it with an empty catch block will result in a program that continues silently in the face of error. The program might then fail at an arbitrary time in the future, at a point in the code that bears no apparent relation to the source of the problem.


C H A P T E R 10:
Concurrency

Item 66: Synchronize access to shared mutable data

You may hear it said that to improve performance, you should avoid synchronization when reading or writing atomic data. This advice is dangerously wrong. Synchronization is required for reliable communication between threads as well as for mutual exclusion.
It is not sufficient to synchronize only the write method! In fact, synchronization has no effect unless both read and write operations are synchronized. Either share immutable data  or don’t share at all. In other words, confine mutable data to a single thread. When multiple threads share mutable data, each thread that reads or writes the data must perform synchronization. Without synchronization, there is no guarantee that one thread’s changes will be visible to another. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the volatile modifier is an acceptable form of synchronization, but it can be tricky to use correctly.

Item 67: Avoid excessive synchronization

Inside a synchronized region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object (Item 21). From the perspective of the class with the synchronized region, such methods are alien. If you do synchronize your class internally, you can use various techniques to achieve high concurrency, such as lock splitting, lock striping, and nonblocking concurrency control.
If you do synchronize your class internally, you can use various techniques to achieve high concurrency, such as lock splitting, lock striping, and nonblocking concurrency control.

Item 68: Prefer executors and tasks to threads

ExecutorService executor = Executors.newSingleThreadExecutor();

Here is how to submit a runnable for execution:

executor.execute(runnable);

And here is how to tell the executor to terminate gracefully (if you fail to do this, it is likely that your VM will not exit):

executor.shutdown();

Choosing the executor service for a particular application can be tricky. If you’re writing a small program, or a lightly loaded server, using Executors.new- CachedThreadPool is generally a good choice, as it demands no configuration and generally “does the right thing.” But a cached thread pool is not a good choice for a heavily loaded production server! In a cached thread pool, submitted tasks are not queued but immediately handed off to a thread for execution. If no threads are available, a new one is created. If a server is so heavily loaded that all of its CPUs are fully utilized, and more tasks arrive, more threads will be created, which will only make matters worse. Therefore, in a heavily loaded production server, you are much better off using Executors.newFixedThreadPool, which gives you a pool with a fixed number of threads, or using the ThreadPoolExecutor class directly, for maximum control.

Item 69: Prefer concurrency utilities to wait and notify

Given the difficulty of using wait and notify correctly, you should use the higher-level concurrency utilities instead. The higher-level utilities in java.util.concurrent fall into three categories: the Executor Framework, concurrent collections; and synchronizers. Use ConcurrentHashMap in preference to Collections.synchronizedMap or Hashtable. BlockingQueue extends Queue and adds several methods, including take, which removes and returns the head element from the queue, waiting if the queue is empty. This allows blocking queues to be used for work queues (also known as producer-consumer queues),most ExecutorService implementations, including ThreadPoolExecutor, use a BlockingQueue.

// Simple framework for timing concurrent execution
public static long time(Executor executor, int concurrency,
                        final Runnable action) throws InterruptedException {
    final CountDownLatch ready = new CountDownLatch(concurrency);
    final CountDownLatch start = new CountDownLatch(1);
    final CountDownLatch done = new CountDownLatch(concurrency);
    for (int i = 0; i < concurrency; i++) {
        executor.execute(new Runnable() {
            public void run() {
                ready.countDown(); // Tell timer we're ready
                try {
                    start.await(); // Wait till peers are ready
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown(); // Tell timer we're done
                }
            }
        });
    }
    ready.await(); // Wait for all workers to be ready
    long startNanos = System.nanoTime();
    start.countDown(); // And they're off!
    done.await(); // Wait for all workers to finish
    return System.nanoTime() - startNanos;
}


Item 70: Document thread safety

In fact, there are several levels of thread safety. To enable safe concurrent use, a class must clearly document what level of thread safety it supports.
The following list summarizes levels of thread safety.
1.	Immutable—Instances of this class appear constant. No external synchronization is necessary. Examples include String, Long, and BigInteger.
2.	Unconditionally thread-safe—Instances of this class are mutable, but the class has sufficient internal synchronization that its instances can be used concurrently without the need for any external synchronization. Examples include Random and ConcurrentHashMap.
3.	Conditionally thread-safe—Like unconditionally thread-safe, except that some methods require external synchronization for safe concurrent use. Examples include the collections returned by the Collections.synchronized wrappers, whose iterators require external synchronization.
4.	Not thread-safe—Instances of this class are mutable. To use them concurrently, clients must surround each method invocation (or invocation sequence) with external synchronization of the clients’ choosing. Examples include the general-purpose collection implementations, such as ArrayList and HashMap.
5.	Thread-hostile—This class is not safe for concurrent use even if all method invocations are surrounded by external synchronization

If you write an unconditionally thread-safe class, consider using a private lock object in place of synchronized methods. This protects you against synchronization interference by clients and subclasses and gives you the flexibility to adopt a more sophisticated approach to concurrency control in a later release.

Item 71: Use lazy initialization judiciously

Lazy initialization is the act of delaying the initialization of a field until its value is needed. If the value is never needed, the field is never initialized. While lazy initialization
is primarily an optimization, it can also be used to break harmful circularities in class and instance initialization  In the presence of multiple threads, lazy initialization is tricky. If two or more threads share a lazily initialized field, it is critical that some form of synchronization be employed, or severe bugs can result.
Under most circumstances, normal initialization is preferable to lazy initialization. Here is a typical declaration for a normally initialized instance field. Note the use of the final modifier 
// Normal initialization of an instance field
private final FieldType field = computeFieldValue();
If you use lazy initialization to break an initialization circularity, use a synchronized accessor, as it is the simplest, clearest alternative:

// Lazy initialization of instance field - synchronized accessor
private FieldType field;
synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}

If you need to use lazy initialization for performance on a static field, use the lazy initialization holder class idiom
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
static FieldType getField() {
    return FieldHolder.field;
}

When the getField method is invoked for the first time, it reads Field- Holder.field for the first time, causing the FieldHolder class to get initialized. The beauty of this idiom is that the getField method is not synchronized and performs only a field access, so lazy initialization adds practically nothing to the cost of access. If you need to use lazy initialization for performance on an instance field, use the double-check idiom.

// Double-check idiom for lazy initialization of instance fields
private volatile FieldType field;
FieldType getField() {
    FieldType result = field;
    if (result == null) { // First check (no locking)
        synchronized(this) {
            result = field;
            if (result == null) // Second check (with locking)
                field = result = computeFieldValue();
        }
    }
    return result;
}

When the double-check or single-check idiom is applied to a numerical primitive field, the field’s value is checked against 0 rather than null.

Item 72: Don’t depend on the thread scheduler

Any program that relies on the thread scheduler for correctness or performance is likely to be nonportable The best way to write a robust, responsive, portable program is to ensure that the average number of runnable threads is not significantly greater than the number of processors.Note that the number of runnable threads isn’t the same as the total number of threads, which can be much higher. Thread priorities are among the least portable features of the Java platform. do not depend on the thread scheduler for the correctness of your program. The resulting program will be neither robust nor portable. As a corollary, do not rely on Thread.yield or thread priorities. These facilities are merely hints to the scheduler.

Item 73: Avoid thread groups

Along with threads, locks, and monitors, a basic abstraction offered by the threading system is thread groups. Thread groups don’t provide much in the way of useful functionality, and much of the functionality they do provide is flawed. Thread groups are best viewed as an unsuccessful experiment, and you should simply ignore their existence. If you design a class that deals with logical groups of threads, you should probably use thread pool executors

C H A P T E R 11:
Serialization

Encoding an object as a byte stream is known as serializing the object; the reverse process is known as deserializing it. Once an object has been serialized, its encoding can be transmitted from one running virtual machine to another or stored on disk for later deserialization.

Item 74: Implement Serializable judiciously

A major cost of implementing Serializable is that it decreases the flexibility to change a class’s implementation once it has been released. If you accept the default serialized form, the class’s private and package-private instance fields become part of its exported API, and the practice of minimizing access to fields loses its effectiveness as a tool for information hiding. 
Every serializable class has a unique identification number associated with it. If you do not specify this number explicitly by declaring a static final long field named serialVersionUID, the system automatically generates it at runtime by applying a complex procedure to the class. The automatically generated value is affected by the class’s name, the names of the interfaces it implements, and all of its public and protected members. If you change any of these things in any way, for example, by adding a trivial convenience method, the automatically generated serial version UID changes. If you fail to declare an explicit serial version UID, compatibility will be broken, resulting in an InvalidClassException at runtime. A second cost of implementing Serializable is that it increases the likelihood of bugs and security holes.
A third cost of implementing Serializable is that it increases the testing burden associated with releasing a new version of a class.The greater the change to a serializable class, the greater the need for testing.
Implementing the Serializable interface is not a decision to be undertaken lightly. It offers real benefits. It is essential if a class is to participate in a framework that relies on serialization for object transmission or persistence. There are, however, many real costs associated with implementing Serializable
As a rule of thumb, value classes such as Date and BigInteger should implement Serializable, as should most collection classes. Classes representing active entities, such as thread pools, should rarely implement Serializable. Classes designed for inheritance should rarely implement Serializable, and interfaces should rarely extend it. Classes designed for inheritance that do implement Serializable include Throwable, Component, and HttpServlet. Throwable implements Serializable so exceptions from remote method invocation (RMI) can be passed from server to client. Component implements Serializable so GUIs can be sent, saved, and restored. HttpServlet implements Serializable so session state can be cached. Inner classes should not implement Serializable.



Item 75: Consider using a custom serialized form

Do not accept the default serialized form without first considering whether it is appropriate. The default serialized form is likely to be appropriate if an object’s physical representation is identical to its logical content. For example, the default serialized form would be reasonable for the following class, which simplistically represents a person’s name:

// Good candidate for default serialized form
public class Name implements Serializable {
    /**
     * Last name. Must be non-null.
     * @serial
     */
    private final String lastName;
    /**
     * First name. Must be non-null.
     * @serial
     */
    private final String firstName;
Even if you decide that the default serialized form is appropriate, you often must provide a readObject method to ensure invariants and security. In the case of Name, the readObject method must ensure that lastName and firstName are non-null.
Using the default serialized form when an object’s physical representation differs substantially from its logical data content has four disadvantages:
1.	It permanently ties the exported API to the current internal representation.
2.	It can consume excessive space
3.	It can consume excessive time.
4.	It can cause stack overflows.

If all instance fields are transient, it is technically permissible to dispense with invoking defaultWriteObject and defaultReadObject, but it is not recommended. Before deciding to make a field nontransient, convince yourself that its value is part of the logical state of the object.
If you are using the default serialized form and you have labeled one or more fields transient, remember that these fields will be initialized to their default values when an instance is deserialized: null for object reference fields, zero for numeric primitive fields, and false for boolean fields. If these values are unacceptable for any transient fields, you must provide a readObject method that invokes the defaultReadObject method and then restores transient fields to acceptable values. Alternatively, these fields can be lazily initialized the first time they are used Regardless of what serialized form you choose, declare an explicit serial version UID in every serializable class you write. This eliminates the serial version UID as a potential source of incompatibility

Item 76: Write readObject methods defensively

readObject method is effectively another public constructor, and it demands all of the same care as any other constructor When an object is deserialized, it is critical to  defensively copy any field containing an object reference that a client must not possess.
Therefore, every serializable immutable class containing private mutable components must defensively copy these components in its readObject method.
Do not use the writeUnshared and readUnshared methods. They are typically faster than defensive copying, but they don’t provide the necessary safety guarantee.


The guidelines for writing a bulletproof readObject method:
1.	For classes with object reference fields that must remain private, defensively copy each object in such a field. Mutable components of immutable classes fall into this category.
2.	Check any invariants and throw an InvalidObjectException if a check fails. The checks should follow any defensive copying.
3.	If an entire object graph must be validated after it is deserialized, use the ObjectInputValidation interface.
4.	Do not invoke any overridable methods in the class, directly or indirectly.

Item 77: For instance control, prefer enum types to readResolve

If you depend on readResolve for instance control, all instance fields with object reference types must be declared transient.

// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
            { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}

The use of readResolve for instance control is not obsolete. If you have to write a serializable instance-controlled class whose instances are not known at compile time, you will not be able to represent the class as an enum type. You should use enum types to enforce instance control invariants wherever possible. If this is not possible and you need a class to be both serializable and instance-controlled, you must provide a readResolve method and ensure that all of the class’s instance fields are either primitive or transient.

Item 78: Consider serialization proxies instead of serialized instances

The decision to implement Serializable increases the likelihood of bugs and security problems, because it causes instances to be created using an extralinguistic mechanism in place of ordinary constructors. There is, however, a technique that greatly reduces these risks. This technique is known as the serialization proxy pattern. The serialization proxy pattern has two limitations. It is not compatible with classes that are extendable by their clients (Item 17). Also, it is not compatible with some classes whose object graphs contain circularities: if you attempt to invoke a method on an object from within its serialization proxy’s readResolve method, you’ll get a ClassCastException, as you don’t have the object yet, only its serialization proxy.
Finally, the added power and safety of the serialization proxy pattern are not free it is expensive than it is with defensive copying consider the serialization proxy pattern whenever you find yourself having to write a readObject or writeObject method on a class that is not extendable by its clients. This pattern is perhaps the easiest way to robustly serialize objects with nontrivial invariants.
