---
layout: reference
title: Calling Ceylon from Java
tab: documentation
unique_id: docspage
milestone: Milestone 2
author: Tom Bentley
---

# #{page.title}

## Description

Although titled '*Calling* Ceylon from Java', this page covers all access to 
Ceylon types from Java, not just method calls.

**Important Note: Everything documented here is subject to change until 
Ceylon 1.0 and has changed in M3.**

### Declaring Ceylon types

In general, a top level Ceylon types and inner classes compile into a Java 
type of the same name.

The situation is more complicated for inner interfaces. The compiler always 
generates a top level interface, using the containing type name(s) separated with a dollar (`$`) and then the interface name, again separated with a dollar. For 
example this Ceylon:

    class C() {
        interface I {
        }
    }
    
results in an interface like this:

<!-- lang: java -->
    interface C$I {
    }

This is done in order to support arbitrary nesting of interfaces and classes 
within other types, which cannot be expressed in Java.

### Instantiating Ceylon classes

Ceylon types are instantiated like every Java type. They have an overloaded 
constructor for each defaulted parameter.

For example suppose you have the following Ceylon class:

    shared class Foo(Integer n = 5, Integer m = n + 1) {
    }

Then in Java you can instantiate a `Foo` in three ways:

<!-- lang: java -->
    Foo foo1 = new Foo(6, 7);
    Foo foo2 = new Foo(6); // use the default m
    Foo foo3 = new Foo();  // use the default n and m

### Accessing Ceylon attributes

If the Ceylon attribute
is annotated `shared` the Java accessor will be declared `public`, otherwise
it will be annotated `private`.

#### Instance attributes

In general a Ceylon instance attribute compiles into a Java Bean-style getter and 
(if the attribute is `variable` or has an `assign` block) setter. 

`Boolean` attributes use `get` accessors rather than an `is` accessors. 
Note that a `get` accessor is still valid for `boolean` properties 
according to the Java Bean specification.

#### Toplevel attributes

A toplevel attribute is compiled into a class of the same name with an underscore (`_`)
suffix, which is `final`,
has a `private` constructor, a `static` getter method and if the attribute is mutable, a
corresponding `static` setter method.

Toplevel attributes like the following:

    shared variable Boolean bool := true;

Are accessed and set using the following Java code:

<!-- lang: java -->
    // bool is the name of the Java class that holds the toplevel attribute 
    boolean value = bool_.getBool$();
    bool_.setBool$(false);

### Calling Ceylon methods

In general a Ceylon method compiles directly into a Java method of the same 
name. The method result type and argument types may not however 
translate directly because of the [type mapping rules](../type-mapping).

If a Ceylon method name is a Java keyword, it will be prefixed with `$`, like `$true`.

#### Calling Ceylon toplevel methods

A Ceylon toplevel method is compiled into a class of the same name with an underscore (`_`)
suffix, which is `final`,
has a `private` constructor, and a `static` method of the same name corresponding to the 
toplevel method.

Toplevel methods like the following:

    shared Boolean foo(Boolean b){
        return b;
    }

Are called using the following Java code:

<!-- lang: java -->
    // foo is the name of the Java class that holds the toplevel method 
    boolean value = foo_.foo(false);

#### Calling a Ceylon method with default parameter values

Ceylon methods can have default parameter values. There will be an overloaded method for each defaulted parameter. 

For example suppose you have the following Ceylon class:

    shared class Foo() {
        shared void foo(Integer n = 5, Integer m = n + 1) {}
    }

Then in Java you can invoke `Foo.foo()` in three ways:

<!-- lang: java -->
    Foo f = new Foo();
    f.foo(6, 88);
    f.foo(6); // use the default m
    f.foo();  // use the default n and m

### Methods and initialisers with varargs

Method and initialisers can have varargs, which are represented in the resulting Java
code as a `Iterable<? extends T>` parameter. There will be an overloaded 
method if you want to use the sequenced parameter's default value (an empty 
`Iterable` by default)

If you wish to invoke a Ceylon method or initialiser that supports varargs you need to
use the following idiom:

<!-- lang: java -->
    // Foo.foo supports varargs of String...
    Foo f = new Foo();
    // use the parameter's default sequence
    f.foo();
    // pass no arguments
    f.foo(ceylon.language.$empty.getEmpty());
    // pass two arguments
    f.foo(new ceylon.language.ArraySequence(ceylon.language.String.instance("a"), 
                                            ceylon.language.String.instance("b")));


### Catching Ceylon exceptions

The root of the exception hierarchy in Ceylon is `ceylon.language.Exception`, 
(a subclass of `java.lang.RuntimeException` at runtime). This means that pure Ceylon code can only
generate unchecked exceptions.

Impure Ceylon (that is, Ceylon code which access Java code) may throw 
*any* exception that is thrown by that Java code, including checked exceptions. 
Ceylon methods do not declare `throws java.lang.Throwable` in their bytecode, 
even though they can in principle throw `Throwable`. In practice 
Ceylon methods are very unlikely to throw `Throwable` itself 
(because Java methods are unlikely to), but Ceylon methods are quite likely 
to throw `java.lang.Exception`. So unless you know otherwise, it's 
probably sensible to wrap calls to Ceylon methods in a Java-side 
`try/catch` which handles this possibility.

### Type conversions

If you need to convert a Java or Ceylon type according to the 
[type mapping rules](../type-mapping), you can do so using the following
methods.

#### Convert from a Java type to a Ceylon type

<table>
    <tr>
        <th>Java type</th>
        <th>Convert to Ceylon object</th>
    </tr>
    <tr>
        <td><code>boolean b</code></td>
        <td><code>ceylon.language.Boolean.instance(b)</code></td>
    </tr>
    <tr>
        <td><code>byte x</code>, <code>short x</code>, <code>int x</code>, <code>long x</code></td>
        <td><code>ceylon.language.Integer.instance(x)</code></td>
    </tr>
    <tr>
        <td><code>float x</code>, <code>double x</code></td>
        <td><code>ceylon.language.Float.instance(x)</code></td>
    </tr>
    <tr>
        <td><code>char c</code></td>
        <td><code>ceylon.language.Character.instance(c)</code></td>
    </tr>
    <tr>
        <td><code>T[] a</code></td>
        <td><code>ceylon.language.Array.instance(a)</code></td>
    </tr>
    <tr>
        <td><code>java.lang.String s</code></td>
        <td><code>ceylon.language.String.instance(s)</code></td>
    </tr>
</table>

#### Convert from a Ceylon type to a Java type

<table>
    <tr>
        <th>Ceylon type</th>
        <th>Convert to Java object</th>
    </tr>
    <tr>
        <td><code>ceylon.language.Boolean b</code></td>
        <td><code>b.booleanValue()</code></td>
    </tr>
    <tr>
        <td><code>ceylon.language.Integer i</code></td>
        <td><code>i.longValue()</code></td>
    </tr>
    <tr>
        <td><code>ceylon.language.Float f</code></td>
        <td><code>f.doubleValue()</code></td>
    </tr>
    <tr>
        <td><code>ceylon.language.Character c</code></td>
        <td><code>c.intValue()</code></td>
    </tr>
    <tr>
        <td><code>ceylon.language.Array a</code></td>
        <td><code>a.toArray()</code></td>
    </tr>
    <tr>
        <td><code>ceylon.language.String s</code></td>
        <td><code>s.toString()</code></td>
    </tr>
</table>

## See also

* [Calling Java from Ceylon](../java-from-ceylon)
* The [type mapping rules](../type-mapping)

