---
title: Subclass: ... equals subclass instance ...
blog: equalsverifier
layout: equalsverifier
category: subclassequalssubclassinstance
---
You have called `withRedefinedSubclass`, but EqualsVerifier has found an instance of the superclass and an instance of the subclass that are equal to each other. However, `withRedefinedSubclass` is meant for situations where the subclass adds state to it parent, and checks that in its `equals` method. In such cases, no instance of the subclass may be equal to any instance of its parent; otherwise your `equals` method will lose either symmetry or transitivity, whithout which it no longer follows the `equals` contract. For more details on this, please read [this article](http://www.artima.com/lejava/articles/equality.html) by Odersky, Spoon and Venners.

To solve this, make sure that you follow the advice laid out by the article mentioned above. Or, if you didn't intend to add state to the subclasses, you can also decide to mark the `equals` and `hashCode` methods as final, and remove the call to `withRedefinedSubclass`. That way, you can't override `equals` or `hashCode` anymore, but you won't have to complicate your `equals` method unnecessarily to make everything work, either. A last resort would be to suppress `Warning.STRICT_INHERITANCE` (and also remove the call to `withRedefinedSubclass`. That will disable the check, but will likely leave your `equals` method without symmetry or transitivity.
