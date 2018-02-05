# Goroutines and Channels
1. Go enables two styles of concurrent programming. Goroutines and channels support communicating sequential processes or CSP, a model of concurrency in which values are passed between independent activities (goroutines) but variables are for the most part confined to a single activity. Go also covers some aspects of the more traditional model of shared memory multithreading.

## Goroutines
1. Other than by returning from main or exiting the program, there is no programmatic way for one goroutine to stop another, but there are ways to communicate with a goroutine to request that it stop itself.

## Channels
1. The type of a channel whose elements have type `int` is written `chan int`. To create a channel, we use the built-in make function: `ch := make(chan int)`
2. a channel is a reference to the data structure created by make. When we copy a channel or pass one as an argument to a function, we are copying a reference, so caller and callee refer to the same data structure. As with other reference types, the zero value of a channel is nil.
3. Two channels of the same type may be compared using ==. The comparison is true if both are references to the same channel data structure. A channel may also be compared to nil.
4. A channel has two principal operations, send and receive, collectively known as communications. 

        ```
        ch <- x  // a send statement
        x = <-ch // a receive expression in an assignment statement
        <-ch     // a receive statement; result is discarded    
        ```

5. we call the built-in `close` function to close a channel, indicating that no more values will ever be sent on this channel; subsequent attempts to send will panic. Receive operations on a closed channel yield the values that have been sent until no more values are left; any receive operations thereafter complete immediately and yield the zero value of the channel’s element type.
6. A channel created with a simple call to make is called an unbuffered channel, but make accepts an optional second argument, an integer called the channel’s capacity. If the capacity is non-zero, make creates a buffered channel.
7. Communication over an unbuffered channel causes the sending and receiving goroutines to synchronize. Because of this, unbuffered channels are sometimes called synchronous channels. When a value is sent on an unbuffered channel, the receipt of the value happens before the reawakening of the sending goroutine.
8. There is no way to test directly whether a channel has been closed, but there is a variant of the receive operation that produces two results: the received channel element, plus a boolean value, conventionally called ok, which is true for a successful receive and false for a receive on a closed and drained channel.
9. the language lets us use a range loop to iterate over channels too. This is a more convenient syntax for receiving all the values sent on a channel and terminating the loop after the last one.
10. You needn’t close every channel when you’ve finished with it. It’s only necessary to close a channel when it is important to tell the receiving goroutines that all data have been sent. A channel that the garbage collector determines to be unreachable will have its resources reclaimed whether or not it is closed.
11. Attempting to close an already-closed channel causes a panic, as does closing a nil channel.
12. the Go type system provides unidirectional channel types that expose only one or the other of the send and receive operations. The type `chan<- int`, a send-only channel of int, allows sends but not receives. Conversely, the type `<-chan int`, a receive-only channel of int, allows receives but not sends. Violations of this discipline are detected at compile time.
13. it is a compile-time error to attempt to close a receive-only channel.
14. Conversions from bidirectional to unidirectional channel types are permitted in any assignment. There is no going back, however: once you have a value of a unidirectional type such as chan<- int, there is no way to obtain from it a value of type chan int that refers to the same channel data structure.
15. A send operation on a buffered channel inserts an element at the back of the queue, and a receive operation removes an element from the front. If the channel is full, the send operation blocks its goroutine until space is made available by another goroutine’s receive. Conversely, if the channel is empty, a receive operation blocks until a value is sent by another goroutine.
16. a program can obtain the channel’s buffer capacity by calling the built-in `cap` function. When applied to a channel, the built-in len function returns the number of elements cur- rently buffered. 
17. An application of a buffered channel: it makes parallel requests to three mirrors, that is, equivalent but geographically distributed servers. It sends their responses over a buffered channel, then receives and returns only the first response, which is the quickest one to arrive. Had we used an unbuffered channel, the two slower goroutines would have gotten stuck trying to send their responses on a channel from which no goroutine will ever receive. This sit- uation, called a goroutine leak, would be a bug. 

## Looping in Parallel
1. Problems that consist entirely of subproblems that are completely independent of each other are described as embarrassingly parallel. Embarrassingly parallel problems are the easiest kind to implement concurrently and enjoy performance that scales linearly with the amount of parallelism.
2. Be aware of the problem of loop variable capture in concurrent calls!
3. Make sure you understand this example. The structure of this code is a common and idiomatic pattern for looping in parallel when we don’t know the number of iterations.

        ```
        // makeThumbnails6 makes thumbnails for each file received from the channel.
        // It returns the number of bytes occupied by the files it creates.
        func makeThumbnails6(filenames <-chan string) int64 {
            sizes := make(chan int64)
            var wg sync.WaitGroup // number of working goroutines
            for f := range filenames {
                wg.Add(1)
                // worker
                go func(f string) {
                    defer wg.Done()
                    thumb, err := thumbnail.ImageFile(f)
                    if err != nil {
                        log.Println(err)
                        return
                    }
                    info, _ := os.Stat(thumb) // OK to ignore error
                    sizes <- info.Size()
                }(f)
            }
            // closer
            go func() {
                wg.Wait()
                close(sizes)
            }()
            var total int64
            for size := range sizes {
                total += size
            }
            return total
        }
        ```

4. Unbounded parallelism is rarely a good idea since there is always a limiting factor in the system. We can limit parallelism using a buffered channel of capacity n to model a concurrency primitive called a counting semaphore. Alternatively, we can spin off n goroutines to wait for a channel and let the main goroutine to send values to the channel.

## Multiplexing with select
1. The general form of a `select` statement is similar to a `switch` statement. It has a number of cases and an optional default. Each case specifies a communication (a send or receive operation on some channel) and an associated block of statements. A select waits until a communication for some case is ready to proceed. It then performs that communication and executes the case’s associated statements; the other communications do not happen.
2. A select with no cases, select{}, waits forever.
3. If multiple cases are ready, select picks one at random, which ensures that every channel has an equal chance of being selected.
4. A select may have a default, which specifies what to do when none of the other communications can proceed immediately. This makes the select non-blocking. Doing it repeatedly is called polling a channel.
5. The zero value for a channel is nil. Because send and receive operations on a nil channel block forever, a case in a select statement whose channel is nil is never selected. This lets us use nil to enable or disable cases that correspond to features like handling timeouts or cancellation, responding to other input events, or emitting output. 

## Cancellation
1. After a channel has been closed and drained of all sent values, subsequent receive operations proceed immediately, yielding zero values. We can exploit this to create a broadcast mechanism: don’t send a value on the channel, close it.