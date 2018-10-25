- [Concurrent computing](#sec-1)
- [Synchronous concurrency](#sec-2)
- [Dining philosophers](#sec-3)
- [Handle a signal](#sec-4)
- [Atomic updates](#sec-5)
- [Determine if only one instance is running](#sec-6)
- [Events](#sec-7)
- [Checkpoint synchronization](#sec-8)
- [Active object](#sec-9)
- [Metered concurrency](#sec-10)
- [Rendezvous](#sec-11)

# Concurrent computing     :concurrency:basic_language_learning:<a id="sec-1"></a>

Task

Using either native language concurrency syntax or freely available libraries, write a program to display the strings "Enjoy" "Rosetta" "Code", one string per line, in random order.

Concurrency syntax must use threads, tasks, co-routines, or whatever concurrency is called in your language.

Channel[edit]

Simple and direct solution: Start three goroutines, give each one a word. Each sleeps, then returns the word on a channel. The main goroutine prints words as they return. The print loop represents a checkpoint &#x2013; main doesn't exit until all words have been returned and printed.

This solution also shows a good practice for generating random numbers in concurrent goroutines. While certainly not needed for this RC task, in the more general case where you have a number of goroutines concurrently needing random numbers, the goroutines can suffer congestion if they compete heavily for the sole default library source. This can be relieved by having each goroutine create its own non-sharable source. Also particularly in cases where there might be a large number of concurrent goroutines, the source provided in subrepository rand package (exp/rand) can be a better choice than the standard library generator. The subrepo generator requires much less memory for "state" and is much faster to seed.

```go
package main
 
import (
    "fmt"
    "golang.org/x/exp/rand"
    "time"
)
 
func main() {
    words := []string{"Enjoy", "Rosetta", "Code"}
    seed := uint64(time.Now().UnixNano())
    q := make(chan string)
    for i, w := range words {
        go func(w string, seed uint64) {
            r := rand.New(rand.NewSource(seed))
            time.Sleep(time.Duration(r.Int63n(1e9)))
            q <- w
        }(w, seed+uint64(i))
    }
    for i := 0; i < len(words); i++ {
        fmt.Println(<-q)
    }
}
```

Afterfunc[edit]

time.Afterfunc combines the sleep and the goroutine start. log.Println serializes output in the case goroutines attempt to print concurrently. sync.WaitGroup is used directly as a checkpoint.

```go
package main
 
import (
    "log"
    "math/rand"
    "os"
    "sync"
    "time"
)
 
func main() {
    words := []string{"Enjoy", "Rosetta", "Code"}
    rand.Seed(time.Now().UnixNano())
    l := log.New(os.Stdout, "", 0)
    var q sync.WaitGroup
    q.Add(len(words))
    for _, w := range words {
        w := w
        time.AfterFunc(time.Duration(rand.Int63n(1e9)), func() {
            l.Println(w)
            q.Done()
        })
    }
    q.Wait()
}
```

Select[edit]

This solution might stretch the intent of the task a bit. It is concurrent but not parallel. Also it doesn't sleep and doesn't call the random number generator explicity. It works because the select statement is specified to make a "pseudo-random fair choice" among multiple channel operations.

```go
package main
 
import "fmt"
 
func main() {
    w1 := make(chan bool, 1)
    w2 := make(chan bool, 1)
    w3 := make(chan bool, 1)
    for i := 0; i < 3; i++ {
        w1 <- true
        w2 <- true
        w3 <- true
        fmt.Println()
        for i := 0; i < 3; i++ {
            select {
            case <-w1:
                fmt.Println("Enjoy")
            case <-w2:
                fmt.Println("Rosetta")
            case <-w3:
                fmt.Println("Code")
            }
        }
    }
}
```

Output:

```go
Code
Rosetta
Enjoy

Enjoy
Rosetta
Code

Rosetta
Enjoy
Code

```

# Synchronous concurrency     :concurrency:clarify_task:<a id="sec-2"></a>

The goal of this task is to create two concurrent activities ("Threads" or "Tasks", not processes.) that share data synchronously. Your language may provide syntax or libraries to perform concurrency. Different languages provide different implementations of concurrency, often with different names. Some languages use the term threads, others use the term tasks, while others use co-processes. This task should not be implemented using fork, spawn, or the Linux/UNIX/Win32 pipe command, as communication should be between threads, not processes.

One of the concurrent units will read from a file named "input.txt" and send the contents of that file, one line at a time, to the other concurrent unit, which will print the line it receives to standard output. The printing unit must count the number of lines it prints. After the concurrent unit reading the file sends its last line to the printing unit, the reading unit will request the number of lines printed by the printing unit. The reading unit will then print the number of lines printed by the printing unit.

This task requires two-way communication between the concurrent units. All concurrent units must cleanly terminate at the end of the program.

```go
package main
 
import (
    "bufio"
    "fmt"
    "log"
    "os"
)
 
func main() {
    lines := make(chan string)
    count := make(chan int)
    go func() {
        c := 0
        for l := range lines {
            fmt.Println(l)
            c++
        }
        count <- c
    }()
 
    f, err := os.Open("input.txt")
    if err != nil {
        log.Fatal(err)
    }
    for s := bufio.NewScanner(f); s.Scan(); {
        lines <- s.Text()
    }
    f.Close()
    close(lines)
    fmt.Println("Number of lines:", <-count)
}
```

# Dining philosophers     :classic_cs_problems_and_programs:concurrency:puzzles:<a id="sec-3"></a>

The dining philosophers problem illustrates non-composability of low-level synchronization primitives like semaphores. It is a modification of a problem posed by Edsger Dijkstra.

Five philosophers, Aristotle, Kant, Spinoza, Marx, and Russell (the tasks) spend their time thinking and eating spaghetti. They eat at a round table with five individual seats. For eating each philosopher needs two forks (the resources). There are five forks on the table, one left and one right of each seat. When a philosopher cannot grab both forks it sits and waits. Eating takes random time, then the philosopher puts the forks down and leaves the dining room. After spending some random time thinking about the nature of the universe, he again becomes hungry, and the circle repeats itself.

It can be observed that a straightforward solution, when forks are implemented by semaphores, is exposed to deadlock. There exist two deadlock states when all five philosophers are sitting at the table holding one fork each. One deadlock state is when each philosopher has grabbed the fork left of him, and another is when each has the fork on his right.

There are many solutions of the problem, program at least one, and explain how the deadlock is prevented.

Channels[edit]

Goroutine synchronization done with Go channels. Deadlock prevented by making one philosopher "left handed."

```go
package main
 
import (
    "hash/fnv"
    "log"
    "math/rand"
    "os"
    "time"
)
 
// Number of philosophers is simply the length of this list.
// It is not otherwise fixed in the program.
var ph = []string{"Aristotle", "Kant", "Spinoza", "Marx", "Russell"}
 
const hunger = 3                // number of times each philosopher eats
const think = time.Second / 100 // mean think time
const eat = time.Second / 100   // mean eat time
 
var fmt = log.New(os.Stdout, "", 0) // for thread-safe output
 
var done = make(chan bool)
 
// This solution uses channels to implement synchronization.
// Sent over channels are "forks."
type fork byte
 
// A fork object in the program models a physical fork in the simulation.
// A separate channel represents each fork place.  Two philosophers
// have access to each fork.  The channels are buffered with capacity = 1,
// representing a place for a single fork.
 
// Goroutine for philosopher actions.  An instance is run for each
// philosopher.  Instances run concurrently.
func philosopher(phName string,
    dominantHand, otherHand chan fork, done chan bool) {
    fmt.Println(phName, "seated")
    // each philosopher goroutine has a random number generator,
    // seeded with a hash of the philosopher's name.
    h := fnv.New64a()
    h.Write([]byte(phName))
    rg := rand.New(rand.NewSource(int64(h.Sum64())))
    // utility function to sleep for a randomized nominal time
    rSleep := func(t time.Duration) {
        time.Sleep(t/2 + time.Duration(rg.Int63n(int64(t))))
    }
    for h := hunger; h > 0; h-- {
        fmt.Println(phName, "hungry")
        <-dominantHand // pick up forks
        <-otherHand
        fmt.Println(phName, "eating")
        rSleep(eat)
        dominantHand <- 'f' // put down forks
        otherHand <- 'f'
        fmt.Println(phName, "thinking")
        rSleep(think)
    }
    fmt.Println(phName, "satisfied")
    done <- true
    fmt.Println(phName, "left the table")
}
 
func main() {
    fmt.Println("table empty")
    // Create fork channels and start philosopher goroutines,
    // supplying each goroutine with the appropriate channels
    place0 := make(chan fork, 1)
    place0 <- 'f' // byte in channel represents a fork on the table.
    placeLeft := place0
    for i := 1; i < len(ph); i++ {
        placeRight := make(chan fork, 1)
        placeRight <- 'f'
        go philosopher(ph[i], placeLeft, placeRight, done)
        placeLeft = placeRight
    }
    // Make one philosopher left handed by reversing fork place
    // supplied to philosopher's dominant hand.
    // This makes precedence acyclic, preventing deadlock.
    go philosopher(ph[0], place0, placeLeft, done)
    // they are all now busy eating
    for range ph {
        <-done // wait for philosphers to finish
    }
    fmt.Println("table empty")
}
```

Output:

```go
table empty
Kant seated
Marx seated
Spinoza seated
Aristotle seated
Kant hungry
Russell seated
Marx hungry
Russell hungry
Kant eating
Marx eating
Aristotle hungry
Spinoza hungry
Kant thinking
Marx thinking
Spinoza eating
Russell eating
Kant hungry
Russell thinking
Aristotle eating
Marx hungry
Spinoza thinking
Marx eating
Russell hungry
Marx thinking
Aristotle thinking
Russell eating
Kant eating
Russell thinking
Aristotle hungry
Kant thinking
Aristotle eating
Spinoza hungry
Spinoza eating
Marx hungry
Aristotle thinking
Russell hungry
Aristotle hungry
Kant hungry
Spinoza thinking
Kant eating
Marx eating
Marx thinking
Russell eating
Kant thinking
Marx satisfied
Marx left the table
Russell thinking
Aristotle eating
Spinoza hungry
Spinoza eating
Russell satisfied
Russell left the table
Kant satisfied
Kant left the table
Spinoza thinking
Aristotle thinking
Aristotle satisfied
Aristotle left the table
Spinoza satisfied
Spinoza left the table
table empty

```

Mutexes and WaitGroup[edit]

The first solution just uses channels for synchronization. Channels can solve lots of problems but the sync library has a few other functions to more directly model common operations. In Dining Philosophers, fork use is mutually exclusive so it's very clear to model forks with sync.Mutex objects. Also waiting for a number of concurrent tasks to finish is a common pattern directly implemented with sync.WaitGroup.

One more concurrency technique actually used in both solutions is to use the log package for output rather than the fmt package. Output from concurrent goroutines can get accidentally interleaved in some cases. While neither package makes claims about this problem, the log package historically has been coded to avoid interleaved output.

```go
package main
 
import (
    "hash/fnv"
    "log"
    "math/rand"
    "os"
    "sync"
    "time"
)
 
var ph = []string{"Aristotle", "Kant", "Spinoza", "Marx", "Russell"}
 
const hunger = 3
const think = time.Second / 100
const eat = time.Second / 100
 
var fmt = log.New(os.Stdout, "", 0)
 
var dining sync.WaitGroup
 
func philosopher(phName string, dominantHand, otherHand *sync.Mutex) {
    fmt.Println(phName, "seated")
    h := fnv.New64a()
    h.Write([]byte(phName))
    rg := rand.New(rand.NewSource(int64(h.Sum64())))
    rSleep := func(t time.Duration) {
        time.Sleep(t/2 + time.Duration(rg.Int63n(int64(t))))
    }
    for h := hunger; h > 0; h-- {
        fmt.Println(phName, "hungry")
        dominantHand.Lock() // pick up forks
        otherHand.Lock()
        fmt.Println(phName, "eating")
        rSleep(eat)
        dominantHand.Unlock() // put down forks
        otherHand.Unlock()
        fmt.Println(phName, "thinking")
        rSleep(think)
    }
    fmt.Println(phName, "satisfied")
    dining.Done()
    fmt.Println(phName, "left the table")
}
 
func main() {
    fmt.Println("table empty")
    dining.Add(5)
    fork0 := &sync.Mutex{}
    forkLeft := fork0
    for i := 1; i < len(ph); i++ {
        forkRight := &sync.Mutex{}
        go philosopher(ph[i], forkLeft, forkRight)
        forkLeft = forkRight
    }
    go philosopher(ph[0], fork0, forkLeft)
    dining.Wait() // wait for philosphers to finish
    fmt.Println("table empty")
}
```

# Handle a signal     :concurrency:signal_handling:<a id="sec-4"></a>

Most operating systems provide interrupt facilities, sometimes called signals either generated by the user or as a result of program failure or reaching a limit like file space. Unhandled signals generally terminate a program in a disorderly manner. Signal handlers are created so that the program behaves in a well-defined manner upon receipt of a signal.

Task

Provide a program that displays an integer on each line of output at the rate of about one per half second. Upon receipt of the SIGINT signal (often generated by the user typing ctrl-C ( or better yet, SIGQUIT ctrl-\\ )) the program will cease outputting integers, output the number of seconds the program has run, and then the program will quit.

```go
package main
 
import (
    "fmt"
    "os"
    "os/signal"
    "time"
)
 
func main() {
    start := time.Now()
    k := time.Tick(time.Second / 2)
    sc := make(chan os.Signal, 1)
    signal.Notify(sc, os.Interrupt)
    for n := 1; ; {
        // not busy waiting, this blocks until one of the two
        // channel operations is possible
        select {
        case <-k:
            fmt.Println(n)
            n++
        case <-sc:
            fmt.Printf("Ran for %f seconds.\n",
                time.Now().Sub(start).Seconds())
            return
        }
    }
}
```

Output:

```go
1
2
3
^C
Ran for 1.804877 seconds.

```

# Atomic updates     :concurrency:<a id="sec-5"></a>

Task

Define a data type consisting of a fixed number of 'buckets', each containing a nonnegative integer value, which supports operations to:

get the current value of any bucket remove a specified amount from one specified bucket and add it to another, preserving the total of all bucket values, and clamping the transferred amount to ensure the values remain non-negative

In order to exercise this data type, create one set of buckets, and start three concurrent tasks:

As often as possible, pick two buckets and make their values closer to equal. As often as possible, pick two buckets and arbitrarily redistribute their values. At whatever rate is convenient, display (by any means) the total value and, optionally, the individual values of each bucket.

The display task need not be explicit; use of e.g. a debugger or trace tool is acceptable provided it is simple to set up to provide the display.

This task is intended as an exercise in atomic operations.   The sum of the bucket values must be preserved even if the two tasks attempt to perform transfers simultaneously, and a straightforward solution is to ensure that at any time, only one transfer is actually occurring — that the transfer operation is atomic.

```go
package main
 
import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)
 
const nBuckets = 10
 
type bucketList struct {
    b [nBuckets]int // bucket data specified by task
 
    // transfer counts for each updater, not strictly required by task but
    // useful to show that the two updaters get fair chances to run.
    tc [2]int
 
    sync.Mutex // synchronization
}
 
// Updater ids, to track number of transfers by updater.
// these can index bucketlist.tc for example.
const (
    idOrder = iota
    idChaos
)
 
const initialSum = 1000 // sum of all bucket values
 
// Constructor.
func newBucketList() *bucketList {
    var bl bucketList
    // Distribute initialSum across buckets.
    for i, dist := nBuckets, initialSum; i > 0; {
        v := dist / i
        i--
        bl.b[i] = v
        dist -= v
    }
    return &bl
}
 
// method 1 required by task, get current value of a bucket
func (bl *bucketList) bucketValue(b int) int {
    bl.Lock() // lock before accessing data
    r := bl.b[b]
    bl.Unlock()
    return r
}
 
// method 2 required by task
func (bl *bucketList) transfer(b1, b2, a int, ux int) {
    // Get access.
    bl.Lock()
    // Clamping maintains invariant that bucket values remain nonnegative.
    if a > bl.b[b1] {
        a = bl.b[b1]
    }
    // Transfer.
    bl.b[b1] -= a
    bl.b[b2] += a
    bl.tc[ux]++ // increment transfer count
    bl.Unlock()
}
 
// additional useful method
func (bl *bucketList) snapshot(s *[nBuckets]int, tc *[2]int) {
    bl.Lock()
    *s = bl.b
    *tc = bl.tc
    bl.tc = [2]int{} // clear transfer counts
    bl.Unlock()
}
 
var bl = newBucketList()
 
func main() {
    // Three concurrent tasks.
    go order() // make values closer to equal
    go chaos() // arbitrarily redistribute values
    buddha()   // display total value and individual values of each bucket
}
 
// The concurrent tasks exercise the data operations by calling bucketList
// methods.  The bucketList methods are "threadsafe", by which we really mean
// goroutine-safe.  The conconcurrent tasks then do no explicit synchronization
// and are not responsible for maintaining invariants.
 
// Exercise 1 required by task: make values more equal.
func order() {
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    for {
        b1 := r.Intn(nBuckets)
        b2 := r.Intn(nBuckets - 1)
        if b2 >= b1 {
            b2++
        }
        v1 := bl.bucketValue(b1)
        v2 := bl.bucketValue(b2)
        if v1 > v2 {
            bl.transfer(b1, b2, (v1-v2)/2, idOrder)
        } else {
            bl.transfer(b2, b1, (v2-v1)/2, idOrder)
        }
    }
}
 
// Exercise 2 required by task: redistribute values.
func chaos() {
    r := rand.New(rand.NewSource(time.Now().Unix()))
    for {
        b1 := r.Intn(nBuckets)
        b2 := r.Intn(nBuckets - 1)
        if b2 >= b1 {
            b2++
        }
        bl.transfer(b1, b2, r.Intn(bl.bucketValue(b1)+1), idChaos)
    }
}
 
// Exercise 3 requred by task: display total.
func buddha() {
    var s [nBuckets]int
    var tc [2]int
    var total, nTicks int
 
    fmt.Println("sum  ---updates---    mean  buckets")
    tr := time.Tick(time.Second / 10)
    for {
        <-tr
        bl.snapshot(&s, &tc)
        var sum int
        for _, l := range s {
            if l < 0 {
                panic("sob") // invariant not preserved
            }
            sum += l
        }
        // Output number of updates per tick and cummulative mean
        // updates per tick to demonstrate "as often as possible"
        // of task exercises 1 and 2.
        total += tc[0] + tc[1]
        nTicks++
        fmt.Printf("%d %6d %6d %7d  %3d\n", sum, tc[0], tc[1], total/nTicks, s)
        if sum != initialSum {
            panic("weep") // invariant not preserved
        }
    }
}
```

Output:

```go
sum  ---updates---    mean  buckets
1000 317832 137235  455067  [100 100 100 100 100 100 100 100 100 100]
1000 391239 339389  592847  [ 85 266  81  85 131  37  62  80 111  62]
1000 509436 497362  730831  [ 70 194 194  62  16 193  10  16 126 119]
1000 512065 499038  800899  [100 100 100 100 100 100 100 100 100 100]
1000 250590 121947  715226  [ 47 271  78  61  34 199  73  58 100  79]
...

```

# Determine if only one instance is running     :concurrency:programming_environment_operations:<a id="sec-6"></a>

This task is to determine if there is only one instance of an application running. If the program discovers that an instance of it is already running, then it should display a message indicating that it is already running and exit.

Port[edit]

Recommended over file based solutions. It has the advantage that the port is always released when the process ends.

```go
package main
 
import (
    "fmt"
    "net"
    "time"
)
 
const lNet = "tcp"
const lAddr = ":12345"
 
func main() {
    if _, err := net.Listen(lNet, lAddr); err != nil {
        fmt.Println("an instance was already running")
        return
    }
    fmt.Println("single instance started")
    time.Sleep(10 * time.Second)
}
```

File[edit]

Solution using O<sub>CREATE</sub>|O<sub>EXCL</sub>. This solution has the problem that if anything terminates the program early, the lock file remains.

```go
package main
 
import (
    "fmt"
    "os"
    "time"
)
 
// The path to the lock file should be an absolute path starting from the root.
// (If you wish to prevent the same program running in different directories,
// that is.)
const lfn = "/tmp/rclock"
 
func main() {
    lf, err := os.OpenFile(lfn, os.O_RDWR|os.O_CREATE|os.O_EXCL, 0666)
    if err != nil {
        fmt.Println("an instance is already running")
        return
    }
    lf.Close()
    fmt.Println("single instance started")
    time.Sleep(10 * time.Second)
    os.Remove(lfn)
}
```

Here's a fluffier version that stores the PID in the lock file to provide better messages. It has the same problem of the lock file remaining if anything terminates the program early.

```go
package main
 
import (
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
)
 
// The path to the lock file should be an absolute path starting from the root.
// (If you wish to prevent the same program running in different directories, that is.)
const lfn = "/tmp/rclock"
 
func main() {
    lf, err := os.OpenFile(lfn, os.O_RDWR|os.O_CREATE|os.O_EXCL, 0666)
    if err == nil {
        // good
        // 10 digit pid seems to be a standard for lock files
        fmt.Fprintf(lf, "%10d", os.Getpid())
        lf.Close()
        defer os.Remove(lfn)
    } else {
        // problem
        fmt.Println(err)
        // dig deeper
        lf, err = os.Open(lfn)
        if err != nil {
            return
        }
        defer lf.Close()
        fmt.Println("inspecting lock file...")
        b10 := make([]byte, 10)
        _, err = lf.Read(b10)
        if err != nil {
            fmt.Println(err)
            return
        }
        pid, err := strconv.Atoi(strings.TrimSpace(string(b10)))
        if err != nil {
            fmt.Println(err)
            return
        }
        fmt.Println("lock file created by pid", pid)
        return
    }
    fmt.Println(os.Getpid(), "running...")
    time.Sleep(1e10)
}
```

# Events     :concurrency:encyclopedia:<a id="sec-7"></a>

Event is a synchronization object. An event has two states signaled and reset. A task may await for the event to enter the desired state, usually the signaled state. It is released once the state is entered. Releasing waiting tasks is called event notification. Programmatically controlled events can be set by a task into one of its states.

In concurrent programming event also refers to a notification that some state has been reached through an asynchronous activity. The source of the event can be:

internal, from another task, programmatically; external, from the hardware, such as user input, timer, etc. Signaling an event from the hardware is accomplished by means of hardware interrupts.

Event is a low-level synchronization mechanism. It neither identify the state that caused it signaled, nor the source of, nor who is the subject of notification. Events augmented by data and/or publisher-subscriber schemes are often referred as messages, signals etc.

In the context of general programming event-driven architecture refers to a design that deploy events in order to synchronize tasks with the asynchronous activities they must be aware of. The opposite approach is polling sometimes called busy waiting, when the synchronization is achieved by an explicit periodic querying the state of the activity. As the name suggests busy waiting consumes system resources even when the external activity does not change its state.

Event-driven architectures are widely used in GUI design and SCADA systems. They are flexible and have relatively short response times. At the same time event-driven architectures suffer to the problems related to their unpredictability. They face race condition, deadlocking, live locks and priority inversion. For this reason real-time systems tend to polling schemes, trading performance for predictability in the worst case scenario.

Variants of events[edit]

A Go channel can represent an manual-reset event, as described by the task. The two states of signaled and reset correspond to the presence or absence of a value on the channel. The program signals by sending a value on the channel. The event is reset when the waiting task explicitly executes the channel receive operation, <-event.

```go
package main
 
import (
    "log"
    "os"
    "time"
)
 
func main() {
    l := log.New(os.Stdout, "", log.Ltime | log.Lmicroseconds)
    l.Println("program start")
    event := make(chan int)
    go func() {
        l.Println("task start")
        <-event
        l.Println("event reset by task")
    }()
    l.Println("program sleeping")
    time.Sleep(1 * time.Second)
    l.Println("program signaling event")
    event <- 0
    time.Sleep(100 * time.Millisecond)
}
```

Output:

```go
01:27:21.862000 program start
01:27:21.862245 program sleeping
01:27:21.867269 task start
01:27:22.868294 program signaling event
01:27:22.868346 event reset by task

```

# Checkpoint synchronization     :concurrency:classic_cs_problems_and_programs:<a id="sec-8"></a>

The checkpoint synchronization is a problem of synchronizing multiple tasks. Consider a workshop where several workers (tasks) assembly details of some mechanism. When each of them completes his work they put the details together. There is no store, so a worker who finished its part first must wait for others before starting another one. Putting details together is the checkpoint at which tasks synchronize themselves before going their paths apart.

The task

Implement checkpoint synchronization in your language.

Make sure that the solution is race condition-free. Note that a straightforward solution based on events is exposed to race condition. Let two tasks A and B need to be synchronized at a checkpoint. Each signals its event (EA and EB correspondingly), then waits for the AND-combination of the events (EA&EB) and resets its event. Consider the following scenario: A signals EA first and gets blocked waiting for EA&EB. Then B signals EB and loses the processor. Then A is released (both events are signaled) and resets EA. Now if B returns and enters waiting for EA&EB, it gets lost.

When a worker is ready it shall not continue before others finish. A typical implementation bug is when a worker is counted twice within one working cycle causing its premature completion. This happens when the quickest worker serves its cycle two times while the laziest one is lagging behind.

If you can, implement workers joining and leaving.

Solution 1, WaitGroup

The type sync.WaitGroup in the standard library implements a sort of checkpoint synchronization. It allows one goroutine to wait for a number of other goroutines to indicate something, such as completing some work.

This first solution is a simple interpretation of the task, starting a goroutine (worker) for each part, letting the workers run concurrently, and waiting for them to all indicate completion. This is efficient and idiomatic in Go.

```go
package main
 
import (
    "log"
    "math/rand"
    "sync"
    "time"
)
 
func worker(part string) {
    log.Println(part, "worker begins part")
    time.Sleep(time.Duration(rand.Int63n(1e6)))
    log.Println(part, "worker completes part")
    wg.Done()
}
 
var (
    partList    = []string{"A", "B", "C", "D"}
    nAssemblies = 3
    wg          sync.WaitGroup
)
 
func main() {
    rand.Seed(time.Now().UnixNano())
    for c := 1; c <= nAssemblies; c++ {
        log.Println("begin assembly cycle", c)
        wg.Add(len(partList))
        for _, part := range partList {
            go worker(part)
        }
        wg.Wait()
        log.Println("assemble.  cycle", c, "complete")
    }
}
```

Output:

Sample run, with race detector option to show no race conditions detected.

```go
$ go run -race r1.go
2018/06/04 15:44:11 begin assembly cycle 1
2018/06/04 15:44:11 A worker begins part
2018/06/04 15:44:11 B worker begins part
2018/06/04 15:44:11 B worker completes part
2018/06/04 15:44:11 D worker begins part
2018/06/04 15:44:11 A worker completes part
2018/06/04 15:44:11 C worker begins part
2018/06/04 15:44:11 D worker completes part
2018/06/04 15:44:11 C worker completes part
2018/06/04 15:44:11 assemble.  cycle 1 complete
2018/06/04 15:44:11 begin assembly cycle 2
2018/06/04 15:44:11 A worker begins part
2018/06/04 15:44:11 B worker begins part
2018/06/04 15:44:11 A worker completes part
2018/06/04 15:44:11 C worker begins part
2018/06/04 15:44:11 D worker begins part
2018/06/04 15:44:11 C worker completes part
2018/06/04 15:44:11 B worker completes part
2018/06/04 15:44:11 D worker completes part
2018/06/04 15:44:11 assemble.  cycle 2 complete
2018/06/04 15:44:11 begin assembly cycle 3
2018/06/04 15:44:11 A worker begins part
2018/06/04 15:44:11 B worker begins part
2018/06/04 15:44:11 A worker completes part
2018/06/04 15:44:11 C worker begins part
2018/06/04 15:44:11 D worker begins part
2018/06/04 15:44:11 B worker completes part
2018/06/04 15:44:11 C worker completes part
2018/06/04 15:44:11 D worker completes part
2018/06/04 15:44:11 assemble.  cycle 3 complete
$

```

Solution 2, channels

Channels also synchronize, and in addition can send data. The solution shown here is very similar to the WaitGroup solution above but sends data on a channel to simulate a completed part. The channel operations provide synchronization and a WaitGroup is not needed.

```go
package main
 
import (
    "log"
    "math/rand"
    "strings"
    "time"
)
 
func worker(part string, completed chan string) {
    log.Println(part, "worker begins part")
    time.Sleep(time.Duration(rand.Int63n(1e6)))
    p := strings.ToLower(part)
    log.Println(part, "worker completed", p)
    completed <- p
}
 
var (
    partList    = []string{"A", "B", "C", "D"}
    nAssemblies = 3
)
 
func main() {
    rand.Seed(time.Now().UnixNano())
    completed := make([]chan string, len(partList))
    for i := range completed {
        completed[i] = make(chan string)
    }
    for c := 1; c <= nAssemblies; c++ {
        log.Println("begin assembly cycle", c)
        for i, part := range partList {
            go worker(part, completed[i])
        }
        a := ""
        for _, c := range completed {
            a += <-c
        }
        log.Println(a, "assembled.  cycle", c, "complete")
    }
}
```

Output:

```go
$ go run -race r2.go
2018/06/04 15:56:33 begin assembly cycle 1
2018/06/04 15:56:33 A worker begins part
2018/06/04 15:56:33 B worker begins part
2018/06/04 15:56:33 A worker completed a
2018/06/04 15:56:33 D worker begins part
2018/06/04 15:56:33 C worker begins part
2018/06/04 15:56:33 B worker completed b
2018/06/04 15:56:33 C worker completed c
2018/06/04 15:56:33 D worker completed d
2018/06/04 15:56:33 abcd assembled.  cycle 1 complete
2018/06/04 15:56:33 begin assembly cycle 2
2018/06/04 15:56:33 A worker begins part
2018/06/04 15:56:33 B worker begins part
2018/06/04 15:56:33 C worker begins part
2018/06/04 15:56:33 D worker begins part
2018/06/04 15:56:33 A worker completed a
2018/06/04 15:56:33 B worker completed b
2018/06/04 15:56:33 D worker completed d
2018/06/04 15:56:33 C worker completed c
2018/06/04 15:56:33 abcd assembled.  cycle 2 complete
2018/06/04 15:56:33 begin assembly cycle 3
2018/06/04 15:56:33 A worker begins part
2018/06/04 15:56:33 B worker begins part
2018/06/04 15:56:33 C worker begins part
2018/06/04 15:56:33 D worker begins part
2018/06/04 15:56:33 B worker completed b
2018/06/04 15:56:33 A worker completed a
2018/06/04 15:56:33 D worker completed d
2018/06/04 15:56:33 C worker completed c
2018/06/04 15:56:33 abcd assembled.  cycle 3 complete
$

```

Solution 3, two-phase barrier

For those that might object to the way the two solutions above start new goroutines in each cycle, here is a technique sometimes called a two-phase barrier, where goroutines loop until being shutdown. In each loop there are two phases, one of making the part, and one of waiting for the completed parts to be assembled. This more literally satisfies the task but in fact is not idiomatic Go. Goroutines are cheap to start up and shut down in Go and the extra complexity of this two-phase barrier technique is not justified.

```go
package main
 
import (
    "log"
    "math/rand"
    "strings"
    "sync"
    "time"
)
 
func worker(part string, completed chan string) {
    log.Println(part, "worker running")
    for {
        select {
        case <-start:
            log.Println(part, "worker begins part")
            time.Sleep(time.Duration(rand.Int63n(1e6)))
            p := strings.ToLower(part)
            log.Println(part, "worker completed", p)
            completed <- p
            <-reset
            wg.Done()
        case <-done:
            log.Println(part, "worker stopped")
            wg.Done()
            return
        }
    }
}
 
var (
    partList    = []string{"A", "B", "C", "D"}
    nAssemblies = 3
    start       = make(chan int)
    done        = make(chan int)
    reset       chan int
    wg          sync.WaitGroup
)
 
func main() {
    rand.Seed(time.Now().UnixNano())
    completed := make([]chan string, len(partList))
    for i, part := range partList {
        completed[i] = make(chan string)
        go worker(part, completed[i])
    }
    for c := 1; c <= nAssemblies; c++ {
        log.Println("begin assembly cycle", c)
        reset = make(chan int)
        close(start)
        a := ""
        for _, c := range completed {
            a += <-c
        }
        log.Println(a, "assembled.  cycle", c, "complete")
        wg.Add(len(partList))
        start = make(chan int)
        close(reset)
        wg.Wait()
    }
    wg.Add(len(partList))
    close(done)
    wg.Wait()
}
```

Output:

```go
$ go run -race r3.go
2018/06/04 16:11:54 A worker running
2018/06/04 16:11:54 B worker running
2018/06/04 16:11:54 C worker running
2018/06/04 16:11:54 begin assembly cycle 1
2018/06/04 16:11:54 A worker begins part
2018/06/04 16:11:54 D worker running
2018/06/04 16:11:54 C worker begins part
2018/06/04 16:11:54 B worker begins part
2018/06/04 16:11:54 D worker begins part
2018/06/04 16:11:54 A worker completed a
2018/06/04 16:11:54 C worker completed c
2018/06/04 16:11:54 D worker completed d
2018/06/04 16:11:54 B worker completed b
2018/06/04 16:11:54 abcd assembled.  cycle 1 complete
2018/06/04 16:11:54 begin assembly cycle 2
2018/06/04 16:11:54 C worker begins part
2018/06/04 16:11:54 D worker begins part
2018/06/04 16:11:54 B worker begins part
2018/06/04 16:11:54 A worker begins part
2018/06/04 16:11:54 D worker completed d
2018/06/04 16:11:54 A worker completed a
2018/06/04 16:11:54 B worker completed b
2018/06/04 16:11:54 C worker completed c
2018/06/04 16:11:54 abcd assembled.  cycle 2 complete
2018/06/04 16:11:54 begin assembly cycle 3
2018/06/04 16:11:54 A worker begins part
2018/06/04 16:11:54 D worker begins part
2018/06/04 16:11:54 C worker begins part
2018/06/04 16:11:54 B worker begins part
2018/06/04 16:11:54 D worker completed d
2018/06/04 16:11:54 A worker completed a
2018/06/04 16:11:54 B worker completed b
2018/06/04 16:11:54 C worker completed c
2018/06/04 16:11:54 abcd assembled.  cycle 3 complete
2018/06/04 16:11:54 D worker stopped
2018/06/04 16:11:54 B worker stopped
2018/06/04 16:11:54 C worker stopped
2018/06/04 16:11:54 A worker stopped

```

Solution 4, workers joining and leaving

This solution shows workers joining and leaving, although it is a rather different interpretation of the task.

```go
package main
 
import (
    "log"
    "math/rand"
    "os"
    "sync"
    "time"
)
 
const nMech = 5
const detailsPerMech = 4
 
var l = log.New(os.Stdout, "", 0)
 
func main() {
    assemble := make(chan int)
    var complete sync.WaitGroup
 
    go solicit(assemble, &complete, nMech*detailsPerMech)
 
    for i := 1; i <= nMech; i++ {
        complete.Add(detailsPerMech)
        for j := 0; j < detailsPerMech; j++ {
            assemble <- 0
        }
        // Go checkpoint feature
        complete.Wait()
        // checkpoint reached
        l.Println("mechanism", i, "completed")
    }
}
 
func solicit(a chan int, c *sync.WaitGroup, nDetails int) {
    rand.Seed(time.Now().UnixNano())
    var id int // worker id, for output
    for nDetails > 0 {
        // some random time to find a worker
        time.Sleep(time.Duration(5e8 + rand.Int63n(5e8)))
        id++
        // contract to assemble a certain number of details
        contract := rand.Intn(5) + 1
        if contract > nDetails {
            contract = nDetails
        }
        dword := "details"
        if contract == 1 {
            dword = "detail"
        }
        l.Println("worker", id, "contracted to assemble", contract, dword)
        go worker(a, c, contract, id)
        nDetails -= contract
    }
}
 
func worker(a chan int, c *sync.WaitGroup, contract, id int) {
    // some random time it takes for this worker to assemble a detail
    assemblyTime := time.Duration(5e8 + rand.Int63n(5e8))
    l.Println("worker", id, "enters shop")
    for i := 0; i < contract; i++ {
        <-a
        l.Println("worker", id, "assembling")
        time.Sleep(assemblyTime)
        l.Println("worker", id, "completed detail")
        c.Done()
    }
    l.Println("worker", id, "leaves shop")
}
```

Output:

```go
worker 1 contracted to assemble 2 details
worker 1 enters shop
worker 1 assembling
worker 2 contracted to assemble 5 details
worker 2 enters shop
worker 2 assembling
worker 1 completed detail
worker 1 assembling
worker 2 completed detail
worker 2 assembling
worker 3 contracted to assemble 1 detail
worker 3 enters shop
worker 1 completed detail
worker 1 leaves shop
worker 2 completed detail
mechanism 1 completed
worker 3 assembling
worker 2 assembling

...

worker 5 completed detail
worker 7 completed detail
worker 7 leaves shop
mechanism 4 completed
worker 6 assembling
worker 5 assembling
worker 6 completed detail
worker 6 assembling
worker 5 completed detail
worker 5 leaves shop
worker 6 completed detail
worker 6 assembling
worker 6 completed detail
worker 6 leaves shop
mechanism 5 completed
```

# Active object     :concurrency:object_oriented:<a id="sec-9"></a>

In object-oriented programming an object is active when its state depends on clock. Usually an active object encapsulates a task that updates the object's state. To the outer world the object looks like a normal object with methods that can be called from outside. Implementation of such methods must have a certain synchronization mechanism with the encapsulated task in order to prevent object's state corruption.

A typical instance of an active object is an animation widget. The widget state changes with the time, while as an object it has all properties of a normal widget.

The task

Implement an active integrator object. The object has an input and output. The input can be set using the method Input. The input is a function of time. The output can be queried using the method Output. The object integrates its input over the time and the result becomes the object's output. So if the input is K(t) and the output is S, the object state S is changed to S + (K(t1) + K(t0)) \* (t1 - t0) / 2, i.e. it integrates K using the trapeze method. Initially K is constant 0 and S is 0.

In order to test the object:

set its input to sin (2π f t), where the frequency f=0.5Hz. The phase is irrelevant. wait 2s set the input to constant 0 wait 0.5s

Verify that now the object's output is approximately 0 (the sine has the period of 2s). The accuracy of the result will depend on the OS scheduler time slicing and the accuracy of the clock.

Using time.Tick to sample K at a constant frequency. Three goroutines are involved, main, aif, and tk. Aif controls access to the accumulator s and the integration function K. Tk and main must talk to aif through channels to access s and K.

```go
package main
 
import (
    "fmt"
    "math"
    "time"
)
 
// type for input function, k.
// input is duration since an arbitrary start time t0.
type tFunc func(time.Duration) float64
 
// active integrator object.  state variables are not here, but in
// function aif, started as a goroutine in the constructor.
type aio struct {
    iCh chan tFunc        // channel for setting input function
    oCh chan chan float64 // channel for requesting output
}
 
// constructor
func newAio() *aio {
    var a aio
    a.iCh = make(chan tFunc)
    a.oCh = make(chan chan float64)
    go aif(&a)
    return &a
}
 
// input method required by task description.  in practice, this method is
// unnecessary; you would just put that single channel send statement in
// your code wherever you wanted to set the input function.
func (a aio) input(f tFunc) {
    a.iCh <- f
}
 
// output method required by task description.  in practice, this method too
// would not likely be best.  instead any client interested in the value would
// likely make a return channel sCh once, and then reuse it as needed.
func (a aio) output() float64 {
    sCh := make(chan float64)
    a.oCh <- sCh
    return <-sCh
}
 
// integration function that returns constant 0
func zeroFunc(time.Duration) float64 { return 0 }
 
// goroutine serializes access to integrated function k and state variable s
func aif(a *aio) {
    var k tFunc = zeroFunc // integration function
    s := 0.                // "object state" initialized to 0
    t0 := time.Now()       // initial time
    k0 := k(0)             // initial sample value
    t1 := t0               // t1, k1 used for trapezoid formula
    k1 := k0
 
    tk := time.Tick(10 * time.Millisecond) // 10 ms -> 100 Hz
    for {
        select {
        case t2 := <-tk: // timer tick event
            k2 := k(t2.Sub(t0))                        // new sample value
            s += (k1 + k2) * .5 * t2.Sub(t1).Seconds() // trapezoid formula
            t1, k1 = t2, k2                            // save time and value
        case k = <-a.iCh: // input method event: function change
        case sCh := <-a.oCh: // output method event: sample object state
            sCh <- s
        }
    }
}
 
func main() {
    a := newAio()                           // create object
    a.input(func(t time.Duration) float64 { // 1. set input to sin function
        return math.Sin(t.Seconds() * math.Pi)
    })
    time.Sleep(2 * time.Second) // 2. sleep 2 sec
    a.input(zeroFunc)           // 3. set input to zero function
    time.Sleep(time.Second / 2) // 4. sleep .5 sec
    fmt.Println(a.output())     // output should be near zero
}
```

Output:

```go
2.4517135756807704e-05

```

# Metered concurrency     :concurrency:<a id="sec-10"></a>

The goal of this task is to create a counting semaphore used to control the execution of a set of concurrent units. This task intends to demonstrate coordination of active concurrent units through the use of a passive concurrent unit. The operations for a counting semaphore are acquire, release, and count. Each active concurrent unit should attempt to acquire the counting semaphore before executing its assigned duties. In this case the active concurrent unit should report that it has acquired the semaphore. It should sleep for 2 seconds and then release the semaphore.

Buffered channel[edit]

Recommended solution for simplicity. Acquire operation is channel send, release is channel receive, and count is provided with cap and len.

To demonstrate, this example implements the Library analogy from Wikipedia with 10 study rooms and 20 students.

The channel type shown here is struct{}. struct{} is nice because it has zero size and zero content, although the syntax is slightly akward. Other popular choices for no-content tokens are ints and bools. They read a little nicer but waste a few bytes and could potentially mislead someone to think the values had some meaning.

A couple of other concurrency related details used in the example are the log package for serializing output and sync.WaitGroup used as a completion checkpoint. Functions of the fmt package are not synchronized and can produce interleaved output with concurrent writers. The log package does nice synchronization to avoid this.

```go
package main
 
import (
    "log"
    "os"
    "sync"
    "time"
)
 
// counting semaphore implemented with a buffered channel
type sem chan struct{}
 
func (s sem) acquire()   { s <- struct{}{} }
func (s sem) release()   { <-s }
func (s sem) count() int { return cap(s) - len(s) }
 
// log package serializes output
var fmt = log.New(os.Stdout, "", 0)
 
// library analogy per WP article
const nRooms = 10
const nStudents = 20
 
func main() {
    rooms := make(sem, nRooms)
    // WaitGroup used to wait for all students to have studied
    // before terminating program
    var studied sync.WaitGroup
    studied.Add(nStudents)
    // nStudents run concurrently
    for i := 0; i < nStudents; i++ {
        go student(rooms, &studied)
    }
    studied.Wait()
}
 
func student(rooms sem, studied *sync.WaitGroup) {
    rooms.acquire()
    // report per task descrption.  also exercise count operation
    fmt.Printf("Room entered.  Count is %d.  Studying...\n",
        rooms.count())
    time.Sleep(2 * time.Second) // sleep per task description
    rooms.release()
    studied.Done() // signal that student is done
}
```

Output for this and the other Go programs here shows 10 students studying immediately, about a 2 second pause, 10 more students studying, then another pause of about 2 seconds before returning to the command prompt. In this example the count values may look jumbled. This is a result of the student goroutines running concurrently.

Sync.Cond[edit]

A more traditional approach implementing a counting semaphore object with sync.Cond. It has a constructor and methods for the three operations requested by the task.

```go
package main
 
import (
    "log"
    "os"
    "sync"
    "time"
)
 
var fmt = log.New(os.Stdout, "", 0)
 
type countSem struct {
    int
    sync.Cond
}
 
func newCount(n int) *countSem {
    return &countSem{n, sync.Cond{L: &sync.Mutex{}}}
}
 
func (cs *countSem) count() int {
    cs.L.Lock()
    c := cs.int
    cs.L.Unlock()
    return c
}
 
func (cs *countSem) acquire() {
    cs.L.Lock()
    cs.int--
    for cs.int < 0 {
        cs.Wait()
    }
    cs.L.Unlock()
}
 
func (cs *countSem) release() {
    cs.L.Lock()
    cs.int++
    cs.L.Unlock()
    cs.Broadcast()
}
 
func main() {
    librarian := newCount(10)
    nStudents := 20
    var studied sync.WaitGroup
    studied.Add(nStudents)
    for i := 0; i < nStudents; i++ {
        go student(librarian, &studied)
    }
    studied.Wait()
}
 
func student(studyRoom *countSem, studied *sync.WaitGroup) {
    studyRoom.acquire()
    fmt.Printf("Room entered.  Count is %d.  Studying...\n", studyRoom.count())
    time.Sleep(2 * time.Second)
    studyRoom.release()
    studied.Done()
}
```

# Rendezvous     :encyclopedia:concurrency:<a id="sec-11"></a>

Demonstrate the “rendezvous” communications technique by implementing a printer monitor.

```go
package main
 
import (
    "errors"
    "fmt"
    "strings"
    "sync"
)
 
var hdText = `Humpty Dumpty sat on a wall.
Humpty Dumpty had a great fall.
All the king's horses and all the king's men,
Couldn't put Humpty together again.`
 
var mgText = `Old Mother Goose,
When she wanted to wander,
Would ride through the air,
On a very fine gander.
Jack's mother came in,
And caught the goose soon,
And mounting its back,
Flew up to the moon.`
 
func main() {
    reservePrinter := startMonitor(newPrinter(5), nil)
    mainPrinter := startMonitor(newPrinter(5), reservePrinter)
    var busy sync.WaitGroup
    busy.Add(2)
    go writer(mainPrinter, "hd", hdText, &busy)
    go writer(mainPrinter, "mg", mgText, &busy)
    busy.Wait()
}
 
// printer is a type representing an abstraction of a physical printer.
// It is a type defintion for a function that takes a string to print
// and returns an error value, (hopefully usually nil, meaning no error.)
type printer func(string) error
 
// newPrinter is a constructor.  The parameter is a quantity of ink.  It
// returns a printer object encapsulating the ink quantity.
// Note that this is not creating the monitor, only the object serving as
// a physical printer by writing to standard output.
func newPrinter(ink int) printer {
    return func(line string) error {
        if ink == 0 {
            return eOutOfInk
        }
        for _, c := range line {
            fmt.Printf("%c", c)
        }
        fmt.Println()
        ink--
        return nil
    }
}
 
var eOutOfInk = errors.New("out of ink")
 
// For the language task, rSync is a type used to approximate the Ada
// rendezvous mechanism that includes the caller waiting for completion
// of the callee.  For this use case, we signal completion with an error
// value as a response.  Exceptions are not idiomatic in Go and there is
// no attempt here to model the Ada exception mechanism.  Instead, it is
// idomatic in Go to return error values.  Sending an error value on a
// channel works well here to signal completion.  Go unbuffered channels
// provide synchronous rendezvous, but call and response takes two channels,
// which are bundled together here in a struct.  The channel types are chosen
// to mirror the parameter and return types of "type printer" defined above.
// The channel types here, string and error are both "reference types"
// in Go terminology.  That is, they are small things containing pointers
// to the actual data.  Sending one on a channel does not involve copying,
// or much less marshalling string data.
type rSync struct {
    call     chan string
    response chan error
}
 
// "rendezvous Print" requested by use case task.
// For the language task though, it is implemented here as a method on
// rSync that sends its argument on rSync.call and returns the result
// received from rSync.response.  Each channel operation is synchronous.
// The two operations back to back approximate the Ada rendezvous.
func (r *rSync) print(data string) error {
    r.call <- data      // blocks until data is accepted on channel
    return <-r.response // blocks until response is received
}
 
// monitor is run as a goroutine.  It encapsulates the printer passed to it.
// Print requests are received through the rSync object "entry," named entry
// here to correspond to the Ada concept of an entry point.
func monitor(hardPrint printer, entry, reserve *rSync) {
    for {
        // The monitor goroutine will block here waiting for a "call"
        // to its "entry point."
        data := <-entry.call
        // Assuming the call came from a goroutine calling rSync.print,
        // that goroutine is now blocked, waiting for this one to send
        // a response.
 
        // attempt output
        switch err := hardPrint(data); {
 
        // consider return value from attempt
        case err == nil:
            entry.response <- nil // no problems
 
        case err == eOutOfInk && reserve != nil:
            // Requeue to "entry point" of reserve printer monitor.
            // Caller stays blocked, and now this goroutine blocks until
            // it gets a response from the reserve printer monitor.
            // It then transparently relays the response to the caller.
            entry.response <- reserve.print(data)
 
        default:
            entry.response <- err // return failure
        }
        // The response is away.  Loop, and so immediately block again.
    }
}
 
// startMonitor can be seen as an rSync constructor.  It also
// of course, starts the monitor for which the rSync serves as entry point.
// Further to the langauge task, note that the channels created here are
// unbuffered.  There is no buffer or message box to hold channel data.
// A sender will block waiting for a receiver to accept data synchronously.
func startMonitor(p printer, reservePrinter *rSync) *rSync {
    entry := &rSync{make(chan string), make(chan error)}
    go monitor(p, entry, reservePrinter)
    return entry
}
 
// Two writer tasks are started as goroutines by main.  They run concurrently
// and compete for printers as resources.  Note the call to "rendezvous Print"
// as requested in the use case task and compare the syntax,
//    Here:           printMonitor.print(line);
//    Ada solution:   Main.Print ("string literal");
func writer(printMonitor *rSync, id, text string, busy *sync.WaitGroup) {
    for _, line := range strings.Split(text, "\n") {
        if err := printMonitor.print(line); err != nil {
            fmt.Printf("**** writer task %q terminated: %v ****\n", id, err)
            break
        }
    }
    busy.Done()
}
```

Output:

\#+BEGIN<sub>SRC</sub> go Humpty Dumpty sat on a wall. Old Mother Goose, Humpty Dumpty had a great fall. When she wanted to wander, All the king's horses and all the king's men, Would ride through the air, Couldn't put Humpty together again. On a very fine gander. Jack's mother came in, And caught the goose soon,

1.  writer task "mg" terminated: out of ink **\*\***

    \#+END<sub>SRC</sub>
