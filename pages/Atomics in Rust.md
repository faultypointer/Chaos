- An atomic operation is an operation that is indivisible. At the hardware level, these are the operations that the cpu cannot pause in the middle and resume later.
- Actually, I don't know how atomic operations work at the hardware level or how they are implemented in rust. I can only guess that is what it means for an operation to be atomic. And with multiple cores I really don't know anything.
- Okay I looked into it very briefly.
	- The atomic operation in high level language gets compiled into cpu specific atomic operation like LOCK CMPXCHG that lock the memory bus/cache line.
	- At the hardware level, atomicity relies on something called [cache coherency protocols](https://en.wikipedia.org/wiki/List_of_cache_coherency_protocols), bus locking and other stuff,
- Atomic operations prevent undefined behavior (when used correctly) in multi-threaded programs since each atomic operation happen either before or after other atomic operations.
- Mutex, CondVar etc are implemented using atomic operations in rust (and probably in other languages as well).
- Atomic types in rust live in `std::sync::atomic` module. Eg: `AtomicUsize`, `AtomicI32`. These types allow modifications through shared references. All these types provide same interface with methods for reading, writing, updating operations. All these operations take `std::sync::atomic::Ordering` as an argument. `Ordering` determines the relative ordering of such operations. More about this in [Memory Ordering]([[Memory Ordering in Rust]]). (maybe [[Jun 23rd, 2025]])
- ## Load and Store
	- `load()` atomically loads the value stored in the atomic variable while `store()` stores a new value in it (also atomically).
	- ### Example: Stop flag
		- ```rust
		  use std::sync::atomic::AtomicBool;
		  use std::sync::atomic::Ordering::Relaxed;
		  fn main() {
		      static STOP: AtomicBool = AtomicBool::new(false);
		      // Spawn a thread to do the work.
		      let background_thread = thread::spawn(|| {
		          while !STOP.load(Relaxed) {
		              some_work();
		          }
		      });
		      // Use the main thread to listen for user input.
		      for line in std::io::stdin().lines() {
		          match line.unwrap().as_str() {
		              "help" => println!("commands: help, stop"),
		              "stop" => break,
		              cmd => println!("unknown command: {cmd:?}"),
		          }
		      }
		      // Inform the background thread it needs to stop.
		      STOP.store(true, Relaxed);
		      // Wait until the background thread finishes.
		      background_thread.join().unwrap();
		  }
		  ```
	- ### Example: Reporting Progress
		- ```rust
		  use std::sync::atomic::AtomicUsize;
		  fn main() {
		      let num_done = AtomicUsize::new(0);
		      thread::scope(|s| {
		          // A background thread to process all 100 items.
		          s.spawn(|| {
		              for i in 0..100 {
		                  process_item(i); // Assuming this takes some time.
		                  num_done.store(i + 1, Relaxed);
		              }
		          });
		          // The main thread shows status updates, every second.
		          loop {
		              let n = num_done.load(Relaxed);
		              if n == 100 { break; }
		              println!("Working.. {n}/100 done");
		              thread::sleep(Duration::from_secs(1));
		          }
		      });
		      println!("Done!");
		  }
		  ```
	- ### Reporting Progress + Synchronization
		- ```rust
		  fn main() {
		      let num_done = AtomicUsize::new(0);
		      let main_thread = thread::current();
		      thread::scope(|s| {
		          // A background thread to process all 100 items.
		          s.spawn(|| {
		              for i in 0..100 {
		                  process_item(i); // Assuming this takes some time.
		                  num_done.store(i + 1, Relaxed);
		                  main_thread.unpark(); // Wake up the main thread.
		              }
		          });
		          // The main thread shows status updates.
		          loop {
		              let n = num_done.load(Relaxed);
		              if n == 100 { break; }
		              println!("Working.. {n}/100 done");
		              thread::park_timeout(Duration::from_secs(1));
		          }
		      });
		      println!("Done!");
		  }
		  ```
	- ### Example: Lazy Initialization
		- ```rust
		  use std::sync::atomic::AtomicU64;
		  fn get_x() -> u64 {
		      static X: AtomicU64 = AtomicU64::new(0);
		      let mut x = X.load(Relaxed);
		      if x == 0 {
		          x = calculate_x();
		          X.store(x, Relaxed);
		      }
		      x
		  }
		  ```
		- Use `std::sync::Once`, `std::sync::OnceLock` if new threads need to wait while the first thread is still initializing the value.
- ## Fetch and Modify
	- Fetch the old value from the atomic variable and modify the variable as a single atomic operation.
	- Example of such operations for `AtomicI32` are `fetch_add`, `fetch_min`, `fetch_and` etc.
	- ### Example: Progress Reporting from multiple threads
		- ```rust
		  fn main() {
		      let num_done = &AtomicUsize::new(0);
		      thread::scope(|s| {
		          // Four background threads to process all 100 items, 25 each.
		          for t in 0..4 {
		              s.spawn(move || {
		                  for i in 0..25 {
		                      process_item(t * 25 + i); // Assuming this takes some time.
		                      num_done.fetch_add(1, Relaxed);
		                  }
		              });
		          }
		          // The main thread shows status updates, every second.
		          loop {
		              let n = num_done.load(Relaxed);
		              if n == 100 { break; }
		              println!("Working.. {n}/100 done");
		              thread::sleep(Duration::from_secs(1));
		          }
		      });
		      println!("Done!");
		  }
		  ```
		- Since `t` only exists for the duration of the loop, we need to move it to the threads instead of threads referencing t. Hence the `move` keyword in `spawn` method. We do want the reference of the same `AtomicUsize` so num_done is a reference as the `move` would also copy the `num_done` variable.
	- ### Example: Statistics
		- ```rust
		  fn main() {
		      let num_done = &AtomicUsize::new(0);
		      let total_time = &AtomicU64::new(0);
		      let max_time = &AtomicU64::new(0);
		      thread::scope(|s| {
		          // Four background threads to process all 100 items, 25 each.
		          for t in 0..4 {
		              s.spawn(move || {
		                  for i in 0..25 {
		                      let start = Instant::now();
		                      process_item(t * 25 + i); // Assuming this takes some time.
		                      let time_taken = start.elapsed().as_micros() as u64;
		                      num_done.fetch_add(1, Relaxed);
		                      total_time.fetch_add(time_taken, Relaxed);
		                      max_time.fetch_max(time_taken, Relaxed);
		                  }
		              });
		          }
		          // The main thread shows status updates, every second.
		          loop {
		              let total_time = Duration::from_micros(total_time.load(Relaxed));
		              let max_time = Duration::from_micros(max_time.load(Relaxed));
		              let n = num_done.load(Relaxed);
		              if n == 100 { break; }
		              if n == 0 {
		                  println!("Working.. nothing done yet.");
		              } else {
		                  println!(
		                      "Working.. {n}/100 done, {:?} average, {:?} peak",
		                      total_time / n as u32,
		                      max_time,
		                  );
		              }
		              thread::sleep(Duration::from_secs(1));
		          }
		      });
		      println!("Done!");
		  }
		  ```
	- ### Example: ID Allocation
		- ```rust
		  use std::sync::atomic::AtomicU32;
		  
		  fn allocate_new_id() -> u32 {
		      static NEXT_ID: AtomicU32 = AtomicU32::new(0);
		      NEXT_ID.fetch_add(1, Relaxed)
		  }
		  ```
		- `fetch_add` has wrapping behavior. so after the `u32::MAX` the function returns 0, 1 and so on.
- ## Compare and Exchange
	- Check if the value of atomic variable is equal to a given value and if it is, replace it with another value.
	- Returns `Result<i32, i32>` in case of `AtomicI32`. That is returns the old value wrapped in `Ok` if the value was replaced or in `Err` if the compare failed.
	- ### Example: ID Allocation Without Overflow
		- ```rust
		  fn allocate_new_id() -> u32 {
		      static NEXT_ID: AtomicU32 = AtomicU32::new(0);
		      let mut id = NEXT_ID.load(Relaxed);
		      loop {
		          assert!(id < 1000, "too many IDs!");
		          match NEXT_ID.compare_exchange_weak(id, id + 1, Relaxed, Relaxed) {
		              Ok(_) => return id,
		              Err(v) => id = v,
		          }
		      }
		  }
		  ```
	- ### Example: Lazy One-Time Initialization
		- ```rust
		  fn get_key() -> u64 {
		      static KEY: AtomicU64 = AtomicU64::new(0);
		      let key = KEY.load(Relaxed);
		      if key == 0 {
		          let new_key = generate_random_key(); 1
		          match KEY.compare_exchange(0, new_key, Relaxed, Relaxed) { 2
		              Ok(_) => new_key, 3
		              Err(k) => k, 4
		          }
		      } else {
		          key
		      }
		  }
		  ```
-
- #rust #low-level #os