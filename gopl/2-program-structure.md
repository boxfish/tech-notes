# 2 Program Structure

## 2.1 Names
1. Go programmers use "camel case" when forming names by combining words
2. The letters of acronyms and initialisms like ASCII and HTML are always rendered in the same case. e.g. escapeHTML, not escapeHtml

## 2.2 Declarations
1. If an entity is declared within a function, it is local to that function. 
2. If declared outside of a function, however, it is visible in all files of the package to which it belongs. 
3. If the name begins with an upper-case letter, it is exported, which means that it is visible and accessible outside of its own package and may be referred to by other parts of the program.

## 2.3 Variables
1. `var name type = expression`: 
    * if the type is ommited, it is determined by the initializer expression.
    * if the expression is omitted, the initial value is the zero value for the type, which is 0 for numbers, false for booleans, "" for strings, and nil for interfaces and reference types (slice, pointer, map, channel, function). The zero value of an aggregate type like an array or a struct has the zero value of all of its elements or fields.
2. A short variable declaration acts like an assignment only to variables that were already declared in the same lexical block.
3. The expression `new(T)` creates an unnamed variable of type `T`, initializes it to the zero value of `T`, and returns its address, which is a value of type `*T`.
4. The lifetime of a package-level variable is the entire execution of the program. By contrast, local variables have dynamic lifetimes: a new instance is created each time the declaration statement is executed, and the variable lives on until it becomes unreachable.
5. The variable becomes unreachable if there is no path from any package-level variable or local variable of each currently active function to the varialbe in question, following pointers and other kinds of references.

## 2.4 Assignments
1. In tupple assignment, all of the right-hand side expressions are evaluated before any of the variables are updated.

      ```
         // swap
         x, y = y, x
         // GCD
         func gcd(x, y int) int {
            for y != 0 {
               x, y = y, x%y
            }
            return x
         }
         // Fibonacci
         func fib(n int) int {
            x, y := 0, 1
            for i := 0; i < n; i++ {
               x, y = y, x + y
            }
            return x
         }
      ```
2. There are three operators that sometimes return multipel results. We can assign unwarted values to the blank identifier.

      ```
         _, ok = m[key] // map lookup
         _, ok = x.(T)  // type assertion
         _, ok = <-ch   // channel receive
      ```

## 2.5 Type Declaration
1. One use case for type declaration is to define Celsius and Fahrenheit as two types, so that we don't accidentally combine values in two units together without explicit conversion.
2. For every type `T`, there is a corresponding conversion operation `T(x)` that converts the value `x` to type `T`. 
3. A conversion from one type to another is allowed if both have the same underlying type, or if both are unnamed pointer types that point to variables of the same underlying type; these conversions change the type but not the representation of the value. 
4. Conversions are also allowed between numeric types, and between string and some slice types. These conversions may change the representation of the value.
5. Converting a string to a `[]byte` slice allocates a copy of the string data.

## 2.6 Pacakges and Files
1. Any file may contain any number of `init` functions that are automatically executed when the program starts, in the order in which they are declared.
2. Understand the population count example!

## 2.7 Scopes
1. A syntactic block is a sequence of statements enclosed in braces like those that surround the body of a function or loop. A name declared inside a syntactic block is not visible outside that block.
2. There is a lexical block for the entire source code, called the universe block; for each package; for each file; for each for, if, and switch statement; for each case in a switch or select statement; and, of course, for each explicit syntactic block.
3. The declarations of built-in types, functions, and constants like `int`, `len`, and `true` are in the universe block and can be referred to throughout the entire program.
4. Declarations outside any function, that is, at package level, can be referred to from any file in the same package.
5. Imported packages are declared at the file level, so they can be referred to from the same file.
6. Normal practice in Go is to deal with the error in the if block and then return, so that the successful execution path is not indented.
