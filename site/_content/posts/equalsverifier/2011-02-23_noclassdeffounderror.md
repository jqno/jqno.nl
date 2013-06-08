---
title: NoClassDefFoundError
blog: equalsverifier
layout: equalsverifier
---
    java.lang.AssertionError: java.lang.NoClassDefFoundError: net/sf/cglib/asm/FieldVisitor

If you get this error message, then you probably have a transitive dependency problem with cglib. EqualsVerifier requires version 2.2, but other libraries may depend on earlier versions. If their dependency happens to be resolved before EqualsVerifier's, the wrong version of cglib will be loaded on your classpath.

To fix this, make sure you have cglib 2.2. If you use Maven, add the following lines to your pom:

<pre class="prettyprint">
&lt;dependencyManagement>
	&lt;dependencies>
		&lt;dependency>
			&lt;groupId>cglib&lt;/groupId>
			&lt;artifactId>cglib-nodep&lt;/artifactId>
			&lt;version>2.2&lt;/version>
		&lt;/dependency>
	&lt;/dependencies>
&lt;/dependencyManagement>
</pre>

Also, make sure that you use cglib-nodep, instead of 'vanilla' cglib!
