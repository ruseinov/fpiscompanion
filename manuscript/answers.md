# Answers to exercises

This section contains answers to exercises in the book. Note that usually only one answer is given per exercise, but there may be more than one correct answer. We highly recommend that you check out [the book's source code repository from GitHub](https://github.com/fpinscala/fpinscala) and look at the answers given there for each chapter. Those answers are shown *in context*, with more detailed explanations, more varied solutions, and in some cases complete implementations that do not fit in this booklet.

## Answers to exercises for chapter 2

### Exercise 2.01

``` scala
def fib(n: Int): Int = {
  @annotation.tailrec
  def loop(n: Int, prev: Int, cur: Int): Int =
    if (n == 0) prev 
    else loop(n - 1, cur, prev + cur)
  loop(n, 0, 1)
}
```

0 and 1 are the first two numbers in the sequence, so we start the accumulators with those. At every iteration, we add the two numbers to get the next one.

### Exercise 2.02

``` scala
def isSorted[A](as: Array[A], gt: (A,A) => Boolean): Boolean = {
  @annotation.tailrec
  def go(n: Int): Boolean =
    if (n >= as.length-1) true
    else if (gt(as(n), as(n+1))) false
    else go(n+1)

  go(0)
}
```

### Exercise 2.03

Note that `=>` associates to the right, so we could write the return type as
   `A => B => C`

``` scala
def curry[A,B,C](f: (A, B) => C): A => (B => C) =
  a => b => f(a, b)
```

NB: The `Function2` trait has a `curried` method already, so if you wanted to
   cheat a little you could write the answer as `f.curried`

### Exercise 2.04

``` scala
def uncurry[A,B,C](f: A => B => C): (A, B) => C =
  (a, b) => f(a)(b)
```

NB: There is a method on the `Function` object in the standard library,
`Function.uncurried` that you can use for uncurrying.

Note that we can go back and forth between the two forms. We can curry and uncurry
and the two forms are in some sense "the same". In FP jargon, we say that they
are _isomorphic_ ("iso" = same; "morphe" = shape, form), a term we inherit from
category theory.

### Exercise 2.05

``` scala
def compose[A,B,C](f: B => C, g: A => B): A => C =
  a => f(g(a))
```

## Answers to exercises for chapter 3

### Exercise 3.01

Three. The third case is the first that matches, with `x` bound to 1 and `y` bound to 2. 

### Exercise 3.02

Although we could return `Nil` when the input list is empty, we choose to throw an exception instead. This is a somewhat subjective choice. In our experience, taking the tail of an empty list is often a bug, and silently returning a value just means this bug will be discovered later, further from the place where it was introduced. 

It's generally good practice when pattern matching to use `_` for any variables you don't intend to use on the right hand side of a pattern. This makes it clear the value isn't relevant.  

``` scala
def tail[A](l: List[A]): List[A] = 
  l match {
    case Nil => sys.error("tail of empty list")
    case Cons(_,t) => t
  }
```

### Exercise 3.03

If a function body consists solely of a match expression, we'll often put the match on the same line as the function signature, rather than introducing another level of nesting.

``` scala
def setHead[A](l: List[A], h: A): List[A] = l match {
  case Nil => sys.error("setHead on empty list")
  case Cons(_,t) => Cons(h,t)
}
```

### Exercise 3.04

Again, it's somewhat subjective whether to throw an exception when asked to drop more elements than the list contains. The usual default for `drop` is not to throw an exception, since it's typically used in cases where this is not indicative of a programming error. If you pay attention to how you use `drop`, it's often in cases where the length of the input list is unknown, and the number of elements to be dropped is being computed from something else. If `drop` threw an exception, we'd have to first compute or check the length and only drop up to that many elements.  

``` scala
def drop[A](l: List[A], n: Int): List[A] = 
  if (n <= 0) l
  else l match {
    case Nil => Nil
    case Cons(_,t) => drop(t, n-1) 
  }
```

### Exercise 3.05

Somewhat overkill, but to illustrate the feature we're using a *pattern guard*,
to only match a `Cons` whose head satisfies our predicate, `f`. 
The syntax is to add `if <cond>` after the pattern, before the `=>`, 
where `<cond>` can use any of the variables introduced by the pattern.

``` scala
def dropWhile[A](l: List[A], f: A => Boolean): List[A] = 
  l match {
    case Cons(h,t) if f(h) => dropWhile(t, f) 
    case _ => l
  }

```

### Exercise 3.06

Note that we're copying the entire list up until the last element. Besides being inefficient, the natural recursive solution will use a stack frame for each element of the list, which can lead to stack overflows for large lists (can you see why?). With lists, it's common to use a temporary, mutable buffer internal to the function (with lazy lists or streams, which we discuss in chapter 5, we don't normally do this). So long as the buffer is allocated internal to the function, the mutation is not observable and RT is preserved.

Another common convention is to accumulate the output list in reverse order, then reverse it at the end, which doesn't require even local mutation. We'll write a reverse function later in this chapter.

``` scala
def init[A](l: List[A]): List[A] = 
  l match { 
    case Nil => sys.error("init of empty list")
    case Cons(_,Nil) => Nil
    case Cons(h,t) => Cons(h,init(t))
  }
def init2[A](l: List[A]): List[A] = {
  import collection.mutable.ListBuffer
  val buf = new ListBuffer[A]
  @annotation.tailrec
  def go(cur: List[A]): List[A] = cur match {
    case Nil => sys.error("init of empty list")
    case Cons(_,Nil) => List(buf.toList: _*)
    case Cons(h,t) => buf += h; go(t)
  }
  go(l)
}
```

### Exercise 3.07

No, this is not possible! The reason is because *before* we ever call our function, `f`, we evaluate its argument, which in the case of `foldRight` means traversing the list all the way to the end. We need _non-strict_ evaluation to support early termination---we discuss this in chapter 5.

### Exercise 3.08

We get back the original list! Why is that? As we mentioned earlier, one way of thinking about what `foldRight` "does" is it replaces the `Nil` constructor of the list with the `z` argument, and it replaces the `Cons` constructor with the given function, `f`. If we just supply `Nil` for `z` and `Cons` for `f`, then we get back the input list. 

``` scala
foldRight(Cons(1, Cons(2, Cons(3, Nil))), Nil:List[Int])(Cons(_,_))
Cons(1, foldRight(Cons(2, Cons(3, Nil)), Nil:List[Int])(Cons(_,_)))
Cons(1, Cons(2, foldRight(Cons(3, Nil), Nil:List[Int])(Cons(_,_))))
Cons(1, Cons(2, Cons(3, foldRight(Nil, Nil:List[Int])(Cons(_,_)))))
Cons(1, Cons(2, Cons(3, Nil))) 
```

### Exercise 3.09

``` scala
def length[A](l: List[A]): Int = 
  foldRight(l, 0)((_,acc) => acc + 1)
```

### Exercise 3.10

It's common practice to annotate functions you expect to be tail-recursive with the `tailrec` annotation. If the function is not tail-recursive, it will yield a compile error, rather than silently compiling the code and resulting in greater stack space usage at runtime. 

``` scala
@annotation.tailrec
def foldLeft[A,B](l: List[A], z: B)(f: (B, A) => B): B = l match { 
  case Nil => z
  case Cons(h,t) => foldLeft(t, f(z,h))(f)
}
```

### Exercise 3.11

``` scala
def sum3(l: List[Int]) = foldLeft(l, 0)(_ + _)
def product3(l: List[Double]) = foldLeft(l, 1.0)(_ * _)

def length2[A](l: List[A]): Int = foldLeft(l, 0)((acc,h) => acc + 1)
```

### Exercise 3.12

``` scala
def reverse[A](l: List[A]): List[A] =
  foldLeft(l, List[A]())((acc,h) => Cons(h,acc))
```

### Exercise 3.13

The implementation of `foldRight` in terms of `reverse` and `foldLeft` is a common trick for avoiding stack overflows when implementing a strict `foldRight` function as we've done in this chapter. (We'll revisit this in a later chapter, when we discuss laziness).

The other implementations build up a chain of functions which, when called, results in the operations being performed with the correct associativity. We are calling `foldRight` with the `B` type being instantiated to `B => B`, then calling the built up function with the `z` argument. Try expanding the definitions by substituting equals for equals using a simple example, like `foldLeft(List(1,2,3), 0)(_ + _)` if this isn't clear. Note these implementations are more of theoretical interest - they aren't stack-safe and won't work for large lists.

``` scala
def foldRightViaFoldLeft[A,B](l: List[A], z: B)(f: (A,B) => B): B = 
  foldLeft(reverse(l), z)((b,a) => f(a,b))

def foldRightViaFoldLeft_1[A,B](l: List[A], z: B)(f: (A,B) => B): B = 
  foldLeft(l, (b:B) => b)((g,a) => b => g(f(a,b)))(z)

def foldLeftViaFoldRight[A,B](l: List[A], z: B)(f: (B,A) => B): B = 
  foldRight(l, (b:B) => b)((a,g) => b => g(f(b,a)))(z)
```

### Exercise 3.14

`append` simply replaces the `Nil` constructor of the first list with the second list, which is exactly the operation performed by `foldRight`.

``` scala
def appendViaFoldRight[A](l: List[A], r: List[A]): List[A] = 
  foldRight(l, r)(Cons(_,_))
```

### Exercise 3.15

Since `append` takes time proportional to its first argument, and this first argument never grows because of the right-associativity of `foldRight`, this function is linear in the total length of all lists. You may want to try tracing the execution of the implementation on paper to convince yourself that this works. 

Note that we're simply referencing the `append` function, without writing something like `(x,y) => append(x,y)` or `append(_,_)`. In Scala there is a rather arbitrary distinction between functions defined as _methods_, which are introduced with the `def` keyword, and function values, which are the first-class objects we can pass to other functions, put in collections, and so on. This is a case where Scala lets us pretend the distinction doesn't exist. In other cases, you'll be forced to write `append _` (to convert a `def` to a function value) or even `(x: List[A], y: List[A]) => append(x,y)` if the function is polymorphic and the type arguments aren't known. 

``` scala
def concat[A](l: List[List[A]]): List[A] = 
  foldRight(l, Nil:List[A])(append)
```

### Exercise 3.16

``` scala
def add1(l: List[Int]): List[Int] = 
  foldRight(l, Nil:List[Int])((h,t) => Cons(h+1,t))
```

### Exercise 3.17

``` scala
def doubleToString(l: List[Double]): List[String] = 
  foldRight(l, Nil:List[String])((h,t) => Cons(h.toString,t))
```

### Exercise 3.18

A natural solution is using `foldRight`, but our implementation of `foldRight` is not stack-safe. We can use `foldRightViaFoldLeft` to avoid the stack overflow (variation 1), but more commonly, with our current implementation of `List`, `map` will just be implemented using local mutation (variation 2). Again, note that the mutation isn't observable outside the function, since we're only mutating a buffer that we've allocated. 

``` scala
def map[A,B](l: List[A])(f: A => B): List[B] = 
  foldRight(l, Nil:List[B])((h,t) => Cons(f(h),t))

def map_1[A,B](l: List[A])(f: A => B): List[B] = 
  foldRightViaFoldLeft(l, Nil:List[B])((h,t) => Cons(f(h),t))

def map_2[A,B](l: List[A])(f: A => B): List[B] = {
  val buf = new collection.mutable.ListBuffer[B]
  def go(l: List[A]): Unit = l match {
    case Nil => ()
    case Cons(h,t) => buf += f(h); go(t)
  }
  go(l)
  List(buf.toList: _*)
  // converting from the standard Scala list to the list we've defined here
}
```

### Exercise 3.19

``` scala
/* 
The discussion about `map` also applies here.
*/
def filter[A](l: List[A])(f: A => Boolean): List[A] = 
  foldRight(l, Nil:List[A])((h,t) => if (f(h)) Cons(h,t) else t)

def filter_1[A](l: List[A])(f: A => Boolean): List[A] = 
  foldRightViaFoldLeft(l, Nil:List[A])((h,t) => if (f(h)) Cons(h,t) else t)

def filter_2[A](l: List[A])(f: A => Boolean): List[A] = {
  val buf = new collection.mutable.ListBuffer[A]
  def go(l: List[A]): Unit = l match {
    case Nil => ()
    case Cons(h,t) => if (f(h)) buf += h; go(t)
  }
  go(l)
  List(buf.toList: _*)
  // converting from the standard Scala list to the list we've defined here
}
```

### Exercise 3.20

``` scala
/* 
This could also be implemented directly using `foldRight`.
*/
def flatMap[A,B](l: List[A])(f: A => List[B]): List[B] = 
  concat(map(l)(f))
```

### Exercise 3.21

``` scala
def filterViaFlatMap[A](l: List[A])(f: A => Boolean): List[A] =
  flatMap(l)(a => if (f(a)) List(a) else Nil)
```

### Exercise 3.22

To match on multiple values, we can put the values into a pair and match on the pair, as shown next, and the same syntax extends to matching on N values (see sidebar "Pairs and tuples in Scala" for more about pair and tuple objects). You can also (somewhat less conveniently, but a bit more efficiently) nest pattern matches: on the right hand side of the `=>`, simply begin another `match` expression. The inner `match` will have access to all the variables introduced in the outer `match`. 

The discussion about stack usage from the explanation of `map` also applies here.

``` scala
def addPairwise(a: List[Int], b: List[Int]): List[Int] = (a,b) match {
  case (Nil, _) => Nil
  case (_, Nil) => Nil
  case (Cons(h1,t1), Cons(h2,t2)) => Cons(h1+h2, addPairwise(t1,t2))
}
```

### Exercise 3.23

This function is usually called `zipWith`. The discussion about stack usage from the explanation of `map` also applies here. By putting the `f` in the second argument list, Scala can infer its type from the previous argument list. 

``` scala
def zipWith[A,B,C](a: List[A], b: List[B])(f: (A,B) => C): List[C] =
  (a,b) match {
    case (Nil, _) => Nil
    case (_, Nil) => Nil
    case (Cons(h1,t1), Cons(h2,t2)) => Cons(f(h1,h2), zipWith(t1,t2)(f))
  }
```

### Exercise 3.24

It's good to specify some properties about these functions.
For example, do you expect these expressions to be true?

``` scala
(xs append ys) startsWith xs

xs startsWith Nil

(xs append ys append zs) hasSubsequence ys

xs hasSubsequence Nil
```

Here is one solution where those properties do in fact hold.
There's nothing particularly bad about this implementation,
except that it's somewhat monolithic and easy to get wrong.
Where possible, we prefer to assemble functions like this using
combinations of other functions. It makes the code more obviously
correct and easier to read and understand. Notice that in this
implementation we need special purpose logic to break out of our
loops early. In Chapter 5 we'll discuss ways of composing functions
like this from simpler components, without giving up the efficiency
of having the resulting functions work in one pass over the data.

``` scala
@annotation.tailrec
def startsWith[A](l: List[A], prefix: List[A]): Boolean = (l,prefix) match {
  case (_,Nil) => true
  case (Cons(h,t),Cons(h2,t2)) if h == h2 => startsWith(t, t2)
  case _ => false
}
@annotation.tailrec
def hasSubsequence[A](sup: List[A], sub: List[A]): Boolean = sup match {
  case Nil => sub == Nil
  case _ => startsWith(sup, sub)
  case Cons(h,t) => hasSubsequence(t, sub)
}
```

### Exercise 3.25

``` scala
def size[A](t: Tree[A]): Int = t match {
  case Leaf(_) => 1
  case Branch(l,r) => 1 + size(l) + size(r)
}
```

### Exercise 3.26

We're using the method `max` that exists on all `Int` values rather than an explicit `if` expression.

Note how similar the implementation is to `size`. We'll abstract out the common pattern in a later exercise. 

``` scala
def maximum(t: Tree[Int]): Int = t match {
  case Leaf(n) => n
  case Branch(l,r) => maximum(l) max maximum(r)
}
```

### Exercise 3.27

Again, note how similar the implementation is to `size` and `maximum`.

``` scala
def depth[A](t: Tree[A]): Int = t match {
  case Leaf(_) => 0
  case Branch(l,r) => 1 + (depth(l) max depth(r))
}
```

### Exercise 3.28

``` scala
def map[A,B](t: Tree[A])(f: A => B): Tree[B] = t match {
  case Leaf(a) => Leaf(f(a))
  case Branch(l,r) => Branch(map(l)(f), map(r)(f))
}
```

### Exercise 3.29

``` scala 
def mapViaFold[A,B](t: Tree[A])(f: A => B): Tree[B] = 
  fold(t)(a => Leaf(f(a)): Tree[B])(Branch(_,_))
```

Like `foldRight` for lists, `fold` receives a "handler" for each of the data constructors of the type, and recursively accumulates some value using these handlers. As with `foldRight`, `fold(t)(Leaf(_))(Branch(_,_)) == t`, and we can use this function to implement just about any recursive function that would otherwise be defined by pattern matching.

``` scala
def fold[A,B](t: Tree[A])(f: A => B)(g: (B,B) => B): B = t match {
  case Leaf(a) => f(a)
  case Branch(l,r) => g(fold(l)(f)(g), fold(r)(f)(g))
}

def sizeViaFold[A](t: Tree[A]): Int = 
  fold(t)(a => 1)(1 + _ + _)

def maximumViaFold(t: Tree[Int]): Int = 
  fold(t)(a => a)(_ max _)

def depthViaFold[A](t: Tree[A]): Int = 
  fold(t)(a => 0)((d1,d2) => 1 + (d1 max d2))
```

Note the type annotation required on the expression `Leaf(f(a))`. Without this annotation, we get an error like this: 

```
type mismatch;
  found   : fpinscala.datastructures.Branch[B]
  required: fpinscala.datastructures.Leaf[B]
     fold(t)(a => Leaf(f(a)))(Branch(_,_))
                                    ^  
```

This error is an unfortunate consequence of Scala using subtyping to encode algebraic data types. Without the annotation, the result type of the fold gets inferred as `Leaf[B]` and it is then expected that the second argument to `fold` will return `Leaf[B]`, which it doesn't (it returns `Branch[B]`). Really, we'd prefer Scala to infer `Tree[B]` as the result type in both cases. When working with algebraic data types in Scala, it's somewhat common to define helper functions that simply call the corresponding data constructors but give the less specific result type:  
 
``` scala
def leaf[A](a: A): Tree[A] = Leaf(a)
def branch[A](l: Tree[A], r: Tree[A]): Tree[A] = Branch(l, r)
```

## Answers to exercises for chapter 4

### Exercise 4.01

``` scala
def map[B](f: A => B): Option[B] = this match {
  case None => None
  case Some(a) => Some(f(a))
}

def getOrElse[B>:A](default: => B): B = this match {
  case None => default
  case Some(a) => a
}

def flatMap[B](f: A => Option[B]): Option[B] = 
  map(f) getOrElse None

/*
Of course, we can also implement `flatMap` with explicit pattern matching.
*/
def flatMap_1[B](f: A => Option[B]): Option[B] = this match {
  case None => None
  case Some(a) => f(a)
}

def orElse[B>:A](ob: => Option[B]): Option[B] = 
  this map (Some(_)) getOrElse ob

/*
Again, we can implement this with explicit pattern matching. 
*/
def orElse_1[B>:A](ob: => Option[B]): Option[B] = this match {
  case None => ob 
  case _ => this
}

/*
This can also be defined in terms of `flatMap`.
*/
def filter_1(f: A => Boolean): Option[A] =
  flatMap(a => if (f(a)) Some(a) else None)

/* Or via explicit pattern matching. */  
def filter(f: A => Boolean): Option[A] = this match {
  case Some(a) if f(a) => this
  case _ => None
}

```

### Exercise 4.02

``` scala
def variance(xs: Seq[Double]): Option[Double] = 
  mean(xs) flatMap (m => mean(xs.map(x => math.pow(x - m, 2))))
```

### Exercise 4.03

A bit later in the chapter we'll learn nicer syntax for writing functions like this.

``` scala
def map2[A,B,C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] =
  a flatMap (aa => b map (bb => f(aa, bb)))
```

### Exercise 4.04

Here's an explicit recursive version:

``` scala
def sequence[A](a: List[Option[A]]): Option[List[A]] =
  a match {
    case Nil => Some(Nil)
    case h :: t => h flatMap (hh => sequence(t) map (hh :: _))
  }
```

It can also be implemented using `foldRight` and `map2`. The type annotation on `foldRight` is needed here; otherwise Scala wrongly infers the result type of the fold as `Some[Nil.type]` and reports a type error (try it!). This is an unfortunate consequence of Scala using subtyping to encode algebraic data types.

``` scala
def sequence_1[A](a: List[Option[A]]): Option[List[A]] =
  a.foldRight[Option[List[A]]](Some(Nil))((x,y) => map2(x,y)(_ :: _))
```

### Exercise 4.05

``` scala
def traverse[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] =
  a match {
    case Nil => Some(Nil)
    case h::t => map2(f(h), traverse(t)(f))(_ :: _)
  }

def traverse_1[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] =
  a.foldRight[Option[List[B]]](Some(Nil))((h,t) => map2(f(h),t)(_ :: _))

def sequenceViaTraverse[A](a: List[Option[A]]): Option[List[A]] =
  traverse(a)(x => x)

```

### Exercise 4.06

``` scala
def map[B](f: A => B): Either[E, B] = 
  this match {
    case Right(a) => Right(f(a))
    case Left(e) => Left(e)
  }
  
def flatMap[EE >: E, B](f: A => Either[EE, B]): Either[EE, B] =
  this match {
    case Left(e) => Left(e)
    case Right(a) => f(a)
  }
def orElse[EE >: E, AA >: A](b: => Either[EE, AA]): Either[EE, AA] =
  this match {
    case Left(_) => b
    case Right(a) => Right(a)
  }
def map2[EE >: E, B, C](b: Either[EE, B])(f: (A, B) => C): 
  Either[EE, C] = for { a <- this; b1 <- b } yield f(a,b1)
```

### Exercise 4.07

``` scala
def traverse[E,A,B](es: List[A])(f: A => Either[E, B]): Either[E, List[B]] = 
  es match {
    case Nil => Right(Nil)
    case h::t => (f(h) map2 traverse(t)(f))(_ :: _)
  }

def traverse_1[E,A,B](es: List[A])(f: A => Either[E, B]): Either[E, List[B]] = 
  es.foldRight[Either[E,List[B]]](Right(Nil))((a, b) => f(a).map2(b)(_ :: _))

def sequence[E,A](es: List[Either[E,A]]): Either[E,List[A]] = 
  traverse(es)(x => x)
```

### Exercise 4.08

There are a number of variations on `Option` and `Either`. If we want to accumulate multiple errors, a simple approach is a new data type that lets us keep a list of errors in the data constructor that represents failures:

``` scala
trait Partial[+A,+B]
case class Errors[+A](get: Seq[A]) extends Partial[A,Nothing]
case class Success[+B](get: B) extends Partial[Nothing,B]
```

There is a type very similar to this called `Validation` in the Scalaz library. You can implement `map`, `map2`, `sequence`, and so on for this type in such a way that errors are accumulated when possible (`flatMap` is unable to accumulate errors--can you see why?). This idea can even be generalized further--we don't need to accumulate failing values into a list; we can accumulate values using any user-supplied binary function. 

It's also possible to use `Either[List[E],_]` directly to accumulate errors, using different implementations of helper functions like `map2` and `sequence`.

## Answers to exercises for chapter 5

### Exercise 5.01

``` scala
// The natural recursive solution
def toListRecursive: List[A] = this match {
  case Cons(h,t) => h() :: t().toListRecursive
  case _ => List()
}
```

The above solution will stack overflow for large streams, since it's
not tail-recursive. Here is a tail-recursive implementation. At each
step we cons onto the front of the `acc` list, which will result in the
reverse of the stream. Then at the end we reverse the result to get the
correct order again.

``` scala
def toList: List[A] = {
  @annotation.tailrec
  def go(s: Stream[A], acc: List[A]): List[A] = s match {
    case Cons(h,t) => go(t(), h() :: acc)
    case _ => acc
  }
  go(this, List()).reverse
}
```

In order to avoid the `reverse` at the end, we could write it using a
mutable list buffer and an explicit loop instead. Note that the mutable
list buffer never escapes our `toList` method, so this function is
still _pure_.

``` scala
def toListFast: List[A] = {
  val buf = new collection.mutable.ListBuffer[A] 
  @annotation.tailrec
  def go(s: Stream[A]): List[A] = s match {
    case Cons(h,t) =>
      buf += h()
      go(t())
    case _ => buf.toList
  }
  go(this)
}
```

### Exercise 5.02

`take` first checks if n==0. In that case we need not look at the stream at all.

``` scala
def take(n: Int): Stream[A] = this match {
  case Cons(h, t) if n > 1 => cons(h(), t().take(n - 1))
  case Cons(h, _) if n == 1 => cons(h(), empty)
  case _ => empty
}
```

Unlike `take`, `drop` is not incremental. That is, it doesn't generate the
answer lazily. It must traverse the first `n` elements of the stream eagerly.

``` scala
@annotation.tailrec
final def drop(n: Int): Stream[A] = this match {
  case Cons(_, t) if n > 0 => t().drop(n - 1)
  case _ => this
}
```

### Exercise 5.03

It's a common Scala style to write method calls without `.` notation, as in `t() takeWhile f`.  

``` scala
def takeWhile(f: A => Boolean): Stream[A] = this match { 
  case Cons(h,t) if f(h()) => cons(h(), t() takeWhile f)
  case _ => empty 
}
```

### Exercise 5.04

Since `&&` is non-strict in its second argument, this terminates the traversal as soon as a nonmatching element is found. 

``` scala
def forAll(f: A => Boolean): Boolean =
  foldRight(true)((a,b) => f(a) && b)
```

### Exercise 5.05

``` scala
def takeWhile_1(f: A => Boolean): Stream[A] =
  foldRight(empty[A])((h,t) => 
    if (f(h)) cons(h,t)
    else      empty)
```

### Exercise 5.06

``` scala
def headOption: Option[A] =
  foldRight(None: Option[A])((h,_) => Some(h))
```

### Exercise 5.07

``` scala
def map[B](f: A => B): Stream[B] =
  foldRight(empty[B])((h,t) => cons(f(h), t)) 

def filter(f: A => Boolean): Stream[A] =
  foldRight(empty[A])((h,t) => 
    if (f(h)) cons(h, t)
    else t) 

def append[B>:A](s: => Stream[B]): Stream[B] = 
  foldRight(s)((h,t) => cons(h,t))

def flatMap[B](f: A => Stream[B]): Stream[B] = 
  foldRight(empty[B])((h,t) => f(h) append t)
```

### Exercise 5.08

``` scala
// This is more efficient than `cons(a, constant(a))` since it's just
// one object referencing itself.
def constant[A](a: A): Stream[A] = {
  lazy val tail: Stream[A] = Cons(() => a, () => tail) 
  tail
}
```

### Exercise 5.09

``` scala
def from(n: Int): Stream[Int] =
  cons(n, from(n+1))
```

### Exercise 5.10

``` scala
val fibs = {
  def go(f0: Int, f1: Int): Stream[Int] = 
    cons(f0, go(f1, f0+f1))
  go(0, 1)
}
```

### Exercise 5.11

``` scala
def unfold[A, S](z: S)(f: S => Option[(A, S)]): Stream[A] =
  f(z) match {
    case Some((h,s)) => cons(h, unfold(s)(f))
    case None => empty
  }
```

### Exercise 5.12

Scala provides shorter syntax when the first action of a function literal is to match on an expression.  The function passed to `unfold` in `fibsViaUnfold` is equivalent to `p => p match { case (f0,f1) => ... }`, but we avoid having to choose a name for `p`, only to pattern match on it.

``` scala
val fibsViaUnfold = 
  unfold((0,1)) { case (f0,f1) => Some((f0,(f1,f0+f1))) }

def fromViaUnfold(n: Int) = 
  unfold(n)(n => Some((n,n+1)))

def constantViaUnfold[A](a: A) = 
  unfold(a)(_ => Some((a,a)))

// could also of course be implemented as constant(1)
val onesViaUnfold = unfold(1)(_ => Some((1,1)))
```

### Exercise 5.13

``` scala
def mapViaUnfold[B](f: A => B): Stream[B] =
  unfold(this) {
    case Cons(h,t) => Some((f(h()), t()))
    case _ => None
  }

def takeViaUnfold(n: Int): Stream[A] =
  unfold((this,n)) {
    case (Cons(h,t), 1) => Some((h(), (empty, 0)))
    case (Cons(h,t), n) if n > 1 => Some((h(), (t(), n-1)))
    case _ => None
  }

def takeWhileViaUnfold(f: A => Boolean): Stream[A] =
  unfold(this) {
    case Cons(h,t) if f(h()) => Some((h(), t()))
    case _ => None
  }

def zipWith[B,C](s2: Stream[B])(f: (A,B) => C): Stream[C] =
  unfold((this, s2)) {
    case (Cons(h1,t1), Cons(h2,t2)) =>
      Some((f(h1(), h2()), (t1(), t2())))
    case _ => None
  }

// special case of `zip`
def zip[B](s2: Stream[B]): Stream[(A,B)] =
  zipWith(s2)((_,_))


def zipAll[B](s2: Stream[B]): Stream[(Option[A],Option[B])] =
  zipWithAll(s2)((_,_))

def zipWithAll[B, C](s2: Stream[B])(f: (Option[A], Option[B]) => C): Stream[C] =
  Stream.unfold((this, s2)) {
    case (Empty, Empty) => None
    case (Cons(h, t), Empty) => Some(f(Some(h()), Option.empty[B]) -> (t(), empty[B]))
    case (Empty, Cons(h, t)) => Some(f(Option.empty[A], Some(h())) -> (empty[A] -> t()))
    case (Cons(h1, t1), Cons(h2, t2)) => Some(f(Some(h1()), Some(h2())) -> (t1() -> t2()))
  }

```

### Exercise 5.14

`s startsWith s2` when corresponding elements of `s` and `s2` are all equal, until the point that `s2` is exhausted. If `s` is exhausted first, or we find an element that doesn't match, we terminate early. Using non-strictness, we can compose these three separate logical steps--the zipping, the termination when the second stream is exhausted, and the termination if a nonmatching element is found or the first stream is exhausted.

``` scala
def startsWith[A](s: Stream[A]): Boolean = 
  zipAll(s).takeWhile(!_._2.isEmpty) forAll {
    case (h,h2) => h == h2
  }
```

### Exercise 5.15

The last element of `tails` is always the empty `Stream`, so we handle this as a special case, by appending it to the output.

``` scala
def tails: Stream[Stream[A]] =
  unfold(this) {
    case Empty => None
    case s => Some((s, s drop 1))
  } append Stream(empty)

```

### Exercise 5.16

The function can't be implemented using `unfold`, since `unfold` generates elements of the `Stream` from left to right. It can be implemented using `foldRight` though.

The implementation is just a `foldRight` that keeps the accumulated value and the stream of intermediate results, which we `cons` onto during each iteration. When writing folds, it's common to have more state in the fold than is needed to compute the result. Here, we simply extract the accumulated list once finished.

``` scala
def scanRight[B](z: B)(f: (A, => B) => B): Stream[B] =
  foldRight((z, Stream(z)))((a, p0) => {
    // p0 is passed by-name and used in by-name args in f and cons.
    // So use lazy val to ensure only one evaluation...
    lazy val p1 = p0
    val b2 = f(a, p1._1)
    (b2, cons(b2, p1._2))
  })._2
```

## Answers to exercises for chapter 6

### Exercise 6.01

We need to be quite careful not to skew the generator.
Since `Int.Minvalue` is 1 smaller than `-(Int.MaxValue)`,
it suffices to increment the negative numbers by 1 and make them positive.
This maps Int.MinValue to Int.MaxValue and -1 to 0.

``` scala
def nonNegativeInt(rng: RNG): (Int, RNG) = {
  val (i, r) = rng.nextInt
  (if (i < 0) -(i + 1) else i, r)
}
```

### Exercise 6.02

We generate an integer >= 0 and divide it by one higher than the
maximum. This is just one possible solution.

``` scala
def double(rng: RNG): (Double, RNG) = {
  val (i, r) = nonNegativeInt(rng)
  (i / (Int.MaxValue.toDouble + 1), r)
}
```

### Exercise 6.03

``` scala
def intDouble(rng: RNG): ((Int, Double), RNG) = {
  val (i, r1) = rng.nextInt
  val (d, r2) = double(r1)
  ((i, d), r2)
}

def doubleInt(rng: RNG): ((Double, Int), RNG) = {
  val ((i, d), r) = intDouble(rng)
  ((d, i), r)
}

def double3(rng: RNG): ((Double, Double, Double), RNG) = {
  val (d1, r1) = double(rng)
  val (d2, r2) = double(r1)
  val (d3, r3) = double(r2)
  ((d1, d2, d3), r3)
}
```

There is something terribly repetitive about passing the RNG along
every time. What could we do to eliminate some of this duplication
of effort?

### Exercise 6.04

``` scala
// A simple recursive solution
def ints(count: Int)(rng: RNG): (List[Int], RNG) =
  if (count == 0) 
    (List(), rng)
  else {
    val (x, r1)  = rng.nextInt
    val (xs, r2) = ints(count - 1)(r1)
    (x :: xs, r2)
  }

// A tail-recursive solution
def ints2(count: Int)(rng: RNG): (List[Int], RNG) = {
  def go(count: Int, r: RNG, xs: List[Int]): (List[Int], RNG) =
    if (count == 0)
      (xs, r)
    else {
      val (x, r2) = r.nextInt
      go(count - 1, r2, x :: xs)
    }
  go(count, rng, List())
}
```

### Exercise 6.05

``` scala
val _double: Rand[Double] =
  map(nonNegativeInt)(_ / (Int.MaxValue.toDouble + 1))
```

### Exercise 6.06

This implementation of `map2` passes the initial `RNG` to the first argument
and the resulting `RNG` to the second argument. It's not necessarily wrong
to do this the other way around, since the results are random anyway.
We could even pass the initial `RNG` to both `f` and `g`, but that might
have unexpected results. E.g. if both arguments are `RNG.int` then we would
always get two of the same `Int` in the result. When implementing functions
like this, it's important to consider how we would test them for
correctness.

``` scala
def map2[A,B,C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
  rng => {
    val (a, r1) = ra(rng)
    val (b, r2) = rb(r1)
    (f(a, b), r2)
  }
```

### Exercise 6.07

In `sequence`, the base case of the fold is a `unit` action that returns
the empty list. At each step in the fold, we accumulate in `acc`
and `f` is the current element in the list.
`map2(f, acc)(_ :: _)` results in a value of type `Rand[List[A]]`
We map over that to prepend (cons) the element onto the accumulated list.

We are using `foldRight`. If we used `foldLeft` then the values in the
resulting list would appear in reverse order. It would be arguably better
to use `foldLeft` followed by `reverse`. What do you think?

``` scala
def sequence[A](fs: List[Rand[A]]): Rand[List[A]] =
  fs.foldRight(unit(List[A]()))((f, acc) => map2(f, acc)(_ :: _))
```

It's interesting that we never actually need to talk about the `RNG` value
in `sequence`. This is a strong hint that we could make this function
polymorphic in that type.

``` scala
def _ints(count: Int): Rand[List[Int]] =
  sequence(List.fill(count)(int))
```

### Exercise 6.08

``` scala
def flatMap[A,B](f: Rand[A])(g: A => Rand[B]): Rand[B] =
  rng => {
    val (a, r1) = f(rng)
    g(a)(r1) // We pass the new state along
  }

def nonNegativeLessThan(n: Int): Rand[Int] = {
  flatMap(nonNegativeInt) { i =>
    val mod = i % n
    if (i + (n-1) - mod >= 0) unit(mod) else nonNegativeLessThan(n)
  }
}
```

### Exercise 6.09

``` scala
def _map[A,B](s: Rand[A])(f: A => B): Rand[B] =
  flatMap(s)(a => unit(f(a)))

def _map2[A,B,C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
  flatMap(ra)(a => map(rb)(b => f(a, b)))

```

### Exercise 6.10

``` scala
case class State[S, +A](run: S => (A, S)) {
  def map[B](f: A => B): State[S, B] =
    flatMap(a => unit(f(a)))
  def map2[B,C](sb: State[S, B])(f: (A, B) => C): State[S, C] =
    flatMap(a => sb.map(b => f(a, b)))
  def flatMap[B](f: A => State[S, B]): State[S, B] = State(s => {
    val (a, s1) = run(s)
    f(a).run(s1)
  })
}

object State {

  def unit[S, A](a: A): State[S, A] =
    State(s => (a, s))

  // The idiomatic solution is expressed via foldRight
  def sequenceViaFoldRight[S,A](sas: List[State[S, A]]): State[S, List[A]] =
    sas.foldRight(unit[S, List[A]](List()))((f, acc) => f.map2(acc)(_ :: _))

  // This implementation uses a loop internally and is the same recursion
  // pattern as a left fold. It is quite common with left folds to build
  // up a list in reverse order, then reverse it at the end.
  // (We could also use a collection.mutable.ListBuffer internally.)
  def sequence[S, A](sas: List[State[S, A]]): State[S, List[A]] = {
    def go(s: S, actions: List[State[S,A]], acc: List[A]): (List[A],S) =
      actions match {
        case Nil => (acc.reverse,s)
        case h :: t => h.run(s) match { case (a,s2) => go(s2, t, a :: acc) }
      }
    State((s: S) => go(s,sas,List()))
  }

  // We can also write the loop using a left fold. This is tail recursive like 
  // the previous solution, but it reverses the list _before_ folding it instead
  // of after. You might think that this is slower than the `foldRight` solution
  // since it walks over the list twice, but it's actually faster! The
  // `foldRight` solution technically has to also walk the list twice, since it
  // has to unravel the call stack, not being tail recursive. And the call stack
  // will be as tall as the list is long.
  def sequenceViaFoldLeft[S,A](l: List[State[S, A]]): State[S, List[A]] =
    l.reverse.foldLeft(unit[S, List[A]](List())) { 
      (acc, f) => f.map2(acc)( _ :: _ )
    }

  def modify[S](f: S => S): State[S, Unit] = for {
    s <- get // Gets the current state and assigns it to `s`.
    _ <- set(f(s)) // Sets the new state to `f` applied to `s`.
  } yield ()

  def get[S]: State[S, S] = State(s => (s, s))

  def set[S](s: S): State[S, Unit] = State(_ => ((), s))
}
```

### Exercise 6.11

``` scala
sealed trait Input
case object Coin extends Input
case object Turn extends Input

case class Machine(locked: Boolean, candies: Int, coins: Int)

object Candy {
  def simulateMachine(inputs: List[Input]): State[Machine, (Int, Int)] = for {
    _ <- sequence(inputs.map(i => modify((s: Machine) => (i, s) match {
      case (_, Machine(_, 0, _)) => s
      case (Coin, Machine(false, _, _)) => s
      case (Turn, Machine(true, _, _)) => s
      case (Coin, Machine(true, candy, coin)) =>
        Machine(false, candy, coin + 1)
      case (Turn, Machine(false, candy, coin)) =>
        Machine(true, candy - 1, coin)
    })))
    s <- get
  } yield (s.coins, s.candies)
}

```

## Answers to exercises for chapter 7

### Exercise 7.01

``` scala
/* def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] */
```

### Exercise 7.02

Keep reading - we choose a representation for `Par` later in the chapter.

### Exercise 7.03

``` scala
/* This version respects timeouts. See `Map2Future` below. */
def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] =
  es => {
    val (af, bf) = (a(es), b(es))
    Map2Future(af, bf, f)
  }

/*
Note: this implementation will not prevent repeated evaluation if multiple
threads call `get` in parallel. We could prevent this using synchronization,
but it isn't needed for our purposes here (also, repeated evaluation of pure
values won't affect results).
*/
case class Map2Future[A,B,C](a: Future[A], b: Future[B],
                             f: (A,B) => C) extends Future[C] {
  @volatile var cache: Option[C] = None
  def isDone = cache.isDefined
  def isCancelled = a.isCancelled || b.isCancelled
  def cancel(evenIfRunning: Boolean) =
    a.cancel(evenIfRunning) || b.cancel(evenIfRunning)
  def get = compute(Long.MaxValue)
  def get(timeout: Long, units: TimeUnit): C =
    compute(TimeUnit.MILLISECONDS.convert(timeout, units))

  private def compute(timeoutMs: Long): C = cache match {
    case Some(c) => c
    case None =>
      val start = System.currentTimeMillis
      val ar = a.get(timeoutMs, TimeUnit.MILLISECONDS)
      val stop = System.currentTimeMillis; val at = stop-start
      val br = b.get(timeoutMs - at, TimeUnit.MILLISECONDS)
      val ret = f(ar, br)
      cache = Some(ret)
      ret
  }
}

```

### Exercise 7.04

``` scala
def asyncF[A,B](f: A => B): A => Par[B] =
  a => lazyUnit(f(a))
```

### Exercise 7.05

``` scala
def sequence_simple[A](l: List[Par[A]]): Par[List[A]] = 
  l.foldRight[Par[List[A]]](unit(List()))((h,t) => map2(h,t)(_ :: _))

// This implementation forks the recursive step off to a new logical thread,
// making it effectively tail-recursive. However, we are constructing
// a right-nested parallel program, and we can get better performance by 
// dividing the list in half, and running both halves in parallel. 
// See `sequenceBalanced` below.
def sequenceRight[A](as: List[Par[A]]): Par[List[A]] = 
  as match {
    case Nil => unit(Nil)
    case h :: t => map2(h, fork(sequence(t)))(_ :: _)
  }

// We define `sequenceBalanced` using `IndexedSeq`, which provides an 
// efficient function for splitting the sequence in half. 
def sequenceBalanced[A](as: IndexedSeq[Par[A]]): Par[IndexedSeq[A]] = fork {
  if (as.isEmpty) unit(Vector())
  else if (as.length == 1) map(as.head)(a => Vector(a))
  else {
    val (l,r) = as.splitAt(as.length/2)
    map2(sequenceBalanced(l), sequenceBalanced(r))(_ ++ _)
  }
}

def sequence[A](as: List[Par[A]]): Par[List[A]] =
  map(sequenceBalanced(as.toIndexedSeq))(_.toList)
```

### Exercise 7.06

``` scala
def parFilter[A](l: List[A])(f: A => Boolean): Par[List[A]] = {
  val pars: List[Par[List[A]]] = 
    l map (asyncF((a: A) => if (f(a)) List(a) else List())) 
  // flatten is a convenience method on `List` for concatenating a list of lists
  map(sequence(pars))(_.flatten)
}
```

### Exercise 7.07

See https://github.com/quchen/articles/blob/master/second_functor_law.md

Also https://gist.github.com/pchiusano/444de1f222f1ceb09596

### Exercise 7.08

Keep reading. The issue is explained in the next paragraph.

### Exercise 7.09

For a thread pool of size 2, `fork(fork(fork(x)))` will deadlock, and so on. Another, perhaps more interesting example is `fork(map2(fork(x), fork(y)))`. In this case, the outer task is submitted first and occupies a thread waiting for both `fork(x)` and `fork(y)`. The `fork(x)` and `fork(y)` tasks are submitted and run in parallel, except that only one thread is available, resulting in deadlock. 

### Exercise 7.10

We give a fully fleshed-out solution in the `Task` data type in the code for Chapter 13.

### Exercise 7.11

``` scala
def choiceN[A](n: Par[Int])(choices: List[Par[A]]): Par[A] = 
  es => {
    val ind = run(es)(n).get // Full source files
    run(es)(choices(ind))
  }

def choiceViaChoiceN[A](a: Par[Boolean])(ifTrue: Par[A], ifFalse: Par[A]): Par[A] =
  choiceN(map(a)(b => if (b) 0 else 1))(List(ifTrue, ifFalse))
```

### Exercise 7.12

``` scala
def choiceMap[K,V](key: Par[K])(choices: Map[K,Par[V]]): Par[V] =
  es => {
    val k = run(es)(key).get
    run(es)(choices(k))
  }
```

### Exercise 7.13

``` scala
def chooser[A,B](p: Par[A])(choices: A => Par[B]): Par[B] = 
  es => {
    val k = run(es)(p).get
    run(es)(choices(k))
  }

/* `chooser` is usually called `flatMap` or `bind`. */
def flatMap[A,B](p: Par[A])(choices: A => Par[B]): Par[B] = 
  es => {
    val k = run(es)(p).get
    run(es)(choices(k))
  }

def choiceViaFlatMap[A](p: Par[Boolean])(f: Par[A], t: Par[A]): Par[A] =
  flatMap(p)(b => if (b) t else f)

def choiceNViaFlatMap[A](p: Par[Int])(choices: List[Par[A]]): Par[A] =
  flatMap(p)(i => choices(i))
```

### Exercise 7.14

``` scala
// see nonblocking implementation in `Nonblocking.scala`
def join[A](a: Par[Par[A]]): Par[A] = 
  es => run(es)(run(es)(a).get())

def joinViaFlatMap[A](a: Par[Par[A]]): Par[A] = 
  flatMap(a)(x => x)

def flatMapViaJoin[A,B](p: Par[A])(f: A => Par[B]): Par[B] = 
  join(map(p)(f))
```

### Exercise 7.15

``` scala
def flatMap[B](f: A => Par[B]): Par[B] = 
  Par.flatMap(p)(f)

def map[B](f: A => B): Par[B] = 
  Par.map(p)(f)

def map2[B,C](p2: Par[B])(f: (A,B) => C): Par[C] =
  Par.map2(p,p2)(f)

def zip[B](p2: Par[B]): Par[(A,B)] = 
  p.map2(p2)((_,_))
```

### Exercise 7.16

``` scala
def flatMap[B](f: A => Par[B]): Par[B] = 
  Par.flatMap(p)(f)

def map[B](f: A => B): Par[B] = 
  Par.map(p)(f)

def map2[B,C](p2: Par[B])(f: (A,B) => C): Par[C] =
  Par.map2(p,p2)(f)

def zip[B](p2: Par[B]): Par[(A,B)] = 
  p.map2(p2)((_,_))
```

### Exercise 7.17

This implementation is not safe for execution on bounded thread pools, and it also does not preserve timeouts. Can you see why? You may wish to try implementing a nonblocking version like was done for `fork`.  

``` scala
def join[A](a: Par[Par[A]]): Par[A] = 
  es => a(es).get.apply(es)

def joinViaFlatMap[A](a: Par[Par[A]]): Par[A] = 
  flatMap(a)(a => a)

def flatMapViaJoin[A,B](p: Par[A])(f: A => Par[B]): Par[B] = 
  join(map(p)(f))
```

## Answers to exercises for chapter 8

### Exercise 8.01

Here are a few properties:

* The sum of the empty list is 0.
* The sum of a list whose elements are all equal to `x` is just the list's length multiplied by `x`. We might express this as `sum(List.fill(n)(x)) == n*x`
* For any list, `l`, `sum(l) == sum(l.reverse)`, since addition is commutative.
* Given a list, `List(x,y,z,p,q)`, `sum(List(x,y,z,p,q)) == sum(List(x,y)) + sum(List(z,p,q))`, since addition is associative. More generally, we can partition a list into two subsequences whose sum is equal to the sum of the overall list.
* The sum of 1,2,3...n is `n*(n+1)/2`.

### Exercise 8.02

* The max of a single element list is equal to that element.
* The max of a list is greater than or equal to all elements of the list.
* The max of a list is an element of that list.
* The max of the empty list is unspecified and should throw an error or return `None`.

### Exercise 8.03

``` scala
/* We can refer to the enclosing `Prop` instance with `Prop.this` */
def &&(p: Prop): Prop = new Prop {
  def check = Prop.this.check && p.check
}

```

### Exercise 8.04

``` scala
def choose(start: Int, stopExclusive: Int): Gen[Int] =
  Gen(State(RNG.nonNegativeInt).map(n => start + n % (stopExclusive-start)))

/* We could write this as an explicit state action, but this is far less
   convenient, since it requires us to manually thread the `RNG` through the
   computation. */
def choose2(start: Int, stopExclusive: Int): Gen[Int] = 
  Gen(State(rng => RNG.nonNegativeInt(rng) match {
    case (n,rng2) => (start + n % (stopExclusive-start), rng2)
  }))
```

### Exercise 8.05

``` scala
def unit[A](a: => A): Gen[A] =
  Gen(State.unit(a))

def boolean: Gen[Boolean] =
  Gen(State(RNG.boolean))

def choose(start: Int, stopExclusive: Int): Gen[Int] =
  Gen(State(RNG.nonNegativeInt).map(n => start + n % (stopExclusive-start)))

def listOfN[A](n: Int, g: Gen[A]): Gen[List[A]] =
  Gen(State.sequence(List.fill(n)(g.sample)))
```

### Exercise 8.06

These methods are defined in the `Gen` class:

``` scala
def flatMap[B](f: A => Gen[B]): Gen[B] =
  Gen(sample.flatMap(a => f(a).sample))

/* A method alias for the function we wrote earlier. */
def listOfN(size: Int): Gen[List[A]] =
  Gen.listOfN(size, this)

/* A version of `listOfN` that generates the size to use dynamically. */
def listOfN(size: Gen[Int]): Gen[List[A]] =
  size flatMap (n => this.listOfN(n))
```

### Exercise 8.07

``` scala
def union[A](g1: Gen[A], g2: Gen[A]): Gen[A] = 
  boolean.flatMap(b => if (b) g1 else g2)
```

### Exercise 8.08

``` scala
def weighted[A](g1: (Gen[A],Double), g2: (Gen[A],Double)): Gen[A] = {
  /* The probability we should pull from `g1`. */
  val g1Threshold = g1._2.abs / (g1._2.abs + g2._2.abs)

  Gen(State(RNG.double).flatMap(d =>
    if (d < g1Threshold) g1._1.sample else g2._1.sample))
}
```

### Exercise 8.09

``` scala
def &&(p: Prop) = Prop {
  (max,n,rng) => run(max,n,rng) match {
    case Passed => p.run(max, n, rng)
    case x => x
  }
}

def ||(p: Prop) = Prop {
  (max,n,rng) => run(max,n,rng) match {
    // In case of failure, run the other prop.
    case Falsified(msg, _) => p.tag(msg).run(max,n,rng)
    case x => x
  }
}

/* This is rather simplistic - in the event of failure, we simply prepend
 * the given message on a newline in front of the existing message.
 */
def tag(msg: String) = Prop {
  (max,n,rng) => run(max,n,rng) match {
    case Falsified(e, c) => Falsified(msg + "\n" + e, c)
    case x => x
  }
}
```

### Exercise 8.10

``` scala
def unsized: SGen[A] = SGen(_ => this)
```

### Exercise 8.11

``` scala
case class SGen[+A](g: Int => Gen[A]) {
  def apply(n: Int): Gen[A] = g(n)

  def map[B](f: A => B): SGen[B] =
    SGen(g andThen (_ map f))

  def flatMap[B](f: A => Gen[B]): SGen[B] =
    SGen(g andThen (_ flatMap f))

  def **[B](s2: SGen[B]): SGen[(A,B)] =
    SGen(n => apply(n) ** s2(n))
}
```

### Exercise 8.12

``` scala
def listOf[A](g: Gen[A]): SGen[List[A]] = 
  SGen(n => g.listOfN(n))
```

### Exercise 8.13

``` scala
def listOf1[A](g: Gen[A]): SGen[List[A]] =
  SGen(n => g.listOfN(n max 1))
    
val maxProp1 = forAll(listOf1(smallInt)) { l => 
  val max = l.max
  !l.exists(_ > max) // No value greater than `max` should exist in `l`
}
```

### Exercise 8.14

``` scala
val sortedProp = forAll(listOf(smallInt)) { ns =>
  val nss = ns.sorted
  // We specify that every sorted list is either empty, has one element,
  // or has no two consecutive elements `(a,b)` such that `a` is greater than `b`.
  (ns.isEmpty || nss.tail.isEmpty || !ns.zip(ns.tail).exists {
    case (a,b) => a > b
  })
    // Also, the sorted list should have all the elements of the input list,
    && !ns.exists(!nss.contains(_))
    // and it should have no elements not in the input list.
    && !nss.exists(!ns.contains(_))
}
```

### Exercise 8.15

A detailed answer is to be found in the file Exhaustive.scala in the code accompanying this chapter.

### Exercise 8.16

``` scala
/* A `Gen[Par[Int]]` generated from a list summation that spawns a new parallel
 * computation for each element of the input list summed to produce the final 
 * result. This is not the most compelling example, but it provides at least some 
 * variation in structure to use for testing. 
 */
val pint2: Gen[Par[Int]] = choose(-100,100).listOfN(choose(0,20)).map(l => 
  l.foldLeft(Par.unit(0))((p,i) => 
    Par.fork { Par.map2(p, Par.unit(i))(_ + _) }))
```

### Exercise 8.17

``` scala
val forkProp = Prop.forAllPar(pint2)(i => equal(Par.fork(i), i)) tag "fork"
```

### Exercise 8.18

* `l.takeWhile(f) ++ l.dropWhile(f) == l`
* We want to enforce that `takeWhile` returns the _longest_ prefix whose elements satisfy the predicate. There are various ways to state this, but the general idea is that the remaining list, if non-empty, should start with an element that does _not_ satisfy the predicate.

## Answers to exercises for chapter 9

### Exercise 9.01

``` scala
def map2[A,B,C](p: Parser[A], p2: Parser[B])(
                f: (A,B) => C): Parser[C] = 
  map(product(p, p2))(f.tupled)
def many1[A](p: Parser[A]): Parser[List[A]] = 
  map2(p, many(p))(_ :: _)
```

### Exercise 9.02

`product` is associative. These two expressions are "roughly" equal:

``` scala
(a ** b) ** c
a ** (b ** c)
```

The only difference is how the pairs are nested. The `(a ** b) ** c` parser returns an `((A,B), C)`, whereas the `a ** (b ** c)` returns an `(A, (B,C))`. We can define functions `unbiasL` and `unbiasR` to convert these nested tuples to flat 3-tuples:

``` scala
def unbiasL[A,B,C](p: ((A,B), C)): (A,B,C) = (p._1._1, p._1._2, p._2)
def unbiasR[A,B,C](p: (A, (B,C))): (A,B,C) = (p._1, p._2._1, p._2._2)
```

With these, we can now state the associativity property:

``` scala
(a ** b) ** c map (unbiasL) == a ** (b ** c) map (unbiasR)
```

We'll sometimes just use `~=` when there is an obvious bijection between the two sides:

``` scala
(a ** b) ** c ~= a ** (b ** c)
```

`map` and `product` also have an interesting relationship--we can `map` either before or after taking the product of two parsers, without affecting the behavior:

``` scala
a.map(f) ** b.map(g) == (a ** b) map { case (a,b) => (f(a), g(b)) }
```

For instance, if `a` and `b` were both `Parser[String]`, and `f` and `g` both computed the length of a string, it doesn't matter if we map over the result of `a` to compute its length, or whether we do that _after_ the product.

See chapter 12 for more discussion of these laws.

### Exercise 9.03

``` scala
def many[A](p: Parser[A]): Parser[List[A]] =
  map2(p, many(p))(_ :: _) or succeed(List())
```

### Exercise 9.04

``` scala
def listOfN[A](n: Int, p: Parser[A]): Parser[List[A]] =
  if (n <= 0) succeed(List())
  else map2(p, listOfN(n-1, p))(_ :: _)
```

### Exercise 9.05

We could introduce a combinator, `wrap`:

``` scala
def wrap[A](p: => Parser[A]): Parser[A]
```  

Then define `many` as:

``` scala
def many[A](p: Parser[A]): Parser[List[A]] =
  map2(p, wrap(many(p)))(_ :: _) or succeed(List())
```

In the parallelism chapter, we were particularly interested in avoiding having `Par` objects that took as much time and space to build as the corresponding serial computation, and the `delay` combinator let us control this more carefully. Here, this isn't as much of a concern, and having to think carefully each time we `map2` to decide whether we need to call `wrap` seems like unnecessary friction for users of the API.

### Exercise 9.06

We'll just have this parser return the number of `'a'` characters read. Note that we can declare a normal value inside a for-comprehension.

``` scala
for {
  digit <- "[0-9]+".r
  val n = digit.toInt
  // we really should catch exceptions thrown by toInt
  // and convert to parse failure
  _ <- listOfN(n, char('a'))
} yield n
```

### Exercise 9.07

These can be implemented using a for-comprehension, which delegates to the `flatMap` and `map` implementations we've provided on `ParserOps`, or they can be implemented in terms of these functions directly. 

``` scala
def product[A,B](p: Parser[A], p2: => Parser[B]): Parser[(A,B)] = 
  flatMap(p)(a => map(p2)(b => (a,b)))

def map2[A,B,C](p: Parser[A], p2: => Parser[B])(f: (A,B) => C): Parser[C] = 
  for { a <- p; b <- p2 } yield f(a,b)
```

### Exercise 9.08

``` scala
def map[A,B](p: Parser[A])(f: A => B): Parser[B] = p.flatMap(a => succeed(f(a)))
```

### Exercise 9.09

See full implementation in [JSON.scala](https://github.com/fpinscala/fpinscala/blob/master/answers/src/main/scala/fpinscala/parsing/JSON.scala).

### Exercise 9.10

The next section works through a possible design in detail.

### Exercise 9.11

``` scala
/**
 * In the event of an error, returns the error that occurred after consuming
 * the most number of characters.
 */
def furthest[A](p: Parser[A]): Parser[A]

/** In the event of an error, returns the error that occurred most recently. */
def latest[A](p: Parser[A]): Parser[A]

```

### Exercise 9.12

Keep reading for a discussion

### Exercise 9.13

There is a completed reference implementation of `Parsers` in [parsing/instances/Reference.scala](https://github.com/fpinscala/fpinscala/blob/master/answers/src/main/scala/fpinscala/parsing/instances/Reference.scala), though we recommend you continue reading before looking at that, since we're still refining the representation.

### Exercise 9.14

Again, see `Reference.scala` for a fully worked implementation.

### Exercise 9.15

Again see the reference implementation.

### Exercise 9.16

``` scala
case class ParseError(stack: List[(Location,String)] = List()) {
  def push(loc: Location, msg: String): ParseError =
    copy(stack = (loc,msg) :: stack)

  def label[A](s: String): ParseError =
    ParseError(latestLoc.map((_,s)).toList)

  def latest: Option[(Location,String)] =
    stack.lastOption

  def latestLoc: Option[Location] =
    latest map (_._1)

  /**
  Display collapsed error stack - any adjacent stack elements with the
  same location are combined on one line. For the bottommost error, we
  display the full line, with a caret pointing to the column of the error.
  Example:

  1.1 file 'companies.json'; array
  5.1 object
  5.2 key-value
  5.10 ':'

  { "MSFT" ; 24,
           ^
  */
  override def toString =
    if (stack.isEmpty) "no error message"
    else {
      val collapsed = collapseStack(stack)
      val context =
        collapsed.lastOption.map("\n\n" + _._1.currentLine).getOrElse("") +
        collapsed.lastOption.map("\n" + _._1.columnCaret).getOrElse("")
      collapsed.map {
        case (loc,msg) => loc.line.toString + "." + loc.col + " " + msg
      }.mkString("\n") + context
    }

  /* Builds a collapsed version of the given error stack -
   * messages at the same location have their messages merged,
   * separated by semicolons */
  def collapseStack(s: List[(Location,String)]): List[(Location,String)] =
    s.groupBy(_._1).
      mapValues(_.map(_._2).mkString("; ")).
      toList.sortBy(_._1.offset)

  def formatLoc(l: Location): String = l.line + "." + l.col
}

```

### Exercise 9.17

See a fully worked implementation in `instances/Sliceable.scala` in the answers.

### Exercise 9.18

We'll just give a sketch here. The basic idea is to add an additional field to `ParseError`:

``` scala
case class ParseError(stack: List[(Location,String)] = List(),
                      otherFailures: List[ParseError] = List()) {

  def addFailure(e: ParseError): ParseError =
    this.copy(otherFailures = e :: this.otherFailures)
  ...
}
```

We then need to make sure we populate this in the implementation of `or`

``` scala
def or[A](p: Parser[A], p2: => Parser[A]): Parser[A] =
  s => p(s) match {
    case Failure(e,false) => p2(s).mapError(_.addFailure(e))
    case r => r // committed failure or success skips running `p2`
  }
```

Of course, we have to decide how to print a `ParseError` for human consumption
We also can expose combinators for selecting which error(s) get reported in the
event that a chain of `a | b | c` fails--we might choose to collect up all the
errors for each of the three parsers, or perhaps only show the parser that got
the furthest in the input before failing, etc

## Answers to exercises for chapter 10

### Exercise 10.01

``` scala
val intAddition: Monoid[Int] = new Monoid[Int] {
  def op(x: Int, y: Int) = x + y
  val zero = 0
}

val intMultiplication: Monoid[Int] = new Monoid[Int] {
  def op(x: Int, y: Int) = x * y
  val zero = 1
}

val booleanOr: Monoid[Boolean] = new Monoid[Boolean] {
  def op(x: Boolean, y: Boolean) = x || y
  val zero = false
}

val booleanAnd: Monoid[Boolean] = new Monoid[Boolean] {
  def op(x: Boolean, y: Boolean) = x && y
  val zero = true
}
```

### Exercise 10.02

Notice that we have a choice in how we implement `op`.
We can compose the options in either order. Both of those implementations
satisfy the monoid laws, but they are not equivalent.
This is true in general--that is, every monoid has a _dual_ where the
`op` combines things in the opposite order. Monoids like `booleanOr` and
`intAddition` are equivalent to their duals because their `op` is commutative
as well as associative.

``` scala
def optionMonoid[A]: Monoid[Option[A]] = new Monoid[Option[A]] {
  def op(x: Option[A], y: Option[A]) = x orElse y
  val zero = None
}

// We can get the dual of any monoid just by flipping the `op`.
def dual[A](m: Monoid[A]): Monoid[A] = new Monoid[A] {
  def op(x: A, y: A): A = m.op(y, x)
  val zero = m.zero
}

// Now we can have both monoids on hand:
def firstOptionMonoid[A]: Monoid[Option[A]] = optionMonoid[A]
def lastOptionMonoid[A]: Monoid[Option[A]] = dual(firstOptionMonoid)
```

### Exercise 10.03

There is a choice of implementation here as well.
Do we implement it as `f compose g` or `f andThen g`? We have to pick one.
We can then get the other one using the `dual` construct (see previous answer).

``` scala
def endoMonoid[A]: Monoid[A => A] = new Monoid[A => A] {
  def op(f: A => A, g: A => A) = f compose g
  val zero = (a: A) => a
}
```

### Exercise 10.04

``` scala
import fpinscala.testing._
import Prop._

def monoidLaws[A](m: Monoid[A], gen: Gen[A]): Prop =
  // Associativity
  forAll(for {
    x <- gen
    y <- gen
    z <- gen
  } yield (x, y, z))(p =>
    m.op(p._1, m.op(p._2, p._3)) == m.op(m.op(p._1, p._2), p._3)) &&
  // Identity
  forAll(gen)((a: A) =>
    m.op(a, m.zero) == a && m.op(m.zero, a) == a)
```

### Exercise 10.05

Notice that this function does not require the use of `map` at all.

``` scala
def foldMap[A, B](as: List[A], m: Monoid[B])(f: A => B): B =
  as.foldLeft(m.zero)((b, a) => m.op(b, f(a)))
```

### Exercise 10.06

The function type `(A, B) => B`, when curried, is `A => (B => B)`.
And of course, `B => B` is a monoid for any `B` (via function composition).

``` scala
def foldRight[A, B](as: List[A])(z: B)(f: (A, B) => B): B =
  foldMap(as, endoMonoid[B])(f.curried)(z)
```

Folding to the left is the same except we flip the arguments to
the function `f` to put the `B` on the correct side.
Then we have to also "flip" the monoid so that it operates from left to right.

``` scala
def foldLeft[A, B](as: List[A])(z: B)(f: (B, A) => B): B =
  foldMap(as, dual(endoMonoid[B]))(a => b => f(b, a))(z)
```

### Exercise 10.07

``` scala
def foldMapV[A, B](as: IndexedSeq[A], m: Monoid[B])(f: A => B): B =
  if (as.length == 0)
    m.zero
  else if (as.length == 1)
    f(as(0))
  else {
    val (l, r) = as.splitAt(as.length / 2)
    m.op(foldMapV(l, m)(f), foldMapV(r, m)(f))
  }
```

### Exercise 10.08

``` scala
// This ability to 'lift' a monoid any monoid to operate within
// some context (here `Par`) is something we'll discuss more in 
// chapters 11 & 12
def par[A](m: Monoid[A]): Monoid[Par[A]] = new Monoid[Par[A]] {
  def zero = Par.unit(m.zero)  
  def op(a: Par[A], b: Par[A]) = a.map2(b)(m.op)
}

// we perform the mapping and the reducing both in parallel
def parFoldMap[A,B](v: IndexedSeq[A], m: Monoid[B])(f: A => B): Par[B] = 
  Par.parMap(v)(f).flatMap { bs => 
    foldMapV(bs, par(m))(b => Par.async(b)) 
  }
```

### Exercise 10.09

``` scala
// This implementation detects only ascending order,
// but you can write a monoid that detects both ascending and descending
// order if you like.
def ordered(ints: IndexedSeq[Int]): Boolean = {
  // Our monoid tracks the minimum and maximum element seen so far
  // as well as whether the elements are so far ordered.
  val mon = new Monoid[Option[(Int, Int, Boolean)]] {
    def op(o1: Option[(Int, Int, Boolean)], o2: Option[(Int, Int, Boolean)]) =
      (o1, o2) match {
        // The ranges should not overlap if the sequence is ordered.
        case (Some((x1, y1, p)), Some((x2, y2, q))) =>
          Some((x1 min x2, y1 max y2, p && q && y1 <= x2))
        case (x, None) => x
        case (None, x) => x
      }
    val zero = None
  }
  // The empty sequence is ordered, and each element by itself is ordered.
  foldMapV(ints, mon)(i => Some((i, i, true))).map(_._3).getOrElse(true)
}
```

### Exercise 10.10

``` scala
sealed trait WC
case class Stub(chars: String) extends WC
case class Part(lStub: String, words: Int, rStub: String) extends WC

val wcMonoid: Monoid[WC] = new Monoid[WC] {
  // The empty result, where we haven't seen any characters yet.
  val zero = Stub("")

  def op(a: WC, b: WC) = (a, b) match {
    case (Stub(c), Stub(d)) => Stub(c + d)
    case (Stub(c), Part(l, w, r)) => Part(c + l, w, r)
    case (Part(l, w, r), Stub(c)) => Part(l, w, r + c)
    case (Part(l1, w1, r1), Part(l2, w2, r2)) =>
      Part(l1, w1 + (if ((r1 + l2).isEmpty) 0 else 1) + w2, r2)
  }
}
```

### Exercise 10.11

``` scala
def count(s: String): Int = {
  def wc(c: Char): WC =
    if (c.isWhitespace)
      Part("", 0, "")
    else
      Stub(c.toString)
  def unstub(s: String) = s.length min 1
  foldMapV(s.toIndexedSeq, wcMonoid)(wc) match {
    case Stub(s) => unstub(s)
    case Part(l, w, r) => unstub(l) + w + unstub(r)
  }
}
```

### Exercise 10.12

``` scala
trait Foldable[F[_]] {
  def foldRight[A, B](as: F[A])(z: B)(f: (A, B) => B): B =
    foldMap(as)(f.curried)(endoMonoid[B])(z)

  def foldLeft[A, B](as: F[A])(z: B)(f: (B, A) => B): B =
    foldMap(as)(a => (b: B) => f(b, a))(dual(endoMonoid[B]))(z)

  def foldMap[A, B](as: F[A])(f: A => B)(mb: Monoid[B]): B =
    foldRight(as)(mb.zero)((a, b) => mb.op(f(a), b))

  def concatenate[A](as: F[A])(m: Monoid[A]): A =
    foldLeft(as)(m.zero)(m.op)
}

object ListFoldable extends Foldable[List] {
  override def foldRight[A, B](as: List[A])(z: B)(f: (A, B) => B) =
    as.foldRight(z)(f)
  override def foldLeft[A, B](as: List[A])(z: B)(f: (B, A) => B) =
    as.foldLeft(z)(f)
  override def foldMap[A, B](as: List[A])(f: A => B)(mb: Monoid[B]): B =
    foldLeft(as)(mb.zero)((b, a) => mb.op(b, f(a)))
}

object IndexedSeqFoldable extends Foldable[IndexedSeq] {
  import Monoid._
  override def foldRight[A, B](as: IndexedSeq[A])(z: B)(f: (A, B) => B) =
    as.foldRight(z)(f)
  override def foldLeft[A, B](as: IndexedSeq[A])(z: B)(f: (B, A) => B) =
    as.foldLeft(z)(f)
  override def foldMap[A, B](as: IndexedSeq[A])(f: A => B)(mb: Monoid[B]): B =
    foldMapV(as, mb)(f)
}

object StreamFoldable extends Foldable[Stream] {
  override def foldRight[A, B](as: Stream[A])(z: B)(f: (A, B) => B) =
    as.foldRight(z)(f)
  override def foldLeft[A, B](as: Stream[A])(z: B)(f: (B, A) => B) =
    as.foldLeft(z)(f)
}
```

### Exercise 10.13

``` scala
sealed trait Tree[+A]
case class Leaf[A](value: A) extends Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]

object TreeFoldable extends Foldable[Tree] {
  override def foldMap[A, B](as: Tree[A])(f: A => B)(mb: Monoid[B]): B =
    as match {
      case Leaf(a) => f(a)
      case Branch(l, r) => mb.op(foldMap(l)(f)(mb), foldMap(r)(f)(mb))
    }
  override def foldLeft[A, B](as: Tree[A])(z: B)(f: (B, A) => B) = as match {
    case Leaf(a) => f(z, a)
    case Branch(l, r) => foldLeft(r)(foldLeft(l)(z)(f))(f)
  }
  override def foldRight[A, B](as: Tree[A])(z: B)(f: (A, B) => B) = as match {
    case Leaf(a) => f(a, z)
    case Branch(l, r) => foldRight(l)(foldRight(r)(z)(f))(f)
  }
}
```

Notice that in `TreeFoldable.foldMap`, we don't actually use the `zero`
from the `Monoid`. This is because there is no empty tree.
This suggests that there might be a class of types that are foldable
with something "smaller" than a monoid, consisting only of an
associative `op`. That kind of object (a monoid without a `zero`) is
called a *semigroup*.

### Exercise 10.14

``` scala
object OptionFoldable extends Foldable[Option] {
  override def foldMap[A, B](as: Option[A])(f: A => B)(mb: Monoid[B]): B =
    as match {
      case None => mb.zero
      case Some(a) => f(a)
    }
  override def foldLeft[A, B](as: Option[A])(z: B)(f: (B, A) => B) = as match {
    case None => z
    case Some(a) => f(z, a)
  }
  override def foldRight[A, B](as: Option[A])(z: B)(f: (A, B) => B) = as match {
    case None => z
    case Some(a) => f(a, z)
  }
}
```

### Exercise 10.15

``` scala
def toList[A](as: F[A]): List[A] =
  foldRight(as)(List[A]())(_ :: _)
```

### Exercise 10.16

``` scala
def productMonoid[A,B](A: Monoid[A], B: Monoid[B]): Monoid[(A, B)] =
  new Monoid[(A, B)] {
    def op(x: (A, B), y: (A, B)) =
      (A.op(x._1, y._1), B.op(x._2, y._2))
    val zero = (A.zero, B.zero)
  }
```

### Exercise 10.17

``` scala
def functionMonoid[A,B](B: Monoid[B]): Monoid[A => B] =
  new Monoid[A => B] {
    def op(f: A => B, g: A => B) = a => B.op(f(a), g(a))
    val zero: A => B = a => B.zero
  }
```

### Exercise 10.18

``` scala
def mapMergeMonoid[K,V](V: Monoid[V]): Monoid[Map[K, V]] =
  new Monoid[Map[K, V]] {
    def zero = Map[K,V]()
    def op(a: Map[K, V], b: Map[K, V]) =
      (a.keySet ++ b.keySet).foldLeft(zero) { (acc,k) =>
        acc.updated(k, V.op(a.getOrElse(k, V.zero),
                            b.getOrElse(k, V.zero)))
      }
  }

def bag[A](as: IndexedSeq[A]): Map[A, Int] =
  foldMapV(as, mapMergeMonoid[A, Int](intAddition))((a: A) => Map(a -> 1))

```

## Answers to exercises for chapter 11

### Exercise 11.01

``` scala
val parMonad = new Monad[Par] {
  def unit[A](a: => A) = Par.unit(a)
  def flatMap[A,B](ma: Par[A])(f: A => Par[B]) = Par.flatMap(ma)(f)
}

def parserMonad[P[+_]](p: Parsers[P]) = new Monad[P] {
  def unit[A](a: => A) = p.succeed(a)
  def flatMap[A,B](ma: P[A])(f: A => P[B]) = p.flatMap(ma)(f)
}

val optionMonad = new Monad[Option] {
  def unit[A](a: => A) = Some(a)
  def flatMap[A,B](ma: Option[A])(f: A => Option[B]) = ma flatMap f
}

val streamMonad = new Monad[Stream] {
  def unit[A](a: => A) = Stream(a)
  def flatMap[A,B](ma: Stream[A])(f: A => Stream[B]) = ma flatMap f
}

val listMonad = new Monad[List] {
  def unit[A](a: => A) = List(a)
  def flatMap[A,B](ma: List[A])(f: A => List[B]) = ma flatMap f
}
```

### Exercise 11.02

Since `State` is a binary type constructor, we need to partially apply it
with the `S` type argument. Thus, it is not just one monad, but an entire
family of monads, one for each type `S`. One solution is to create a class
`StateMonads` that accepts the `S` type argument and then has a _type member_
for the fully applied `State[S, A]` type inside:

``` scala
class StateMonads[S] {
  type StateS[A] = State[S, A]

  // We can then declare the monad for the `StateS` type constructor:
  val monad = new Monad[StateS] {
    def unit[A](a: => A): State[S, A] = State(s => (a, s))
    override def flatMap[A,B](st: State[S, A])(f: A => State[S, B]): State[S, B] =
      st flatMap f
  }
}
```

But we don't have to create a full class like `StateMonads`. We can create
an anonymous class inline, inside parentheses, and project out its type member `f`.
This is sometimes called a "type lambda", since it's very similar to a type-level
anonymous function.

``` scala
def stateMonad[S] = new Monad[({type f[x] = State[S, x]})#f] {
  def unit[A](a: => A): State[S, A] = State(s => (a, s))
  override def flatMap[A,B](st: State[S, A])(f: A => State[S, B]): State[S, B] =
    st flatMap f
}
```

### Exercise 11.03

``` scala
def sequence[A](lma: List[F[A]]): F[List[A]] =
  lma.foldRight(unit(List[A]()))((ma, mla) => map2(ma, mla)(_ :: _))

def traverse[A,B](la: List[A])(f: A => F[B]): F[List[B]] =
  la.foldRight(unit(List[B]()))((a, mlb) => map2(f(a), mlb)(_ :: _))

/** 
 * 'Balanced' sequencing, which should behave like `sequence`,
 * but it can use less stack for some data types. We'll see later
 * in this chapter how the monad _laws_ let us conclude both 
 * definitions 'mean' the same thing.
 */
def bsequence[A](ms: Seq[F[A]]): F[IndexedSeq[A]] = {
  if (ms.isEmpty) point(Vector())
  else if (ms.size == 1) ms.head.map(Vector(_))
  else {
    val (l,r) = ms.toIndexedSeq.splitAt(ms.length / 2)
    map2(bsequence(l), bsequence(r))(_ ++ _)
  }
}
```

### Exercise 11.04

``` scala
// Recursive version:
def _replicateM[A](n: Int, ma: F[A]): F[List[A]] =
  if (n <= 0) unit(List[A]()) else map2(ma, replicateM(n - 1, ma))(_ :: _)

// Using `sequence` and the `List.fill` function of the standard library:
def replicateM[A](n: Int, ma: F[A]): F[List[A]] =
  sequence(List.fill(n)(ma))
```

### Exercise 11.05

For `List`, the `replicateM` function will generate a list of lists. It will contain all the lists of length `n` with elements selected from the input list.

For `Option`, it will generate either `Some` or `None` based on whether the input is `Some` or `None`. The `Some` case will contain a list of length `n` that repeats the element in the input `Option`.

The general meaning of `replicateM` is described well by the implementation `sequence(List.fill(n)(ma))`. It repeats the `ma` monadic value `n` times and gathers the results in a single value, where the monad `F` determines how values are actually combined.

### Exercise 11.06

For `Par`, `filterM` filters a list, applying the functions in
parallel; for `Option`, it filters a list, but allows
the filtering function to fail and abort the filter
computation; for `Gen`, it produces a generator for 
subsets of the input list, where the function `f` picks a 
'weight' for each element (in the form of a
`Gen[Boolean]`)

``` scala
def filterM[A](ms: List[A])(f: A => F[Boolean]): F[List[A]] =
  ms match {
    case Nil => unit(Nil)
    case h :: t => flatMap(f(h))(b =>
      if (!b) filterM(t)(f)
      else map(filterM(t)(f))(h :: _))
  }

```

### Exercise 11.07

``` scala
def compose[A,B,C](f: A => F[B], g: B => F[C]): A => F[C] =
  a => flatMap(f(a))(g)
```

### Exercise 11.08

``` scala
def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B] =
  compose((_:Unit) => ma, f)(())
```

### Exercise 11.09

Let's rewrite the following in terms of `flatMap`:

``` scala
compose(compose(f, g), h) == compose(f, compose(g, h))

a => flatMap(compose(f, g)(a))(h) == a => flatMap(f(a))(compose(g, h))

a => flatMap((b => flatMap(f(b))(g))(a))(h) ==
  a => flatMap(f(a))(b => flatMap(g(b))(h))
```

So far we have just expanded the definition of `compose`. Equals substituted for equals.
Let's simplify the left side a little:

``` scala
a => flatMap(flatMap(f(a))(g))(h) == a => flatMap(f(a))(b => flatMap(g(b))(h))
```

Let's simplify again by eliminating the `a` argument and substituting a hypothetical value `x` for `f(a)`:

``` scala
flatMap(flatMap(x)(g))(h) == flatMap(x)(b => flatMap(g(b))(h))
```

This now looks exactly like the monad law stated in terms of `flatMap`, just with different names:

``` scala
flatMap(flatMap(x)(f))(g) == flatMap(x)(a => flatMap(f(a))(g))
```

Q.E.D.

### Exercise 11.10

We simply substitute the definition of `compose` in terms of `flatMap`.

``` scala
compose(f, unit)(v) == f(v)           // for all functions f and values v
(a => flatMap(f(a))(unit))(v) == f(v) // Expand `compose`
flatMap(f(v))(unit) == f(v)           // Simplify function application
flatMap(x)(unit) == x                 // Abstract out `f(v)`

compose(unit, f)(x) == f(x)
flatMap(unit(x))(f) == f(x) // Expand `compose`
```

Q.E.D.

### Exercise 11.11

For `Option`, we again consider both cases `None` and `Some` and expand the equation.
The monadic `unit` is the `Some(_)` constructor.

``` scala
// Left identity is trivially true for None:
flatMap(None)(Some(_)) == None

// And here it is for Some:
flatMap(Some(v))(Some(_)) == Some(v)
// Substitute the definition of `flatMap`:
Some(v) == Some(v)

// Right identity is just as easy for None:
flatMap(Some(None))(f) == f(None)
// Substitute definition of flatMap:
f(None) == f(None)

// And for Some:
flatMap(Some(Some(v)))(f) == f(Some(v))
// Substitute definition of flatMap:
f(Some(v)) == f(Some(v))
```

### Exercise 11.12

``` scala
def join[A](mma: F[F[A]]): F[A] = flatMap(mma)(ma => ma)
```

### Exercise 11.13

``` scala
def flatMap[A,B](ma: F[A])(f: A => F[B]) =
  join(map(ma)(f))

def compose[A,B,C](f: A => F[B], g: B => F[C]): A => F[C] =
  a => join(map(f(a))(g))

```

### Exercise 11.14

We can look at the associative law in terms of `flatMap` from another perspective. It says that `x.flatMap(f).flatMap(g)` is equal to `x.flatMap(a => f(a).flatMap(g))` _for all_ choices of `f` and `g`. So let's pick a particular `f` and `g` that's easy to think about. We can just pick the identity function:

``` scala
x.flatMap(z => z).flatMap(z => z) == x.flatMap(a => a.flatMap(z => z))
```

And of course, flatMapping with the identify function is `join`! The associative law can now be stated as:

``` scala
join(join(x)) == x.flatMap(join)
```

And we know that `flatMap` is "map, then join," so let's eliminate this last call to `flatMap`:

``` scala
join(join(x)) == join(map(x)(join))
```

The identity laws in terms of `join` are:

``` scala
join(map(x)(unit)) == x
join(unit(x)) == x
```

This follows from the definition of `join` and the identity laws in terms of `flatMap`:

``` scala
flatMap(x)(unit) == x
flatMap(unit(x))(f) == f(x)
```

For the second law, we simply substitute the identity function for `f`, which gives us `join`.

Let's make a fast-and-loose proof for this version of the associative law using the `List` monad as an example. Of course, `join` in the `List` monad is just _list concatenation_:

```
scala> listMonad.join(List(List(1, 2), List(3, 4)))
res0: List[Int] = List(1, 2, 3, 4)
```

Now let's say we have some lists, nested to a depth of three:

``` scala
val ns: List[List[List[Int]]] =
  List(List(List(1,2), List(3,4)),
       List(List(), List(5)))
```

If we `join` this list, the outer lists get concatenated and we have a list of lists two levels deep:

```
scala> ns.flatten
res1: List[List[Int]] = List(List(1, 2), List(3, 4), List(), List(5))
```

If we instead _map_ `join` over it, we get a different nesting but again two levels deep. This flattens the _inner_ lists.

```
scala> ns.map(listMonad.join)
res2: List[List[Int]] = List(List(1, 2, 3, 4), List(5))
```

And then joining `res1` should be the same as joining `res2`:

```
scala> listMonad.join(res1) == listMonad.join(res2)
res3: Boolean = true
```

So all that the associative law is saying for the `List` monad is that concatenating the outer lists and then the inner ones (`join(join(ns))`) is the same as first concatenating the inner lists and then the outer ones (`join(ns.map(join))`).

### Exercise 11.15

We can state the associative law in terms of `join`:

``` scala
join(join(x)) == join(map(x)(join))
```

For `Par`, the `join` combinator means something like "make the outer thread wait for the inner one to finish." What this law is saying is that if you have threads starting threads three levels deep, then joining the inner threads and then the outer ones is the same as joining the outer threads and then the inner ones.

For `Parser`, the `join` combinator is running the outer parser to produce a `Parser`, then running the inner `Parser` _on the remaining input_. The associative law is saying, roughly, that only the _order_ of nesting matters, since that's what affects the order in which the parsers are run.

### Exercise 11.16

Recall the identity laws:

* left identity:  `flatMap(unit(x))(f) == f(x)`
* right identity: `flatMap(x)(unit)    == x`

The left identity law for `Gen`:
The law states that if you take the values generated by `unit(x)` (which are always `x`) and apply `f` to those values, that's exactly the same as the generator returned by `f(x)`.

The right identity law for `Gen`:
The law states that if you apply `unit` to the values inside the generator `x`, that does not in any way differ from `x` itself.

The left identity law for `List`:
The law says that wrapping a list in a singleton `List` and then flattening the result is the same as doing nothing.

The right identity law for `List`:
The law says that if you take every value in a list, wrap each one in a singleton `List`, and then flatten the result, you get the list you started with.

### Exercise 11.17

``` scala
case class Id[A](value: A) {
  def map[B](f: A => B): Id[B] = Id(f(value))
  def flatMap[B](f: A => Id[B]): Id[B] = f(value)
}

object Id {
  val idMonad = new Monad[Id] {
    def unit[A](a: => A) = Id(a)
    def flatMap[A,B](ida: Id[A])(f: A => Id[B]): Id[B] = ida flatMap f
  }
}
```

### Exercise 11.18

`replicateM` for `State` repeats the same state transition a number of times and returns a list of the results. It's not passing the same starting state many times, but chaining the calls together so that the output state of one is the input state of the next.

`map2` works similarly in that it takes two state transitions and feeds the output state of one to the input of the other. The outputs are not put in a list, but combined with a function `f`.

`sequence` takes an entire list of state transitions and does the same kind of thing as `replicateM`: it feeds the output state of the first state transition to the input state of the next, and so on. The results are accumulated in a list.

### Exercise 11.19

``` scala
// Getting and setting the same state does nothing:
getState.flatMap(setState) == unit(())

// written as for-comprehension:
for {
  x <- getState
  _ <- setState(x)
} yield ()

// Setting the state to `s` and getting it back out yields `s`.
setState(s).flatMap(_ => getState) == unit(s)

// alternatively:
for {
  _ <- setState(s)
  x <- getState
} yield x
```

### Exercise 11.20

``` scala
object Reader {
  def readerMonad[R] = new Monad[({type f[x] = Reader[R,x]})#f] {
    def unit[A](a: => A): Reader[R,A] = Reader(_ => a)
    def flatMap[A,B](st: Reader[R,A])(f: A => Reader[R,B]): Reader[R,B] =
      Reader(r => f(st.run(r)).run(r))
  }

  // A primitive operation for it would be simply to ask for the `R` argument:
  def ask[R]: Reader[R, R] = Reader(r => r)
}
```

The action of Reader's `flatMap` is to pass the `r` argument along to both the
outer Reader and also to the result of `f`, the inner Reader. Similar to how
`State` passes along a state, except that in `Reader` the "state" is read-only.

The meaning of `sequence` here is that if you have a list of functions, you can
turn it into a function that takes one argument and passes it to all the functions
in the list, returning a list of the results.

The meaning of `join` is simply to pass the same value as both arguments to a
binary function.

The meaning of `replicateM` is to apply the same function a number of times to
the same argument, returning a list of the results. Note that if this function
is _pure_, (which it should be), this can be exploited by only applying the
function once and replicating the result instead of calling the function many times.
This means the Reader monad can override replicateM to provide a very efficient
implementation.

## Answers to exercises for chapter 12

### Exercise 12.01

``` scala
def sequence[A](fas: List[F[A]]): F[List[A]] =
  traverse(fas)(fa => fa)

def replicateM[A](n: Int, fa: F[A]): F[List[A]] =
  sequence(List.fill(n)(fa))

def product[A,B](fa: F[A], fb: F[B]): F[(A,B)] =
  map2(fa, fb)((_,_))
```

### Exercise 12.02

``` scala
trait Applicative[F[_]] extends Functor[F] {
  // `map2` is implemented by first currying `f` so we get a function
  // of type `A => B => C`. This is a function that takes `A` and returns
  // another function of type `B => C`. So if we map `f.curried` over an
  // `F[A]`, we get `F[B => C]`. Passing that to `apply` along with the
  // `F[B]` will give us the desired `F[C]`.
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] = 
    apply(map(fa)(f.curried), fb)

  // We simply use `map2` to lift a function into `F` so we can apply it
  // to both `fab` and `fa`. The function being lifted here is `_(_)`,
  // which is the same as the lambda notation `(f, x) => f(x)`. That is,
  // It's a function that takes two arguments:
  //   1. A function `f`
  //   2. An argument `x` to that function
  // and it simply applies `f` to `x`.
  def apply[A,B](fab: F[A => B])(fa: F[A]): F[B] =
    map2(fab, fa)(_(_))
  def unit[A](a: => A): F[A]
  
  def map[A,B](fa: F[A])(f: A => B): F[B] =
    apply(unit(f))(fa)
}

```

### Exercise 12.03

``` scala
/* 
The pattern is simple. We just curry the function 
we want to lift, pass the result to `unit`, and then `apply` 
as many times as there are arguments. 
Each call to `apply` is a partial application of the function
*/
def map3[A,B,C,D](fa: F[A],
                  fb: F[B],
                  fc: F[C])(f: (A, B, C) => D): F[D] =
  apply(apply(apply(unit(f.curried))(fa))(fb))(fc)

def map4[A,B,C,D,E](fa: F[A],
                    fb: F[B],
                    fc: F[C],
                    fd: F[D])(f: (A, B, C, D) => E): F[E]
  apply(apply(apply(apply(unit(f.curried))(fa))(fb))(fc))(fd)
```

### Exercise 12.04

This transposes the list! That is, we start with a list of rows, each of which is possibly infinite in length. We get back a single row, where each element is the column of values at that position. Try it yourself in the REPL.

### Exercise 12.05

``` scala
def eitherMonad[E]: Monad[({type f[x] = Either[E, x]})#f] =
  new Monad[({type f[x] = Either[E, x]})#f] {
    def unit[A](a: => A): Either[E, A] = Right(a)
    def flatMap[A,B](eea: Either[E, A])(f: A => Either[E, B]) = eea match {
      case Right(a) => f(a)
      case Left(e) => Left(e)
    }
  }
```

### Exercise 12.06

``` scala
def validationApplicative[E]: Applicative[({type f[x] = Validation[E,x]})#f] =
  new Applicative[({type f[x] = Validation[E,x]})#f] {
    def unit[A](a: => A) = Success(a)
    override def map2[A,B,C](fa: Validation[E,A],
                             fb: Validation[E,B])(f: (A, B) => C) =
      (fa, fb) match {
        case (Success(a), Success(b)) => Success(f(a, b))
        case (Failure(h1, t1), Failure(h2, t2)) =>
          Failure(h1, t1 ++ Vector(h2) ++ t2)
        case (e@Failure(_, _), _) => e
        case (_, e@Failure(_, _)) => e
      }
  }
```

### Exercise 12.07

We'll just work through left and right identity, but the basic idea for all these proofs is to substitute the definition of all functions, then use the monad laws to make simplifications to the applicative identities. 

Let's start with left and right identity:

``` scala
map2(unit(()), fa)((_,a) => a) == fa // Left identity
map2(fa, unit(()))((a,_) => a) == fa // Right identity
```

We'll do left identity first. We expand definition of `map2`:

``` scala
def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C]
  flatMap(fa)(a => map(fb)(b => f(a,b)))

flatMap(unit())(u => map(fa)(a => a)) == fa
```

We just substituted `unit(())` and `(_,a) => a` in for `f`. `map(fa)(a => a)` is just `fa` by the functor laws, giving us:

``` scala
flatMap(unit())(u => fa) == fa
```

Recall that `flatMap` can be rewritten using `compose`, by using `Unit` as the argument to the first function.

``` scala
compose(unit, u => fa)(()) == fa
```

And by the monad laws:

``` scala
compose(unit, f) == f
```

Therefore, `compose(unit, u => fa)` simplifies to `u => fa`. And `u` is just `Unit` here, and is ignored, so this is equivalent to `fa`:

``` scala
(u => fa)(()) == fa
fa == fa
```

Right identity is symmetric; we just end up using the other identity for `compose`, that `compose(f, unit) == f`.

``` scala
flatMap(fa)(a => map(unit(()))(u => a)) == fa
flatMap(fa)(a => unit(a)) == fa  // via functor laws
compose(u => fa, unit)(()) == fa  
(u => fa)(()) == fa
fa == fa
```

Associativity and naturality are left as an exercise.

### Exercise 12.08

``` scala
def product[G[_]](G: Applicative[G]): Applicative[({type f[x] = (F[x], G[x])})#f] = {
  val self = this
  new Applicative[({type f[x] = (F[x], G[x])})#f] {
    def unit[A](a: => A) = (self.unit(a), G.unit(a))
    override def apply[A,B](fs: (F[A => B], G[A => B]))(p: (F[A], G[A])) =
      (self.apply(fs._1)(p._1), G.apply(fs._2)(p._2))
  }
}
```

### Exercise 12.09

``` scala
def compose[G[_]](G: Applicative[G]): Applicative[({type f[x] = F[G[x]]})#f] = {
  val self = this
  new Applicative[({type f[x] = F[G[x]]})#f] {
    def unit[A](a: => A) = self.unit(G.unit(a))
    override def map2[A,B,C](fga: F[G[A]], fgb: F[G[B]])(f: (A,B) => C) =
      self.map2(fga, fgb)(G.map2(_,_)(f))
  }
}
```

### Exercise 12.10

If `self` and `G` both satisfy the laws, then so does the composite.
The full proof of the laws can be found at:
https://github.com/runarorama/sannanir/blob/master/Applicative.v

### Exercise 12.11

You want to try writing `flatMap` in terms of `Monad[F]` and `Monad[G]`.

``` scala
def flatMap[A,B](mna: F[G[A]])(f: A => F[G[B]]): F[G[B]] =
  self.flatMap(na => G.flatMap(na)(a => ???))
```

Here all you have is `f`, which returns an `F[G[B]]`. For it to have the appropriate type to return from the argument to `G.flatMap`, you'd need to be able to "swap" the `F` and `G` types. In other words, you'd need a *distributive law*. Such an operation is not part of the `Monad` interface.

### Exercise 12.12

``` scala
def sequenceMap[K,V](ofa: Map[K,F[V]]): F[Map[K,V]] =
  (ofa foldLeft unit(Map.empty[K,V])) {
    case (acc, (k, fv)) =>
      map2(acc, fv)((m, v) => m + (k -> v))
  }
```

### Exercise 12.13

``` scala
val listTraverse = new Traverse[List] {
  override def traverse[G[_],A,B](as: List[A])(
    f: A => G[B])(implicit G: Applicative[G]): G[List[B]] =
      as.foldRight(G.unit(List[B]()))((a, fbs) => G.map2(f(a), fbs)(_ :: _))
}

val optionTraverse = new Traverse[Option] {
  override def traverse[G[_],A,B](oa: Option[A])(
    f: A => G[B])(implicit G: Applicative[G]): G[Option[B]] =
      oa match {
        case Some(a) => G.map(f(a))(Some(_))
        case None    => G.unit(None)
      }
}

val treeTraverse = new Traverse[Tree] {
  override def traverse[G[_],A,B](ta: Tree[A])(
    f: A => G[B])(implicit G: Applicative[G]): G[Tree[B]] =
      G.map2(f(ta.head),
             listTraverse.traverse(ta.tail)(a => traverse(a)(f)))(Tree(_, _))
}
```

### Exercise 12.14

The simplest possible `Applicative` we can use is `Id`:

    type Id[A] = A

We already know this forms a `Monad`, so it's also an applicative functor:

    val idMonad = new Monad[Id] {
      def unit[A](a: => A) = a
      override def flatMap[A,B](a: A)(f: A => B): B = f(a)
    }

We can now implement `map` by calling `traverse`, picking `Id` as the `Applicative`:

    def map[A,B](fa: F[A])(f: A => B): F[B] =
      traverse[Id, A, B](xs)(f)(idMonad)

This implementation is suggestive of laws for `traverse`, since we expect this implementation to obey the usual functor laws. See the chapter notes for discussion of the laws for `Traverse`.

Note that we can define `traverse` in terms of `sequence` and `map`, which means that a valid `Traverse` instance may define `sequence` and `map`, or just `traverse`: 

    trait Traverse[F[_]] extends Functor[F] {
      def traverse[G[_]:Applicative,A,B](fa: F[A])(f: A => G[B]): G[F[B]] =
        sequence(map(fa)(f))
      def sequence[G[_]:Applicative,A](fga: F[G[A]]): G[F[A]] =
        traverse(fga)(ga => ga)
      def map[A,B](fa: F[A])(f: A => B): F[B] =
        traverse[Id, A, B](fa)(f)(idMonad)
    }

### Exercise 12.15

It's because `foldRight`, `foldLeft`, and `foldMap` don't give us any way of constructing a value of the foldable type. In order to `map` over a structure, you need the ability to create a new structure (such as `Nil` and `Cons` in the case of a `List`). `Traverse` is able to extend `Functor` precisely because a traversal preserves the original structure. 
An example of a Foldable that is not a functor:

``` scala
case class Iteration[A](a: A, f: A => A, n: Int) {
  def foldMap[B](g: A => B)(M: Monoid[B]): B = {
    def iterate(n: Int, b: B, c: A): B =
      if (n <= 0) b else iterate(n-1, g(c), f(a))
    iterate(n, M.zero, a)
  }
}
```

This class conceptually represents a sequence of `A` values, generated by repeated function application starting from some seed value. But can you see why it's not possible to define `map` for this type?

### Exercise 12.16

``` scala
def reverse[A](fa: F[A]): F[A] =
  mapAccum(fa, toList(fa).reverse)((_, as) => (as.head, as.tail))._1

```

### Exercise 12.17

``` scala
override def foldLeft[A,B](fa: F[A])(z: B)(f: (B, A) => B): B =
  mapAccum(fa, z)((a, b) => ((), f(b, a)))._2
```

### Exercise 12.18

``` scala
def fuse[G[_],H[_],A,B](fa: F[A])(f: A => G[B], g: A => H[B])
                       (implicit G: Applicative[G],
                                 H: Applicative[H]): (G[F[B]], H[F[B]]) =
  traverse[({type f[x] = (G[x], H[x])})#f, A, B](fa)(a => (f(a), g(a)))(G product H)
```

### Exercise 12.19

``` scala
def compose[G[_]](implicit G: Traverse[G]): Traverse[({type f[x] = F[G[x]]})#f] =
  new Traverse[({type f[x] = F[G[x]]})#f] {
    override def traverse[M[_]:Applicative,A,B](fa: F[G[A]])(f: A => M[B]) =
      self.traverse(fa)((ga: G[A]) => G.traverse(ga)(f))
  }
```

### Exercise 12.20

``` scala
def composeM[G[_],H[_]](implicit G: Monad[G], H: Monad[H], T: Traverse[H]):
  Monad[({type f[x] = G[H[x]]})#f] = new Monad[({type f[x] = G[H[x]]})#f] {
    def unit[A](a: => A): G[H[A]] = G.unit(H.unit(a))
    override def flatMap[A,B](mna: G[H[A]])(f: A => G[H[B]]): G[H[B]] =
      G.flatMap(mna)(na => G.map(T.traverse(na)(f))(H.join))
  }
```

## Answers to exercises for chapter 13

### Exercise 13.01

``` scala
def freeMonad[F[_]]: Monad[({type f[a] = Free[F,a]})#f] =
  new Monad[({type f[a] = Free[F,a]})#f] {
    def unit[A](a: => A) = Return(a)
    def flatMap[A,B](fa: Free[F, A])(f: A => Free[F, B]) = fa flatMap f
  }
```

### Exercise 13.02

``` scala
@annotation.tailrec
def runTrampoline[A](a: Free[Function0,A]): A = (a) match {
  case Return(a) => a
  case Suspend(r) => r()
  case FlatMap(x,f) => x match {
    case Return(a) => runTrampoline { f(a) }
    case Suspend(r) => runTrampoline { f(r()) }
    case FlatMap(a0,g) => runTrampoline { a0 flatMap { a0 => g(a0) flatMap f } }
  }
}
```

### Exercise 13.03

``` scala
// Exercise 3: Implement a `Free` interpreter which works for any `Monad`
def run[F[_],A](a: Free[F,A])(implicit F: Monad[F]): F[A] = step(a) match {
  case Return(a) => F.unit(a)
  case Suspend(r) => r
  case FlatMap(Suspend(r), f) => F.flatMap(r)(a => run(f(a)))
  case _ => sys.error("Impossible, since `step` eliminates these cases")
}
```

### Exercise 13.04

``` scala
def translate[F[_],G[_],A](f: Free[F,A])(fg: F ~> G): Free[G,A] = {
  type FreeG[A] = Free[G,A]
  val t = new (F ~> FreeG) {
    def apply[A](a: F[A]): Free[G,A] = Suspend { fg(a) }
  }
  runFree(f)(t)(freeMonad[G])
}

def runConsole[A](a: Free[Console,A]): A =
  runTrampoline { translate(a)(new (Console ~> Function0) {
    def apply[A](c: Console[A]) = c.toThunk
  })}
```

### Exercise 13.05

``` scala
/*
 * Exercise 5: Implement a non-blocking read from an asynchronous file channel.
 * We'll just give the basic idea - here, we construct a `Future`
 * by reading from an `AsynchronousFileChannel`, a `java.nio` class
 * which supports asynchronous reads.
 */

import java.nio._
import java.nio.channels._

def read(file: AsynchronousFileChannel,
         fromPosition: Long,
         numBytes: Int): Par[Either[Throwable, Array[Byte]]] =
  Par.async { (cb: Either[Throwable, Array[Byte]] => Unit) =>
    val buf = ByteBuffer.allocate(numBytes)
    file.read(buf, fromPosition, (), new CompletionHandler[Integer, Unit] {
      def completed(bytesRead: Integer, ignore: Unit) = {
        val arr = new Array[Byte](bytesRead)
        buf.slice.get(arr, 0, bytesRead)
        cb(Right(arr))
      }
      def failed(err: Throwable, ignore: Unit) =
        cb(Left(err))
    })
  }

// note: We can wrap `read` in `Free[Par,A]` using the `Suspend` constructor
```

## Answers to exercises for chapter 14

### Exercise 14.01

``` scala
def fill(xs: Map[Int,A]): ST[S,Unit] =
  xs.foldRight(ST[S,Unit](())) {
    case ((k, v), st) => st flatMap (_ => write(k, v))
  }
```

### Exercise 14.02

``` scala
// An action that does nothing
def noop[S] = ST[S,Unit](())

def partition[S](a: STArray[S,Int], l: Int, r: Int, pivot: Int): ST[S,Int] =
  for {
    vp <- a.read(pivot)
    _ <- a.swap(pivot, r)
    j <- STRef(l)
    _ <- (l until r).foldLeft(noop[S])((s, i) => for {
      _ <- s
      vi <- a.read(i)
      _  <- if (vi < vp) (for {
        vj <- j.read
        _  <- a.swap(i, vj)
        _  <- j.write(vj + 1)
      } yield ()) else noop[S]
    } yield ())
    x <- j.read
    _ <- a.swap(x, r)
  } yield x

def qs[S](a: STArray[S,Int], l: Int, r: Int): ST[S, Unit] = if (l < r) for {
  pi <- partition(a, l, r, l + (r - l) / 2)
  _ <- qs(a, l, pi - 1)
  _ <- qs(a, pi + 1, r)
} yield () else noop[S]
```

### Exercise 14.03

``` scala
import scala.collection.mutable.HashMap

sealed trait STMap[S,K,V] {
  protected def table: HashMap[K,V]

  def size: ST[S,Int] = ST(table.size)

  // Get the value under a key
  def apply(k: K): ST[S,V] = ST(table(k))

  // Get the value under a key, or None if the key does not exist
  def get(k: K): ST[S, Option[V]] = ST(table.get(k))

  // Add a value under a key
  def +=(kv: (K, V)): ST[S,Unit] = ST(table += kv)

  // Remove a key
  def -=(k: K): ST[S,Unit] = ST(table -= k)
}

object STMap {
  def empty[S,K,V]: ST[S, STMap[S,K,V]] = ST(new STMap[S,K,V] {
    val table = HashMap.empty[K,V]
  })

  def fromMap[S,K,V](m: Map[K,V]): ST[S, STMap[S,K,V]] = ST(new STMap[S,K,V] {
    val table = (HashMap.newBuilder[K,V] ++= m).result
  })
}
```

## Answers to exercises for chapter 15

### Exercise 15.01

``` scala
def take[I](n: Int): Process[I,I] =
  if (n <= 0) Halt()
  else await(i => emit(i, take[I](n-1)))

def drop[I](n: Int): Process[I,I] =
  if (n <= 0) id
  else await(i => drop[I](n-1))

def takeWhile[I](f: I => Boolean): Process[I,I] =
  await(i =>
    if (f(i)) emit(i, takeWhile(f))
    else      Halt())

def dropWhile[I](f: I => Boolean): Process[I,I] =
  await(i =>
    if (f(i)) dropWhile(f)
    else      emit(i,id))

```

### Exercise 15.02

``` scala
/*
 * Exercise 2: Implement `count`.
 *
 * Here's one implementation, with three stages - we map all inputs
 * to 1.0, compute a running sum, then finally convert the output
 * back to `Int`. The three stages will be interleaved - as soon
 * as the first element is examined, it will be converted to 1.0,
 * then added to the running total, and then this running total
 * will be converted back to `Int`, then the `Process` will examine
 * the next element, and so on.
 */
def count[I]: Process[I,Int] =
  lift((i: I) => 1.0) |> sum |> lift(_.toInt)

/* For comparison, here is an explicit recursive implementation. */
def count2[I]: Process[I,Int] = {
  def go(n: Int): Process[I,Int] =
    await((i: I) => emit(n+1, go(n+1)))
  go(0)
}


```

### Exercise 15.03

``` scala
/*
 * Exercise 3: Implement `mean`.
 *
 * This is an explicit recursive definition. We'll factor out a
 * generic combinator shortly.
 */
def mean: Process[Double,Double] = {
  def go(sum: Double, count: Double): Process[Double,Double] =
    await((d: Double) => emit((sum+d) / (count+1), go(sum+d,count+1)))
  go(0.0, 0.0)
}

```

### Exercise 15.04

``` scala
def sum2: Process[Double,Double] =
  loop(0.0)((d:Double, acc) => (acc+d,acc+d))

def count3[I]: Process[I,Int] =
  loop(0)((_:I,n) => (n+1,n+1))


```

### Exercise 15.05

``` scala
/*
 * Exercise 5: Implement `|>`. Let the types guide your implementation.
 */
def |>[O2](p2: Process[O,O2]): Process[I,O2] = {
  p2 match {
    case Halt() => Halt()
    case Emit(h,t) => Emit(h, this |> t)
    case Await(f) => this match {
      case Emit(h,t) => t |> f(Some(h))
      case Halt() => Halt() |> f(None)
      case Await(g) => Await((i: Option[I]) => g(i) |> p2)
    }
  }
}

```

### Exercise 15.06

``` scala
// this uses the `zip` function defined in exercise 7
def zipWithIndex: Process[I,(O,Int)] =
  this zip (count map (_ - 1))

```

### Exercise 15.07

``` scala
/*
 * Exercise 7: Can you think of a generic combinator that would
 * allow for the definition of `mean` in terms of `sum` and
 * `count`?
 *
 * Yes, it is `zip`, which feeds the same input to two processes.
 * The implementation is a bit tricky, as we have to make sure
 * that input gets fed to both `p1` and `p2`.
 */
def zip[A,B,C](p1: Process[A,B], p2: Process[A,C]): Process[A,(B,C)] =
  (p1, p2) match {
    case (Halt(), _) => Halt()
    case (_, Halt()) => Halt()
    case (Emit(b, t1), Emit(c, t2)) => Emit((b,c), zip(t1, t2))
    case (Await(recv1), _) =>
      Await((oa: Option[A]) => zip(recv1(oa), feed(oa)(p2)))
    case (_, Await(recv2)) =>
      Await((oa: Option[A]) => zip(feed(oa)(p1), recv2(oa)))
  }
  
def feed[A,B](oa: Option[A])(p: Process[A,B]): Process[A,B] =
  p match {
    case Halt() => p
    case Emit(h,t) => Emit(h, feed(oa)(t))
    case Await(recv) => recv(oa)
  }


```

### Exercise 15.08

``` scala
/*
 * Exercise 8: Implement `exists`
 *
 * We choose to emit all intermediate values, and not halt.
 * See `existsResult` below for a trimmed version.
 */
def exists[I](f: I => Boolean): Process[I,Boolean] =
  lift(f) |> any

/* Emits whether a `true` input has ever been received. */
def any: Process[Boolean,Boolean] =
  loop(false)((b:Boolean,s) => (s || b, s || b))

/* A trimmed `exists`, containing just the final result. */
def existsResult[I](f: I => Boolean) =
  exists(f) |> takeThrough(!_) |> dropWhile(!_) |> echo.orElse(emit(false))

/*
 * Like `takeWhile`, but includes the first element that tests
 * false.
 */
def takeThrough[I](f: I => Boolean): Process[I,I] =
  takeWhile(f) ++ echo

/* Awaits then emits a single value, then halts. */
def echo[I]: Process[I,I] = await(i => emit(i))

```

### Exercise 15.09

This process defines the here is core logic, a transducer that converts input lines
(assumed to be temperatures in degrees fahrenheit) to output lines (temperatures in
degress celsius). Left as an exercise to supply another wrapper like `processFile`
to actually do the IO and drive the process.

``` scala
def convertFahrenheit: Process[String,String] =
  filter((line: String) => !line.startsWith("#")) |>
  filter(line => line.trim.nonEmpty) |>
  lift(line => toCelsius(line.toDouble).toString)

def toCelsius(fahrenheit: Double): Double =
  (5.0 / 9.0) * (fahrenheit - 32.0)

```

### Exercise 15.10

``` scala
/*
 * Exercise 10: This function is defined only if given a `MonadCatch[F]`.
 * Unlike the simple `runLog` interpreter defined in the companion object
 * below, this is not tail recursive and responsibility for stack safety
 * is placed on the `Monad` instance.
 */
def runLog(implicit F: MonadCatch[F]): F[IndexedSeq[O]] = {
  def go(cur: Process[F,O], acc: IndexedSeq[O]): F[IndexedSeq[O]] =
    cur match {
      case Emit(h,t) => go(t, acc :+ h)
      case Halt(End) => F.unit(acc)
      case Halt(err) => F.fail(err)
      case Await(req,recv) => F.flatMap (F.attempt(req)) {
        e => go(Try(recv(e)), acc)
      }
    }
  go(this, IndexedSeq())
}

```

### Exercise 15.11

``` scala
/*
 * Create a `Process[IO,O]` from the lines of a file, using
 * the `resource` combinator above to ensure the file is closed
 * when processing the stream of lines is finished.
 */
def lines(filename: String): Process[IO,String] =
  resource
    { IO(io.Source.fromFile(filename)) }
    { src =>
        lazy val iter = src.getLines // a stateful iterator
        def step = if (iter.hasNext) Some(iter.next) else None
        lazy val lines: Process[IO,String] = eval(IO(step)).flatMap {
          case None => Halt(End)
          case Some(line) => Emit(line, lines)
        }
        lines
    }
    { src => eval_ { IO(src.close) } }

/* Exercise 11: Implement `eval`, `eval_`, and use these to implement `lines`. */
def eval[F[_],A](a: F[A]): Process[F,A] =
  await[F,A,A](a) {
    case Left(err) => Halt(err)
    case Right(a) => Emit(a, Halt(End))
  }

/* Evaluate the action purely for its effects. */
def eval_[F[_],A,B](a: F[A]): Process[F,B] =
  eval[F,A](a).drain[B]
```

### Exercise 15.12

Notice this is the standard monadic `join` combinator!

``` scala
def join[F[_],A](p: Process[F,Process[F,A]]): Process[F,A] =
  p.flatMap(pa => pa)
```

