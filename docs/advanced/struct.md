# struct
`struct`s can have a positive impact on performance due to their nature of living on the stack instead of the heap.
Of course `struct` will be put onto the heap if they outlive their stack frame.

## Passing by value can be faster than by reference
If the `struct` is small enough (at best as wide as a machine word, so 8 bytes on 64bit applications or 4 bytes on 32 bit applications) passing a `struct` by reference can be significant faster than any reference.
The reason is that copying a `struct` on the stack is a cheap operation when the `struct` is small enough. A reference in contrast is always as wide as the machine word but on top, for reading values, it has to be dereferences.  

> 💡 Info: In detail explanation can be found [here](https://steven-giesel.com/blogPost/d205e99c-e784-49be-a2e6-7f9c44ab890f).