# Functions

## Function Declarations
1. If the function returns one unnamed result or no results at all, parentheses are optional and usually omitted.
2. Like parameters, results may be named. In that case, each name declares a local variable ini- tialized to the zero value for its type.
3. A sequence of parameters or results of the same type can be factored so that the type itself is written only once.
4. The type of a function is sometimes called its signature. Two functions have the same type or signature if they have the same sequence of parameter types and the same sequence of result types.
5. Go has no concept of default parameter values, nor any way to specify arguments by name, so the names of parameters and results don’t matter to the caller except as documentation.
6. Parameters are local variables within the body of the function, with their initial values set to the arguments supplied by the caller. Function parameters and named results are variables in the same lexical block as the function’s outermost local variables.
7. Arguments are passed by value, so the function receives a copy of each argument; modifica- tions to the copy do not affect the caller. 
8. You may occasionally encounter a function declaration without a body, indicating that the function is implemented in a language other than Go. Such a declaration defines the function signature.

## Recursion
1. typical Go implementations use variable-size stacks that start small and grow as needed up to a limit on the order of a gigabyte. This lets us use recursion safely and without worrying about overflow

## Multiple Return Values
1. Go’s garbage collector recycles unused memory, but do not assume it will release unused operating system resources like open files and network connections. They should be closed explicitly.
2. In a function with named results, the operands of a return statement may be omitted. This is called a bare return. A bare return is a shorthand way to return each of the named result variables in order.
3. Bare returns can reduce code duplication, but they rarely make code easier to understand. For this reason, bare returns are best used sparingly.

## Errors
1. A function for which failure is an expected behavior returns an additional result, convention- ally the last one. If the failure has only one possible cause, the result is a `boolean`, usually called ok. More often, and especially for I/O, the failure may have a variety of causes for which the caller will need an explanation. In such cases, the type of the additional result is `error`.
2. The built-in type `error` is an interface type. An error may be nil or non-nil, that nil implies success and non-nil implies failure, and that a non-nil error has an error message string which we can obtain by calling its `Error` method.
3. Exceptions tend to entangle the description of an error with the control flow required to handle it, often leading to an undesirable outcome: routine errors are reported to the end user in the form of an incomprehensible stack trace, full of information about the structure of the program but lacking intelligible context about what went wrong. By contrast, Go programs use ordinary control-flow mechanisms like if and return to respond to errors. This style undeniably demands that more attention be paid to error-han- dling logic, but that is precisely the point.
4. We should build descriptive errors by successively prefixing additional context information to the original error message. Because error messages are frequently chained together, message strings should not be capitalized and newlines should be avoided.
5. In general, the call f(x) is responsible for reporting the attempted operation f and the argu- ment value x as they relate to the context of the error. The caller is responsible for adding fur- ther information that it has but the call f(x) does not.
6. For errors that represent transient or unpredictable problems, it may make sense to retry the failed operation, possibly with a delay between tries, and perhaps with a limit on the number of attempts or the time spent trying before giving up entirely.
7. Functions tend to exhibit a common structure, with a series of initial checks to reject errors, followed by the substance of the function at the end, minimally indented.

## Function Values
1. Functions are first-class values in Go: like other values, function values have types, and they may be assigned to variables or passed to or returned from functions.
2. The zero value of a function type is nil.
3. Function values may be compared with nil, but they are not comparable.

## Anonymouse Functions
1. A function literal is written like a function declaration, but without a name following the func keyword. It is an expression, and its value is called an anonymous function.
2. Functions defined in this way have access to the entire lexical environment, so the inner function can refer to variables from the enclosing function. This indicates function values are not just code but can have state. This is why we classify functions as reference types and why function values are not comparable. Function values like these are implemented using a technique called closures.
3. When an anonymous function requires recursion, we must first declare a variable, and then assign the anonymous function to that variable.
4. Topological sorting is to generate a sequence from a acyclic directed graph so that a preceding item does not depends on the items after it in the sequence. We can achieve this by a depth-first search.
5. Watch out for the problem of iteration variable capture. This is most often encoutered when using the `go` statement or with `defer`.
        
        ```
        var rmdirs []func()
        for _, dir := range tempDirs() {
            os.MkdirAll(dir, 0755)
            rmdirs = append(rmdirs, func() {
                os.RemoveAll(dir) // NOTE: incorrect! This will always be the last item!
            }) 
        }
        ```

## Variadic Functions
1. To declare a variadic function, the type of the final parameter is preceded by an ellipsis, `...`, which indicates that the function may be called with any number of arguments of this type: `func sum(vals ...int) int`. The `vals` in this example is a `[]int` slice.
2. To invoke a variadic function when the arguments are already in a slice, place an ellipsis after the final argument. `sum(values...)`
3. The suffix f is a widely followed naming convention for variadic functions that accept a Printf-style format string, e.g `errorf`
4. A parameter with interface{} type means that this parameter can accept any type of value.

## Deferred Function Calls
1. Syntactically, a `defer` statement is an ordinary function or method call prefixed by the keyword defer. The function and argument expressions are evaluated when the statement is executed, but the actual call is deferred until the function that contains the defer statement has finished.
2. Any number of calls may be deferred; they are executed in the reverse of the order in which they were deferred.
3. A defer statement is often used with paired operations like open and close, connect and disconnect, or lock and unlock to ensure that resources are released in all cases, no matter how complex the control flow. The right place for a defer statement that releases a resource is immediately after the resource has been successfully acquired, after the error handing on opening.
4. The defer statement can also be used to pair ‘‘on entry’’ and ‘‘on exit’’ actions when debugging a complex function. 

        ```
        func bigSlowOperation() {
            defer trace("bigSlowOperation")() // don't forget the extra parentheses
            // ...lots of work...
            time.Sleep(10 * time.Second) // simulate slow operation by sleeping
        }
        func trace(msg string) func() {
            start := time.Now()
            log.Printf("enter %s", msg)
            return func() { log.Printf("exit %s (%s)", msg, time.Since(start)) }
        }
        ```

5. Deferred functions run after return statements have updated the function’s result variables. Because an anonymous function can access its enclosing function’s variables, including named results, a deferred anonymous function can observe the function’s results, or even change the values that the enclosing function returns to its caller:

        ```
        func triple(x int) (result int) {
            defer func() { result += x }()
            return double(x)
        }
        fmt.Println(triple(4)) // "12"
        ```

6. Because deferred functions aren’t executed until the very end of a function’s execution, a defer statement in a loop deserves extra scrutiny.

        ```
        for _, filename := range filenames {
            f, err := os.Open(filename)
            if err != nil {
                return err
            }
            defer f.Close() // NOTE: risky; could run out of file descriptors
            // ...process f...
        }
        ```

7. Only use defer if the return results from the deferred function does not matter. For example, using `defer` to close a writable file would be subtly wrong because on many file systems, notably NFS, write errors are not reported immediately but may be postponed until the file is closed. Failure to check the result of the close operation could cause serious data loss to go unnoticed.

## Panic
1. During a typical panic, normal execution stops, all deferred function calls in that goroutine are executed, and the program crashes with a log message.
2. The built-in `panic` function may be called directly; it accepts any value as an argument. A panic is often the best thing to do when some "impossible" situation happens, for instance, execution reaches a case that logically can’t happen.
3. panic generally used for grave errors, such as a logical inconsistency in the program. On the other side, "expected" errors, the kind that arise from incorrect input, misconfiguration, or failing I/O, should be handled gracefully; they are best dealt with using error values.
4. It’s good practice to assert that the preconditions of a function hold, but this can easily be done to excess. Unless you can provide a more informative error message or detect an error sooner, there is no point asserting a condition that the runtime will check for you.
5. The `Must` prefix is a common naming convention for wraper functions that causes a panic if an error occurs in the inner function. Examples are the MustCompile function for regexp, when the caller knows the pattern must to be well-formed. Or the simular use case in `template.Must`.


## Recover
1. If the built-in recover function is called within a deferred function and the function contain- ing the defer statement is panicking, recover ends the current state of panic and returns the panic value. The function that was panicking does not continue where it left off but returns normally. If recover is called at any other time, it has no effect and returns nil.
2. Recovering from a panic within the same package can help simplify the handling of complex or unexpected errors, but as a general rule, you should not attempt to recover from another package’s panic. When recovering, make sure to recover only from panics that were intended to be recovered from. This intention can be encoded by using a distinct, unexported type for the panic value and testing whether the value returned by recover has that type. If so, we report the panic as an ordinary error; if not, we call panic with the same value to resume the state of panic.