---
title: FAQ
blog: equalsverifier
layout: equalsverifier
---
* [Philosophy](#philosophy)
    * [Can EqualsVerifier give false positives?](#falsepositives)
    * [Can EqualsVerifier give false negatives?](#falsenegatives)
    * [So what's the point of EqualsVerifier then?](#whatsthepoint)
    * [How should I write a good equals method?](#howtowriteequals)
    * [Does EqualsVerifier give me 100% code coverage?](#coverage)
* [Error messages](#errormessages)
    * [Can you explain the error message I just got?](#explainerrormessages)
    * [I'm testing java.lang.String, but it fails](#testingString)
    * [Why do other tests sometimes fail if I use #forExamples?](#forExamples)
* [Build](#build)
    * [How do I build EqualsVerifier?](#howtobuild)
    * [Why can't Eclipse resolve the dependencies?](#eclipseresolvedeps)
    * [I get generics errors while building](#genericserrors)


<a id="philosophy"></a>
Philosophy
----------

<a id="falsepositives"></a>
### Can EqualsVerifier give false positives?
Yes, it can. EqualsVerifier will not search a class's entire state space. If you write an `equals` method as below, EqualsVerifier will not notice that it's incorrect.

<pre class="prettyprint">
public boolean equals(Object obj) {
    if (!(obj instanceof Synonym)) {
        return false;
    }
    Synonym other = (Synonym)obj;
    if ("equality".equals(word) && "sameness".equals(other.word)) {
        return true;
    }
    return word.equals(other.word);
}
</pre>

The example is a little far-fetched, but it illustrates the point that you have to write bad code _on purpose_ to make EqualsVerifier think your `equals` method is correct when it is not.

<a id="falsenegatives"></a>
### Can EqualsVerifier give false negatives?
Yes, it can. Just try to verify `String`: it will fail. `String` employs some tricks for better performance, that EqualsVerifer can't verify. However, the good people who wrote `String` knew what they were doing, and have proven their implementation correct. If you prove your implementation correct, you don't need EqualsVerifier.

<a id="whatsthepoint"></a>
### So what's the point of EqualsVerifier then?
EqualsVerifier is a useful tool for anyone who wants to implement the `equals` and `hashCode` methods, without trying anything fancy. Implementing `equals` is hard, even for simple classes. Since most classes are simple, EqualsVerifier can be a great help in making sure that your implementation fulfills the contract.

EqualsVerifier catches common mistakes, such as forgetting to include a field in `hashCode` that is tested in `equals`. It also encourages robustness, for example by warning about non-final fields, and by watching out for inheritance issues.

If you run into a situation where EqualsVerifier can't help you, you are probably in one (or more) of the following situations:

1. You're writing non-trivial, high-performance niche code at NASA.
1. You're a Java API developer.
1. You're writing malicious code on purpose.
1. You're writing code that's way too complicated, and you should probably refactor.

(And even in the first two cases, EqualsVerifier can help creating an initial working implementation.)

<a id="howtowriteequals"></a>
### How should I write a good `equals` method?
By far the best explanation on the subject, in my opinion, can be found in Chapter 28 of _Programming in Scala_ by Martin Odersky, Bill Venners and Lex Spoon. The authors have adapted the chapter to Java and placed it online [here](http://www.artima.com/lejava/articles/equality.html).

Another excellent source, of course, is Items 8 and 9 from Josh Bloch's _Effective Java, Second Edition_.

Note: if you use the `canEqual` method described by Odersky et al., you need to use `#withRedefinedSubclass(Class)` in the tests of every class that you expect to be overridden, and `#withRedefinedSuperclass()` in the tests of every class that override a superclass's `equals` method. The two can be combined, if necessary.

<a id="coverage"></a>
### Does EqualsVerifier give me 100% code coverage?
Yes, it does. EqualsVerifier has a collection many different common implementation patterns of `equals` and `hashCode`, including those generated by Eclipse, Netbeans and IntelliJ IDEA, which are tested for 100% code coverage by [JaCoCo](http://www.eclemma.org/jacoco/).

Of course, there are many ways to implement `equals`, and despite my efforts, it's possible that 'I missed a spot'. See [this post]({{ pcposturl(2013, 12, 28, "coverage-is-not-100-percent") }}) for more information on what to do when you find such a case.


<a id="errormessages"></a>
Error messages
--------------
<a id="explainerrormessages"></a>
### Can you explain the error message I just got from EqualsVerifier?
Take a look at [the Error Messages page]({{ pcurl('equalsverifier/errormessages') }}).

<a id="testingString"></a>
### I'm testing `java.lang.String`, but it fails.
Please don't test `java.lang.String`, because indeed it fails. Please see the section on [false negatives](#falsenegatives) above.

<a id="forExamples"></a>
### Why do other tests sometimes fail if I use `#forExamples`?
As part of the verification process, EqualsVerifier will assign values to all fields of the class under test, including static fields (except static final fields). These values are unpredictable, may break your class invariants, and may even be invalid under normal conditions.

If you use `EqualsVerifier.forExamples`, and you re-use the instances of the class under test, this may impact other tests that also use these instances and expect them to have different values. Instead, use dedicated instances for the EqualsVerifier test.


<a id="build"></a>
Build
-----
<a id="howtobuild"></a>
### How do I build EqualsVerifier?
First, check out the code. You can open the project in Eclipse, or you can run ANT directly. Note that Java 6 is required to compile the project, although the ANT script will generate Java 5 compatible JAR files.

<a id="eclipseresolvedeps"></a>
### Why can't Eclipse resolve the dependencies?
Until version 1.5, EqualsVerifier used [Apache Ivy](http://ant.apache.org/ivy/) to resolve all dependencies. If you want these older versions of EqualsVerifier to compile in Eclipse, you need to install the [IvyDE](http://ant.apache.org/ivy/ivyde/) plugin, which will take care of the dependencies for you.

<a id="genericserrors"></a>
### I get generics errors while building
    type parameters of <S>S cannot be determined; no unique maximal instance exists for 
    type variable S with upper bounds T,java.lang.Object

EqualsVerifier makes heavy use of generics. Sun's Java compiler has a known bug regarding generics. Since I use Eclipse and OpenJDK, which do not exhibit this bug, it's possible I write code that triggers the bug, and the code might not compile. If this happens, please let me know in an issue and I'll fix it.
