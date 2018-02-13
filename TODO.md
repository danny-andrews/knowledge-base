## Subtyping vs inheritance

In the object-oriented framework, inheritance is usually presented as a feature that goes hand in hand with subtyping when one organizes abstract datatypes in a hierarchy of classes. However, the two are orthogonal ideas.

Subtyping refers to compatibility of interfaces. A type B is a subtype of A if every function that can be invoked on an object of type A can also be invoked on an object of type B.
Inheritance refers to reuse of implementations. A type B inherits from another type A if some functions for B are written in terms of functions of A.
In the object-oriented framework, if B is a subclass of A then it is clear that B is a subtype of A. Very often, B also inherits from A--for example, recall the way the implementation of bonus() for Manager called on the function bonus() defined in Employee.

However, subtyping and inheritance need not go hand in hand. Consider the data structure deque, a double-ended queue. A deque supports insertion and deletion at both ends, so it has four functions insert-front, delete-front, insert-rear and delete-rear. If we use just insert-rear and delete-front we get a normal queue. On the other hand, if we use just insert-front and delete-front, we get a stack. In other words, we can implement queues and stacks in terms of deques, so as datatypes, Stack and Queue inherit from Deque. On the other hand, neither Stack not Queue are subtypes of Deque since they do not support all the functions provided by Deque. In fact, in this case, Deque is a subtype of both Stack and Queue!

In general, therefore, subtyping and inheritance are orthogonal concepts. Since inheritance involves reuse of implementations, we could have an inheritance relationship between classes that are incomparable in the subtype relationship. In a more accurate sense, this is what holds between Stack, Queue and Deque. The type Stack supports functions push and pop that are implemented in terms of the Deque functions insert-front and delete-front. Similarly, Queue supporst functions insert and delete that are implemented in terms of the Deque functions insert-rear and delete-front. The function names are different, so none of these types is comparable in the subtype relation, but Stack and Queue do inherit from Deque.

A relative unfortunately died and left you his bookstore.

You can now read all the books there, sell them, you can look at his accounts, his customer list, etc. This is inheritance - you have everything the relative had. Inheritance is a form of code reuse.

You can also re-open the book store yourself, taking on all of the relative's roles and responsibilities, even though you add some changes of your own - this is subtyping - you are now a bookstore owner, just like your relative used to be.

## Objects vs Records
As a general rule of thumb, don't use objects. The additional complexity they bring in is not that often worth it. I think that's a rule that apply to other languages as well, but that's another story. At least with OCaml one can objectively (no pun intended) say that the common practice is to not use object except in rare cases.

Object provide a bundle of:

A "standard" style for carrying around and using records of functions, with possibly polymorphic types
Facilities for open recursion through self (implementation inheritance)
A structural, extensible product type with row polymorphism (for open object types) and subtyping (for closed object types)
You may use any of those together, or separately.

In my experience, point (1) alone is not particularly worth using objects: you can just use records of functions and it is just as clear.

Uses Cases of Open Recursive / Inheritance

Point (2) on the contrary is a good justification for using an object-oriented style; it is used in this manner by Camlp4 for example: Camlp4 defines classes that fold over an AST doing nothing, and you can inherit this traversal object to implement the behavior you want on only the syntactic constructions you want (and defer the boring traversal plumbing to your mother class).

For example, one may extend the Camlp4Ast.map object, that defines a simple map function on the Camlp4 representation of the OCaml Abstract Syntax Trees, just mapping every construct to itself, recursively. If you want to, say, map all (fun x -> e1) e2 expressions to let x = e2 in e1, you inherit from this object, and override the expr method, handling only the case you want (left-hand-side is a function), delegating the other to the inherited behavior. This will give you an object that knows how to apply this transform over a complete program, recursively, without you having to write any boilerplate code; and you can further extend this transformation with additional behavior if you so wish.

Fun with object types

Point (3) is also a justification for using objects as "extensible records", or as "type-level arrays"; some libraries using object types, but not object at runtime: they use object types as phantom types to carry around information, benefiting from the richer type-level operations you can have on objects. Besides, structural typing allow for different authors to have compatible types without strong dependencies to a common component defining the (nominal) types they share; objects have been used for standardization of input/output components for example.

A not-uncommon, very simple use case of this is an idiomatic way to represent types that have a lot of parameters. Instead of writing:

type ('name, 'addr, 'job, 'id) person = ....
val me : (string, string, Job.t, Big_int.big_int) person
you can use object types as structural "type-level records" to write instead:

type 'a person = .... constraint 'a = < name:'n; addr:'a; job:'j; id:'i >
val me : < name:string; addr:string; job:Job.t; id:Big_int.big_int > person
For the more advanced use of object types as phantom types, you can have a look at the ShCaml (doc) library (where it is used to represent which shell commands a string input is compatible with) by Alec Heller and Jesse Tov, or my own Macaque library (doc and api doc), which uses object types to represent SQL values (including nullability information) and table row types.
Polymorphic variants (another advanced feature of OCaml type system; in a sentence, the relation between objects and records is the same as the relation between polymorphic variants and algebraic sum types).have also been used as phantom types, for example in this simple example by Richard Jones, or to check validity of HTML documents in the Ocsigen framework.

Beware however that those advanced type hackeries come at a significant complexity cost; before using them, you must carefully balance it with the additional expressivity and static safety that they bring.

Summing up

as a base assumption, you're safe with not using objects at all; you should only introduce them in your design if you feel you're missing something, not by default
objects are convenient for open recursion / inheritance: refining a behavior that is already defined in the default/boring cases
structural typing is occasionally useful when you want to reason on values independently providing a set of features/capacities

## Terms

Open Recursion
Implementation inheritance

## OO Manifestation in Functional Langauges Questions
What form will Inheritance take? How will code be shared? Explicit composition?
Polymorphism?
Should all objects be immutable?
Just use records?
What do objects give us that records don't? Can we define an ADT with records?

Row polymorphism

http://www.di.ens.fr/~zappa/teaching/stt05/types-3.pdf
https://www.cs.cmu.edu/~neelk/rows.pdf

Objects are with row polymorphism
