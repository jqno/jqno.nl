---
title: Documentation
blog: equalsverifier
layout: equalsverifier
---
EqualsVerifier is a Java tool that can be used in unit tests. It verifies whether the contract for the `equals` and `hashCode` methods is met. The contracts are described in the Javadoc comments for the `java.lang.Object` class.


Usage
-----
Use, within unit test method, as follows:

* Create an instance of EqualsVerifier. Call {@link #forClass(Class)} to supply a reference to the class that contains the {@code equals} method to test. Also, {@link #forRelaxedEqualExamples(Object, Object, Object...)} can be used if the class under test has relaxed equality rules, for example, if the contents of two fields of the same type can be interchanged without breaking equality.
* If the class under test is designed for inheritance, and the `equals` and `hashCode` methods can be overridden, an instance of the class is not permitted to be equal to an instance of a subclass, even though all the relevant fields are equal. Call `#withRedefinedSubclass(Class)` to supply a reference to such a subclass, or call `suppress(Warning)` with `Warning.STRICT_INHERITANCE` to disable the check.
* Call `suppress(Warning)` to suppress warnings given by EqualsVerifier.
* Call `#verify()` to perform the actual verifications.

Example use:

<pre class="prettyprint">
@Test
public void equalsContract() {
    EqualsVerifier.forClass(My.class).verify();
}
</pre>

Or, with some warnings suppressed:

<pre class="prettyprint">
@Test
public void equalsContract() {
    EqualsVerifier.forClass(My.class)
            .suppress(Warning.NONFINAL_FIELDS, Warning.NULL_FIELDS)
            .verify();
}
</pre>

When EqualsVerifier detects a problem, it will explain what it thinks is wrong. In some cases (mostly when it needs more information), it will say what to do to fix the problem.


What does it do?
----------------
EqualsVerifier checks the following properties:

* Preconditions for EqualsVerifier itself.
* Reflexivity and symmetry of the `equals` method.
* Symmetry and transitivity of the `equals` method within an inheritance hierarchy, when applicable.
* Consistency (by repeatedly calling `equals`).
* "Non-nullity".
* That `equals`, `hashCode` and `toString` not be able to throw `NullPointerException`. (Optional)
* The `hashCode` contract.
* That `equals` and `hashCode` be defined in terms of the same fields.
* Immutability of the fields in terms of which `equals` and `hashCode` are defined. (Optional)
* The finality of the fields in terms of which `equals` and `hashCode` are defined. (Optional)
* Finality of the class under test and of the `equals` method itself, when applicable.


Equality and inheritance
------------------------
While a class can define a perfect `equals` method, a subclass can still break this contract, even for its superclass. Therefore, a class should either be final, or the `equals` contract should hold for its subclasses as well. This means that an instance of a class should be equal to an instance of a subclass for which all fields are equal.

Similarly, if `equals` is overridden, it can break the contract. So either should the `equals` method be final, thereby guaranteeing its adherence to the contracts for itself and all its subclasses, or instances of a class that redefines `equals` may never equal instances of its superclass, even when their (shared) state is equal. This should be tested by calling `#withRedefinedSuperclass()` (in the subclass) or `#withRedefinedSubclass(Class)` (in the superclass). EqualsVerifier should be used separately on each of the classes in the hierarchy. If necessary, `#withRedefinedSuperclass()` and `#withRedefinedSubclass(Class)` can be combined.

For an example of an implementation of such redefined `equals` methods, see `CanEqualColorPoint` in the EqualsVerifier's unit tests. See Chapter 28 of _Programming in Scala_ (full reference below) for an explanation of how and why this works.


Inspiration
-----------
The verifications are inspired by:

* _Effective Java, Second Edition_ by Joshua Bloch, Addison-Wesley, 2008: Items 8 (_Obey the general contract when overriding `equals`_) and 9 (_Always override `hashCode` when you override `equals`_).
* _Programming in Scala_ by Martin Odersky, Lex Spoon and Bill Venners, Artima Press, 2008: Chapter 28 (_Object equality_) / Second Edition, 2011: Chapter 30 (_Object equality_). Note: a Java-centric version of this chapter can be found online [here](http://www.artima.com/lejava/articles/equality.html).
* _JUnit Recipes_ by J.B. Rainsberger, Manning, 2005: Appendix B.2 (_Strangeness and transitivity_).
* _How Do I Correctly Implement the equals() Method?_ by Tal Cohen, Dr. Dobb's Journal, May 2002. Read the [article](http://www.ddj.com/java/184405053) and Cohen's [follow-up](http://tal.forum2.org/equals).

And of course GSBase's `EqualsTester` by Mike Bowler, Gargoyle Software, which I think is pretty good, but lacking with respect to inheritance. Also, it requires that examples be given, while EqualsVerifier can auto-generate them in most cases (which, of course, is way cool). You can find it on [SourceForge](http://gsbase.sourceforge.net/index.html).
