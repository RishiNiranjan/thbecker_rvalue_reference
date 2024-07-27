## Move Semantics
Suppose X is a class that holds a pointer or handle to some resource, say, m_pResource. 
By a resource, I mean anything that takes considerable effort to construct, clone, or 
destruct. A good example is std::vector, which holds a collection of objects that live
in an array of allocated memory. Then, logically, the copy assignment operator for X looks like this:
```
X& X::operator=(X const & rhs)
{
  // [...]
  // Make a clone of what rhs.m_pResource refers to.
  // Destruct the resource that m_pResource refers to. 
  // Attach the clone to m_pResource.
  // [...]
}
```
Similar reasoning applies to the copy constructor. Now suppose X is used as follows:
```
X foo();
X x;
// perhaps use x in various ways
x = foo();
```

The last line above
clones the resource from the temporary returned by foo,
destructs the resource held by x and replaces it with the clone,
destructs the temporary and thereby releases its resource.
Rather obviously, it would be ok, and much more efficient, to swap resource 
pointers (handles) between x and the temporary, and then let the temporary's
destructor destruct x's original resource. In other words, in the special case
where the right hand side of the assignment is an rvalue, we want the copy assignment operator to act like this:
```
// [...]
// swap m_pResource and rhs.m_pResource
// [...]  
```
This is called move semantics. With C++11, this conditional behavior can be achieved via an overload:
```
X& X::operator=(<mystery type> rhs)
{
  // [...]
  // swap this->m_pResource and rhs.m_pResource
  // [...]  
}
```
Since we're defining an overload of the copy assignment operator, our "mystery type" must essentially 
be a reference: we certainly want the right hand side to be passed to us by reference. Moreover, we 
expect the following behavior of the mystery type: when there is a choice between two overloads where 
one is an ordinary reference and the other is the mystery type, then rvalues must prefer the mystery type, 
while lvalues must prefer the ordinary reference.

If you now substitute "rvalue reference" for "mystery type" in the above, you're essentially looking at the definition of rvalue references.
