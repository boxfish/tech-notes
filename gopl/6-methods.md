# Methods
1. In Go, an object is simply a value or variable that has methods, and a method is a function associated with a particular type. An object-oriented program is one that uses methods to express the properties and operations of each data structure so that clients need not access the object’s representation directly.

## Method Declaration
1. A method is declared with a variant of the ordinary function declaration in which an extra parameter, called receiver, appears before the function name. The parameter attaches the function to the type of that parameter.
2. In Go, we don’t use a special name like this or self for the receiver; we choose receiver names just as we would for any other parameter. A common choice for the receiver name is the first letter of the type name.
3. Since methods and fields inhabit the same name space, declaring a method X on the struct type that has a field with same name `X` would be ambiguous and the compiler will reject it.
4. Go allows methods to be associated with any type, even basic types such as numbers, strings, etc. However, method cannot be defined on unnamed type directly, e.g. the following is invalid: `func (s []int) sum() int`

## Methods with a Pointer Receiver
1. Because calling a function makes a copy of each argument value, if a function needs to update a variable, or if an argument is so large that we wish to avoid copying it, we must pass the address of the variable using a pointer. The same goes for methods that need to update the receiver variable.
2. convention dictates that if any method of a type has a pointer receiver, then all methods of the type should have a pointer receiver, even ones that don’t strictly need it.
3. Named types and pointers to them are the only types that may appear in a receiver declaration. Furthermore, to avoid ambiguities, method declarations are not permitted on named types that are themselves pointer types. 
4. If the receiver `p` is a variable of type `Point` but the method requires a `*Point` receiver, we can use this shorthand: `p.ScaleBy(2)` and the compiler will perform an implicit `&p` on the variable. This works only on addressable variables. For example there is no way to obtain the address of a struct literal: `Point{1, 2}.ScaleBy(2)`, or inside a map value `m["key"].ScaleBy(2)`.
5. On the other side, we can call a Point method with a *Point receiver, and the compiler inserts an implicit * operation for us.
6. Argument v.s. Parameter: argument is the real value when the function is called. parameter is for function declaration.
7. Nil is a valid receiver value.

## Composing Types by Struct Embedding
1. The type of an anonymous field may be a pointer to a named type, in which case fields and methods are promoted indirectly from the pointed-to object.
2. When the compiler resolves a selector to a method, it first looks for a directly declared method, then for methods promoted once from embedded fields, then for methods promoted twice from embedded fields, and so on. The compiler reports an error if the selector was ambiguous because two methods were promoted from the same rank.
3. It’s possible and sometimes useful for unnamed struct types to have methods too by embedding other type that have methods. 

## Method Values and Expressions
1. A method value is a function that bind a method to a specific receiver value. This function can then be invoked without a receiver value; it needs only the non-receiver arguments. Method values are useful when a package’s API calls for a function value, and the client’s desired behavior for that function is to call a method on a specific receiver.
2. A method expression, written `T.f` or `(*T).f` where `T` is a type, yields a function value with a regular first parameter taking the place of the receiver, so it can be called in the usual way. Method expressions can be helpful when you need a value to represent a choice among several methods belonging to the same type so that you can call the chosen method with many different receivers.

## Encapsulation
1. to encapsulate an object, we must make it a struct.
2. the unit of encapsulation is the package, not the type as in many other languages. The fields of a struct type are visible to all code within the same package.