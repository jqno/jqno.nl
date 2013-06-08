---
title: Significant fields: equals relies on foo, but hashCode does not
blog: equalsverifier
layout: equalsverifier
---
The cause for this message is usually the obvious one.

If it's not, then it might be caused by a cached hashCode field, similar to the one in `java.lang.String`. For example, given the following (abridged) class:

<pre class="prettyprint">
class CachedHashCode {
	private final int foo;
	private final int bar;
	private int cachedHashCode = 0;

	public int hashCode() {
		if (cachedHashCode == 0) {
			cachedHashCode += 31 * foo;
			cachedHashCode += 31 * bar;
		}
		return cachedHashCode;
	}
}
</pre>

EqualsVerifier thinks `cachedHashCode` is a regular field like all the others, and assigns it a value other than 0. The first thing the `hashCode` method does is compare it to 0. This yields false, so the method always returns EqualsVerifier's (incorrect) value. EqualsVerifier doesn't know about this, so when it tries changing `foo`, it notices a difference in the result of `equals`, but not in the result of `hashCode` (since the latter is never properly initialized). And this is why EqualsVerifier thinks the problem is caused by `foo`.

Unfortunately, this kind of hashCode caching is really hard for EqualsVerifier to detect. Furthermore, if you actually do this, you either _really_ know what you're doing and then you probably don't need EqualsVerifier anyway (this applies to `String`), or you run the risk of doing it wrong (as in the example above, which isn't thread-safe) and shouldn't be using this trick in the first place. It's basically an optimization that will, in most cases anyway, be premature. The chances of EqualsVerifier supporting cached hashCodes in the future are therefore quite slim.
