% Software Engineering
% 2016-10-24

These are some of my observations and thoughts about best-practices for software
engineering.

## Er-er

Classes that end in _er_ are suspicious. Especially managers and controllers.
They tend to have more than one responsibility to begin with, and easily
accumulate more responsibilities by being easy targets to match against new
functionality. "Manager" and "Controller" are so non-specific as to be
completely invisible when you're looking around a code base looking for where to
put a new little bit of code. A `SearchManager` or `SearchController` are
only bound by the general domain of "something having to do with search".

For OOP, consider instead decomposing search into `Query`, `SearchParameters`,
`Result`, and so on.

## Logging is a side-effect

Logging is a side effect. And depending on how you're logging, it's possibly
adding a dependency, or strongly coupling a domain, ui, or other component to a
logging component. Especially deep in domain models, consider surfacing all the
information needed to log, all errors for example. And logging further out in
the stack.

## Always Return

Methods with a `void` return are suspicious. They're guaranteed to have
side-effects if they're at all useful. I'd like to practice a convention where
`void` methods return `self` instead. This makes chaining easier. `void` methods
that take arguments may be better off returning the arguments themselves, though
I'm inclined towards `self`. This needs more thought.

Methods with a `void` return that don't take arguments are doubly suspicious.
They are a very procedural in nature, and in my experience are an OOP trap into
spaghetti code. That's not to say that these are forbidden. It can be very
useful to capture a bit of repetitive code out of an object. But be vigilant as
to whether a different pattern may be appropriate.

## Use Self in Methods

If an instance method doesn't use `self`, it shouldn't be an instance method.
Maybe it should be a class method, or a free function. But examine what
structure it operates on and you may find it should be an extension on that
type.
