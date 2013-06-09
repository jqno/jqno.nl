---
title: EqualsVerifier
blog: equalsverifier
layout: equalsverifier
---
Quick start
-----------
1. Get EqualsVerifier.
    * Maven users, add this to your POM:

        <pre class="prettyprint">
        &lt;dependency>
            &lt;groupId>nl.jqno.equalsverifier&lt;/groupId>
            &lt;artifactId>equalsverifier&lt;/artifactId>
            &lt;version>???&lt;/version>
            &lt;scope>test&lt;/scope>
        &lt;/dependency>
        </pre>
        Don't forget to replace `???` with the most recent version number!
    * ANT users, make sure the following jars are on the classpath:

        * [objenesis-1.1.jar](http://code.google.com/p/objenesis/)
        * [cglib-nodep-2.2.jar](http://cglib.sourceforge.net/)
        * The most recent EqualsVerifier jar. You can download it from [maven.org](http://search.maven.org/#search|gav|1|g%3A%22nl.jqno.equalsverifier%22%20AND%20a%3A%22equalsverifier%22).

2. Use, as follows, in your unit test:
    <pre class="prettyprint">
    @Test
    public void equalsContract() {
        EqualsVerifier.forClass(My.class).verify();
    }
    </pre>
    Or, if you prefer to use a `getClass()` check instead of an `instanceof` check in the body of your `equals` method:

    <pre class="prettyprint">
    @Test
    public void equalsContract() {
        EqualsVerifier.forClass(My.class)
                .usingGetClass()
                .verify();
    }
    </pre>
    With some warnings suppressed:

    <pre class="prettyprint">
    @Test
    public void equalsContract() {
        EqualsVerifier.forClass(My.class)
                .suppress(Warning.NONFINAL_FIELDS, Warning.NULL_FIELDS)
                .verify();
    }
    </pre>

When EqualsVerifier detects a problem, it will explain what it thinks is wrong. In some cases (mostly when it needs more information), it will say what to do to fix the problem.


Need help?
----------
Please take a look at the [Help page]({{ pcurl('equalsverifier/help') }}).


Fork me on GitHub!
------------------
The source for this project is hosted on [GitHub](https://github.com/jqno/equalsverifier). The [issue tracker](https://code.google.com/p/equalsverifier/issues/list) can be found on Google Code.

Pull Requests are welcome! But please also [open an issue](https://code.google.com/p/equalsverifier/issues/list) or [send a message to the Google Group](https://groups.google.com/forum/?fromgroups#!forum/equalsverifier) so we can discuss it.
