# Interfaces
1. Interface types express generalizations or abstractions about the behaviors of other types. It lets us write functions that are more flexible and adaptable because they are not tied to the details of one particular implementation.
2. what makes Go’s interfaces so distinctive is that they are satisfied implicitly. In other words, there’s no need to declare all the interfaces that a given concrete type satisfies; simply possessing the necessary methods is enough.

## Interfaces as Contract
1. This freedom to substitute one type for another that satisfies the same interface is called substitutability, and is a hallmark of object-oriented programming.

## Interface Types
1. We can declare new interface types as combinations of existing ones.

## Interface Satisfaction
1. a value of type T does not possess all the methods that a *T pointer does, and as a result it might satisfy fewer interfaces.
2. Only the methods revealed by the interface type may be called, even if the concrete type has others:

## Interface Values
1. A value of an interface type, or interface value, has two components, a concrete type and a value of that type. These are called the interface’s dynamic type and dynamic value.
2. In general, we cannot know at compile time what the dynamic type of an interface value will be, so a call through an interface must use dynamic dispatch. Instead of a direct call, the compiler must generate code to obtain the address of the method named Write from the type descriptor, then make an indirect call to that address. The receiver argument for the call is a copy of the interface’s dynamic value.
3. Interface values may be compared using == and !=. Two interface values are equal if both are nil, or if their dynamic types are identical and their dynamic values are equal according to the usual behavior of == for that type. Because interface values are comparable, they may be used as the keys of a map or as the operand of a switch statement.
4. However, if two interface values are compared and have the same dynamic type, but that type is not comparable (a slice, for instance), then the comparison fails with a panic.
5. A nil interface value, which contains no value at all, is not the same as an interface value containing a pointer that happens to be nil.

## Sorting with sort.Interface
1. An in-place sort algorithm needs three things—the length of the sequence, a means of com- paring two elements, and a way to swap two elements—so they are the three methods of sort.Interface: `Len() int`, `Less(i, j int) bool`, `Swap(i, j int)`.

## The http.Handler Interface
1. If we need to wrap a function into an interfce with sole method that has the same funciton signature, we can define it as a function type and the behavior of the method is just to call the undelrying function.

## The error interface
1. In `errors` package, the `New` funciton returns a pointer to the underlying error struct so that each error created won't compare equal to others.

## Type Assertions
1. If the asserted type `T` is a concrete type, then the type assertion checks whether x’s dynamic type is identical to T. If this check succeeds, the result of the type assertion is x’s dynamic value, whose type is of course T. In other words, a type assertion to a concrete type extracts the concrete value from its operand. If the check fails, then the operation panics.
2. If the asserted type `T` is an interface type, then the type assertion checks whether x’s dynamic type satisfies T. If this check succeeds, the dynamic value is not extracted; the result is still an interface value with the same type and value components, but the result has the interface type T.
3. No matter what type was asserted, if the operand is a nil interface value, the type assertion fails. 
4. If the type assertion appears in an assignment in which two results are expected, the operation does not panic on failure but instead returns an additional second result, a boolean indicating success.
5. Interface Type Assertions can be used to query behaviors, that is to check whether certain method exists by defining a new interface with that method and test whether the given interface also satisfies this new interface.

## Type Switches
1. Interfaces are used in two distinct styles. In the first style, an interface’s meth- ods express the similarities of the concrete types that satisfy the interface but hide the rep- resentation details and intrinsic operations of those concrete types. The emphasis is on the methods, not on the concrete types. The second style exploits the ability of an interface value to hold values of a variety of concrete types and considers the interface to be the union of those types. Type assertions are used to discriminate among these types dynamically and treat each case differently. In this style, the emphasis is on the concrete types that satisfy the interface, not on the interface’s methods (if indeed it has any), and there is no hiding of information. If you’re familiar with object-oriented programming, you may recognize these two styles as subtype polymorphism and ad hoc polymorphism.
2. a type switch looks like an ordinary switch statement in which the operand is `x.(type)` — that’s literally the keyword `type` — and each case has one or more types. A type switch enables a multi-way branch based on the interface value’s dynamic type. The nil case matches if x == nil, and the default case matches if no other case does. If the case needs access to the value extracted by the type assertion, the type switch statement has an extended form that binds the extracted value to a new variable within each case: `switch x := x.(type) { ... }`
