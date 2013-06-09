---
title: Changelog
blog: equalsverifier
layout: equalsverifier
---
What's new? Well, you can now ...

Version 1.2
-----------
_March 26, 2013_

* ...get a transitivity error message for classes that use `or` in their equals method. ([Issue 75](https://code.google.com/p/equalsverifier/issues/detail?id=75); blog post about [the problem](http://www.jqno.nl/post/2013/02/17/reaction-to-cedric-beusts-equals-challenge/) and [the solution](http://www.jqno.nl/post/2013/03/26/on-transitivity))
* ...expect more robustness from `EqualsVerifier` when the `toString` method of the class under tests throws exceptions.
* ...use single value enums (and hence, singletons) in your classes, even if they don't count for `equals` and `hashCode`. ([Issue 74](https://code.google.com/p/equalsverifier/issues/detail?id=74))
* ...use `forRelaxedEqualExamples` with classes containing null fields. ([Issue 73](https://code.google.com/p/equalsverifier/issues/detail?id=73))
* ...run `EqualsVerifier`'s test suite with less chance of false negatives. ([Issue 77](https://code.google.com/p/equalsverifier/issues/detail?id=77))
* ...build `EqualsVerifier` using Ant 1.9. ([Issue 76](https://code.google.com/p/equalsverifier/issues/detail?id=76))

Version 1.1.4
-----------
_January 14, 2013_

* ... fork `EqualsVerifier` on !GitHub! `EqualsVerifier` has moved its code to !GitHub. The project frontpage will remain at Google Code.
* ... use `EqualsVerifier` on a class whose superclasses have no declarations for `equals` and `hashCode`. ([Issue 63](https://code.google.com/p/equalsverifier/issues/detail?id=63))
* ... get an error when using `#allFieldsShouldBeUsed()` on a class that has fields but no declarations for `equals` and `hashCode` (and hence doesn't use these fields). ([Issue 67](https://code.google.com/p/equalsverifier/issues/detail?id=67))
* ... get a more informative error message on Abstract Delegation errors: `EqualsVerifier` now also mentions the name of the abstract method that was called.

Version 1.1.3
-------------
_April 21, 2012_

* ... add static fields to your classes that are tested with `#allFieldsShouldBeUsed()`. `EqualsVerifier` will no longer complain if these static fields aren't used in your `equals` and `hashCode` methods. ([Issue 57](https://code.google.com/p/equalsverifier/issues/detail?id=57))
* ... get a complaint from `EqualsVerifier` when your `equals` method works incorrectly if a field in the class under test is null. ([Issue 59](https://code.google.com/p/equalsverifier/issues/detail?id=59))

Version 1.1.2
-------------
_March 1, 2012_

* ... stop worrying about `EqualsVerifier` _recursively_ changing the value of non-final static fields, causing other tests to fail. (Issue 55, [http://code.google.com/p/equalsverifier/issues/detail?id=55#c8 Comment 8])

Version 1.1.1
-------------
_February 21, 2012_

* ... suppress `Warning.IDENTICAL_COPY` if you want to use reference equality in an overridden `equals` method. ([Issue 56](https://code.google.com/p/equalsverifier/issues/detail?id=56))

Version 1.1
-----------
_February 11, 2012_

* ... get a warning if you forgot to include a field in your `equals` or `hashCode` method, by calling the `#allFieldsShouldBeUsed()` method on `EqualsVerifier`. ([Issue 53](https://code.google.com/p/equalsverifier/issues/detail?id=53))
* ... get an error message if you include a reference-equality check (`this == obj`), followed by an incorrect instanceof check in your `equals` method. `EqualsVerifier` will now notice if you do an instanceof check for a class other than the one that you are testing. (Issue 55, [http://code.google.com/p/equalsverifier/issues/detail?id=55#c2 Comment 2])
* ... stop worrying about `EqualsVerifier` changing the value of non-final static fields, causing other tests to fail. (Issue 52 and Issue 55)
* ... stop worrying about adding prefab values for `File`, `List`, `Set`, `Map` and `Collection` to avoid unjustified warning. (Issue 49 and Issue 51)

Version 1.0.2
-------------
_August 14, 2011_

* ... find `EqualsVerifier` in Maven Central! ([Issue 36](https://code.google.com/p/equalsverifier/issues/detail?id=36))
* ... get an error message when you forget to include an `instanceof` check or a `getClass()` check in your `equals` method. ([Issue 47](https://code.google.com/p/equalsverifier/issues/detail?id=47))
* ... use `EqualsVerifier` on a class whose superclass has abstract declarations for `equals` and `hashCode`. ([Issue 48](https://code.google.com/p/equalsverifier/issues/detail?id=48))

Version 1.0.1
-------------
_April 17, 2011_

* ... use `EqualsVerifier` on dynamically generated classes by disabling annotation processing. ([Issue 41](https://code.google.com/p/equalsverifier/issues/detail?id=41))
* ... use `EqualsVerifier` on classes that are a subclass of one of the classes from `rt.jar`, which are loaded by the system classloader. ([Issue 43](https://code.google.com/p/equalsverifier/issues/detail?id=43))
* ... use `EqualsVerifier` on classes with `float` or `double` fields but no `equals` method. ([Issue 44](https://code.google.com/p/equalsverifier/issues/detail?id=44))
* ... stop worrying about adding prefab values for `Throwable` and `Exception`, to avoid that pesky recursion warning. ([Issue 45](https://code.google.com/p/equalsverifier/issues/detail?id=45))

Version 1.0
-----------
_February 23, 2011_

* ... use the following annotations on your classes and fields: `EqualsVerifier` will know what to do!
    * `@Immutable`: `EqualsVerifier` will not complain about non-final fields if your class is `@Immutable`. (It doesn't matter in which package the annotation is defined; `javax.annotations.concurrent.Immutable`, `net.jcip.annotations.Immutable` or your own implementation will all work fine.)
    * `@Nonnull`, `@NonNull` and `@NotNull`: `EqualsVerifier` will not check for potential `NullPointerException`s for any field marked with any of these annotations. (Again: the source package doesn't matter.) This is similar to calling `suppress(Warning.NULL_FIELDS)`, but on a per-field basis, instead of all-or-nothing. ([Issue 28](https://code.google.com/p/equalsverifier/issues/detail?id=28))
    * `@Entity`: `EqualsVerifier` will not complain about non-final fields if your class is a JPA Entity. Note that this only works for `javax.persistence.Entity`, not for `Entity` annotations from other packages. ([Issue 37](https://code.google.com/p/equalsverifier/issues/detail?id=37))
    * `@Transient`: Fields marked with this annotation (again, only from `javax.persistence.Transient`), will be treated the same as fields marked with Java's `transient` modifier; i.e., `EqualsVerifier` will complain if they are used in the `equals` contract.
* ... see which field throws a potential `NullPointerException`, instead of having to guess which one it was. ([Issue 39](https://code.google.com/p/equalsverifier/issues/detail?id=39))
* ... be sure that `EqualsVerifier` doesn't detect recursive data structures where there are none. ([Issue 34](https://code.google.com/p/equalsverifier/issues/detail?id=34))
* ... stop worrying about adding prefab values for `BigDecimal` and `BigInteger`, to avoid that pesky recursion warning. ([Issue 34](https://code.google.com/p/equalsverifier/issues/detail?id=34))

Version 0.7
-----------
_November 15, 2010_

* ... test `equals` methods that use a call to `getClass()`, instead of an `instanceof` check, to determine the type of the object passed in, by calling the `#useGetClass()` method on `EqualsVerifier`.
* ... get a warning when you use a transient field in your `equals` or `hashCode` method. Don't worry; you can suppress this warning too.
* ... stop worrying about adding prefab values for certain Java API classes, like `Data`, `GregorianCalendar`, or `Pattern`, to avoid that pesky recursion warning.
* ... get a link to the [ErrorMessages Error messages] page to get more help, when !EqualsVerifier finds an error.
* ... enjoy many javadoc improvements (including the one in Issue 32).
* Also, the back end is almost completely re-written.

Version 0.6.5
-------------
_August 5, 2010_

* ... use `EqualsVerifier` on classes with fields of a wide variety of Java API types, without getting 'Recursive datastructure' errors all the time. ([Issue 30](https://code.google.com/p/equalsverifier/issues/detail?id=30))
* ... get a more helpful error message when `equals`, `hashCode` or `toString` throws something other than a `NullPointerException` when one of the fields in `null`. ([Issue 31](https://code.google.com/p/equalsverifier/issues/detail?id=31))

Version 0.6.4
-------------
_June 13, 2010_

* ... get an appropriate error message when `Arrays.hashCode()` or `Arrays.deepHashCode()` should have been used, instead of a message that a certain field is used in `equals` but not in `hashCode` or the other way around. ([Issue 27](https://code.google.com/p/equalsverifier/issues/detail?id=27))
* ... get less frequent non-nullity errors on `toString`.

Version 0.6.3
-------------
_May 18, 2010_

* ... test the equals/hashCode contract for classes that don't override `equals` or `hashCode`. ([Issue 23](https://code.google.com/p/equalsverifier/issues/detail?id=23))
* ... use `EqualsVerifier` in conjuction with the [http://www.eclemma.org/ EclEmma] code coverage tool. ([Issue 22](https://code.google.com/p/equalsverifier/issues/detail?id=22))
* ... test classes that contain (indirect) references to non-static inner classes, without getting messages about recursive data structures. ([Issue 21](https://code.google.com/p/equalsverifier/issues/detail?id=21))

Version 0.6.2
-------------
_February 11, 2010_

* ... use `#withPrefabValues()` for recursive data structure fields in superclasses again. ([Issue 20](https://code.google.com/p/equalsverifier/issues/detail?id=20))

Version 0.6.1
-------------
_February 6, 2010_

* ... use `#withPrefabValues()` for fields which delegate to abstract methods again. ([Issue 14](https://code.google.com/p/equalsverifier/issues/detail?id=14))
* ... access the `#verify()` method more quickly using autocompletion in IDEs. (The method `#verbose()` has been renamed to `#debug()`.)

Version 0.6
-----------
_January 30, 2010_

* ... use a more consistent API. `#with(Feature)` is now `#suppress(Warning)`, which feels more Java-y. Former features that were not warnings, are now proper methods on `EqualsVerifier`: `#verbose()` and `#withRedefinedSuperclass()`.
* ... get better error messages:
    * many messages now span multiple lines for improved readability;
    * hashCodes are printed (where relevant);
    * unexpected exceptions are no longer eaten by `EqualsVerifier`, so they can be read without calling `#verbose()`;
    * calls to abstract methods from within `equals` and `hashCode`, which cannot be resolved, are now detected and properly reported. ([Issue 14](https://code.google.com/p/equalsverifier/issues/detail?id=14))
* ... verify classes that contain fields whose `equals` methods might throw `NullPointerException`s. ([Issue 19](https://code.google.com/p/equalsverifier/issues/detail?id=19))
* ... be sure that `EqualsVerifier` doesn't detect recursive data structures where there are none. ([Issue 18](https://code.google.com/p/equalsverifier/issues/detail?id=18))
* ... use fields of type `Class` in your classes. ([Issue 17](https://code.google.com/p/equalsverifier/issues/detail?id=17))

Version 0.5
-----------
_September 1, 2009_

* ... test classes that contain fields of interface or abstract class types (such as Lists, and other Collection types). ([Issue 12](https://code.google.com/p/equalsverifier/issues/detail?id=12))
* ... step through the source code in Eclipse after adding the EqualsVerifier jar to your project. ([Issue 11](https://code.google.com/p/equalsverifier/issues/detail?id=11))

Version 0.4
-----------
_August 29, 2009_

* ... verify equality rules that are more relaxed than simple field-by-field comparison. ([Issue 9](https://code.google.com/p/equalsverifier/issues/detail?id=9))

Version 0.3
-----------
_August 1, 2009_

* ... use `EqualsVerifier` without having to call `#withPrefabValues(...)` all the time. ([Issue 7](https://code.google.com/p/equalsverifier/issues/detail?id=7))

Version 0.2
-----------
_July 3, 2009_

* ... use the _fields are never null_ feature on classes instantiated with `#forClass(Class)`. ([Issue 1](https://code.google.com/p/equalsverifier/issues/detail?id=1))
* ... check if `Arrays.deepEquals` was used for multidimensional and `Object` arrays. ([Issue 3](https://code.google.com/p/equalsverifier/issues/detail?id=3))
* ... access optional features through a clean enum instead of through lots of different method calls. ([Issue 4](https://code.google.com/p/equalsverifier/issues/detail?id=4))
* ... use `EqualsVerifier` in unit test frameworks other than JUnit 4 ([Issue 5](https://code.google.com/p/equalsverifier/issues/detail?id=5))
* ... have stacktraces be printed to `System.err`. ([Issue 6](https://code.google.com/p/equalsverifier/issues/detail?id=6))

Version 0.1
-----------
_June 1, 2009_

* ... use `EqualsVerifier`!
