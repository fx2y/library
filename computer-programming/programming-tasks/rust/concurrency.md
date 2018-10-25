- [Concurrent computing](#sec-1)
- [Synchronous concurrency](#sec-2)
- [Dining philosophers](#sec-3)
- [Atomic updates](#sec-4)
- [Determine if only one instance is running](#sec-5)

# Concurrent computing<a id="sec-1"></a>

Task

Using either native language concurrency syntax or freely available libraries, write a program to display the strings "Enjoy" "Rosetta" "Code", one string per line, in random order.

Concurrency syntax must use threads, tasks, co-routines, or whatever concurrency is called in your language.

Library: rand

```rust
extern crate rand;
use std::thread;
use rand::thread_rng;
use rand::distributions::{Range, IndependentSample};
 
fn main() {
    let mut rng = thread_rng();
    let rng_range = Range::new(0u32, 100);
    for word in "Enjoy Rosetta Code".split_whitespace() {
        let snooze_time = rng_range.ind_sample(&mut rng);
        let local_word = word.to_owned();
        std::thread::spawn(move || {
            thread::sleep_ms(snooze_time);
            println!("{}", local_word);
        });
    }
    thread::sleep_ms(1000);
}
```

# Synchronous concurrency<a id="sec-2"></a>

The goal of this task is to create two concurrent activities ("Threads" or "Tasks", not processes.) that share data synchronously. Your language may provide syntax or libraries to perform concurrency. Different languages provide different implementations of concurrency, often with different names. Some languages use the term threads, others use the term tasks, while others use co-processes. This task should not be implemented using fork, spawn, or the Linux/UNIX/Win32 pipe command, as communication should be between threads, not processes.

One of the concurrent units will read from a file named "input.txt" and send the contents of that file, one line at a time, to the other concurrent unit, which will print the line it receives to standard output. The printing unit must count the number of lines it prints. After the concurrent unit reading the file sends its last line to the printing unit, the reading unit will request the number of lines printed by the printing unit. The reading unit will then print the number of lines printed by the printing unit.

This task requires two-way communication between the concurrent units. All concurrent units must cleanly terminate at the end of the program.

Works with: rustc 1.4.0-nightly version f84d53ca0 2015-09-06

```rust
use std::fs::File;
use std::io::BufReader;
use std::io::BufRead;
 
use std::thread::spawn;
use std::sync::mpsc::{SyncSender, Receiver, sync_channel};
 
fn main() {
    let (tx, rx): (SyncSender<String>, Receiver<String>) = sync_channel::<String>(0);
 
    // Reader thread.
    spawn(move || {
        let file = File::open("input.txt").unwrap();
        let reader = BufReader::new(file);
 
        for line in reader.lines() {
            match line {
                Ok(msg) => tx.send(msg).unwrap(),
                Err(e) => println!("{}", e)
            }
        }
 
        drop(tx);
    });
 
    // Writer thread.
    spawn(move || {
        let mut loop_count: u16 = 0;
 
        loop {
            let recvd = rx.recv();
 
            match recvd {
                Ok(msg) => {
                    println!("{}", msg);
                    loop_count += 1;
                },
                Err(_) => break // rx.recv() will only err when tx is closed.
            }
        }
 
        println!("Line count: {}", loop_count);
    }).join().unwrap();
}
```

# Dining philosophers<a id="sec-3"></a>

The dining philosophers problem illustrates non-composability of low-level synchronization primitives like semaphores. It is a modification of a problem posed by Edsger Dijkstra.

Five philosophers, Aristotle, Kant, Spinoza, Marx, and Russell (the tasks) spend their time thinking and eating spaghetti. They eat at a round table with five individual seats. For eating each philosopher needs two forks (the resources). There are five forks on the table, one left and one right of each seat. When a philosopher cannot grab both forks it sits and waits. Eating takes random time, then the philosopher puts the forks down and leaves the dining room. After spending some random time thinking about the nature of the universe, he again becomes hungry, and the circle repeats itself.

It can be observed that a straightforward solution, when forks are implemented by semaphores, is exposed to deadlock. There exist two deadlock states when all five philosophers are sitting at the table holding one fork each. One deadlock state is when each philosopher has grabbed the fork left of him, and another is when each has the fork on his right.

There are many solutions of the problem, program at least one, and explain how the deadlock is prevented.

A Rust implementation of a solution for the Dining Philosophers Problem. We prevent a deadlock by using Dijkstra's solution of making a single diner "left-handed." That is, all diners except one pick up the chopstick "to their left" and then the chopstick "to their right." The remaining diner performs this in reverse.

```rust
use std::thread;
use std::sync::{Mutex, Arc};
 
struct Philosopher {
    name: String,
    left: usize,
    right: usize,
}
 
impl Philosopher {
    fn new(name: &str, left: usize, right: usize) -> Philosopher {
        Philosopher {
            name: name.to_string(),
            left: left,
            right: right,
        }
    }
 
    fn eat(&self, table: &Table) {
        let _left = table.forks[self.left].lock().unwrap();
        let _right = table.forks[self.right].lock().unwrap();
 
        println!("{} is eating.", self.name);
 
        thread::sleep_ms(1000);
 
        println!("{} is done eating.", self.name);
    }
}
 
struct Table {
    forks: Vec<Mutex<()>>,
}
 
fn main() {
    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});
 
    let philosophers = vec![
        Philosopher::new("Baruch Spinoza", 0, 1),
        Philosopher::new("Gilles Deleuze", 1, 2),
        Philosopher::new("Karl Marx", 2, 3),
        Philosopher::new("Friedrich Nietzsche", 3, 4),
        Philosopher::new("Michel Foucault", 0, 4),
    ];
 
    let handles: Vec<_> = philosophers.into_iter().map(|p| {
        let table = table.clone();
 
        thread::spawn(move || {
            p.eat(&table);
        })
    }).collect();
 
    for h in handles {
        h.join().unwrap();
    }
}
```

# Atomic updates<a id="sec-4"></a>

Task

Define a data type consisting of a fixed number of 'buckets', each containing a nonnegative integer value, which supports operations to:

get the current value of any bucket remove a specified amount from one specified bucket and add it to another, preserving the total of all bucket values, and clamping the transferred amount to ensure the values remain non-negative

In order to exercise this data type, create one set of buckets, and start three concurrent tasks:

As often as possible, pick two buckets and make their values closer to equal. As often as possible, pick two buckets and arbitrarily redistribute their values. At whatever rate is convenient, display (by any means) the total value and, optionally, the individual values of each bucket.

The display task need not be explicit; use of e.g. a debugger or trace tool is acceptable provided it is simple to set up to provide the display.

This task is intended as an exercise in atomic operations.   The sum of the bucket values must be preserved even if the two tasks attempt to perform transfers simultaneously, and a straightforward solution is to ensure that at any time, only one transfer is actually occurring — that the transfer operation is atomic.

Library: rand

```rust
extern crate rand;
 
use std::sync::{Arc, Mutex};
use std::thread;
use std::cmp;
use std::time::Duration;
 
use rand::Rng;
use rand::distributions::{IndependentSample, Range};
 
trait Buckets {
    fn equalize<R:Rng>(&mut self, rng: &mut R);
    fn randomize<R:Rng>(&mut self, rng: &mut R);
    fn print_state(&self);
}
 
impl Buckets for [i32] {
    fn equalize<R:Rng>(&mut self, rng: &mut R) {
        let range = Range::new(0,self.len()-1);
        let src = range.ind_sample(rng);
        let dst = range.ind_sample(rng);
        if dst != src {
            let amount = cmp::min(((dst + src) / 2) as i32, self[src]);
            let multiplier = if amount >= 0 { -1 } else { 1 };
            self[src] += amount * multiplier;
            self[dst] -= amount * multiplier;
        }
    }
    fn randomize<R:Rng>(&mut self, rng: &mut R) {
        let ind_range = Range::new(0,self.len()-1);
        let src = ind_range.ind_sample(rng);
        let dst = ind_range.ind_sample(rng);
        if dst != src {
            let amount = cmp::min(Range::new(0,20).ind_sample(rng), self[src]);
            self[src] -= amount;
            self[dst] += amount;
 
        }
    }
    fn print_state(&self) {
        println!("{:?} = {}", self, self.iter().sum::<i32>());
    }
}
 
fn main() {
    let e_buckets = Arc::new(Mutex::new([10; 10]));
    let r_buckets = e_buckets.clone();
    let p_buckets = e_buckets.clone();
 
    thread::spawn(move || {
        let mut rng = rand::thread_rng();
        loop {
            let mut buckets = e_buckets.lock().unwrap();
            buckets.equalize(&mut rng);
        }
    });
    thread::spawn(move || {
        let mut rng = rand::thread_rng();
        loop {
            let mut buckets = r_buckets.lock().unwrap();
            buckets.randomize(&mut rng);
        }
    });
 
    let sleep_time = Duration::new(1,0);
    loop {
        {
            let buckets = p_buckets.lock().unwrap();
            buckets.print_state();
        }
        thread::sleep(sleep_time);
    }
}
```

# Determine if only one instance is running<a id="sec-5"></a>

This task is to determine if there is only one instance of an application running. If the program discovers that an instance of it is already running, then it should display a message indicating that it is already running and exit.

Using TCP socket

```rust
use std::net::TcpListener;
 
fn create_app_lock(port: u16) -> TcpListener {
    match TcpListener::bind(("0.0.0.0", port)) {
        Ok(socket) => {
            socket
        },
        Err(_) => {
            panic!("Couldn't lock port {}: another instance already running?", port);
        }
    }
}
 
fn remove_app_lock(socket: TcpListener) {
    drop(socket);
}
 
fn main() {
    let lock_socket = create_app_lock(12345);
    // ...
    // your code here
    // ...
    remove_app_lock(lock_socket);
}
```
