## Rvalue References
If X is any type, then X&& is called an rvalue reference to X. For better distinction, the ordinary 
reference X& is now also called an lvalue reference.
An rvalue reference is a type that behaves much like the ordinary reference X&, with several exceptions. 
The most important one is that when it comes to function overload resolution, lvalues prefer old-style 
lvalue references, whereas rvalues prefer the new rvalue references:
```
void foo(X& x); // lvalue reference overload
void foo(X&& x); // rvalue reference overload

X x;
X foobar();

foo(x); // argument is lvalue: calls foo(X&)
foo(foobar()); // argument is rvalue: calls foo(X&&)
```
So the gist of it is:
 
Rvalue references allow a function to branch at compile time (via overload resolution) on the condition 
"Am I being called on an lvalue or an rvalue?"
 
It is true that you can overload any function in this manner, as shown above. But in the overwhelming
majority of cases, this kind of overload should occur only for copy constructors and assignment 
operators, for the purpose of achieving move semantics:
```
X& X::operator=(X const & rhs); // classical implementation
X& X::operator=(X&& rhs)
{
  // Move semantics: exchange content between this and rhs
  return *this;
}
```
Implementing an rvalue reference overload for the copy constructor is similar.
 
Caveat: As it happens so often in C++, what looks just right at first glance is still a little shy of perfect. 
It turns out that in some cases, the simple exchange of content between this and rhs in the implementation of 
the copy assignment operator above is not quite good enough. We'll come back to this in Section 4, "Forcing Move Semantics" below.

Note: If you implement
```
void foo(X&);
but not
void foo(X&&);
```
then of course the behavior is unchanged: foo can be called on l-values, but not on r-values.If you implement
```
void foo(X const &);
but not
void foo(X&&);
```
then again, the behavior is unchanged: foo can be called on l-values and r-values, but it is not
possible to make it distinguish between l-values and r-values. That is possible only by implementing
void foo(X&&); as well. 
Finally, if you implement
```
void foo(X&&);
but neither one of
void foo(X&);
and
void foo(X const &);
```
then, according to the final version of C++11, foo can be called on r-values, but trying to call it on
an l-value will trigger a compile error.
