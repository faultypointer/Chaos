- The compilers and processors are free to reorder the instructions that don't affect each others written in the original program when there is only one thread of execution since the program behavior will be the same.
- When multiple threads are involved and we are operating on data that can be mutated through shared references (like the Atomic types) reordering instructions may alter the behavior of the program. Thus it is important to specify how the compiler and processor can order the instructions so that the behavior will stay the same.
- The way to tell the compiler about the relative ordering in rust is through the `std::sync::atomic::Ordering`.  The available options are
	- Relaxed ordering: `Ordering::Relaxed`
	- Release and acquire ordering: `Ordering::{Release, Acquire, AcqRel}`
	- Sequentially consistent ordering: `Ordering::SeqCst`
- The memory model defines the order of operations in terms of *happens-before* relationship. A *happens-before* B implies that the operation A happens before B (shock!!)
- ## Spawning and Joining
	- Spawning a thread creates a *happens-before* relationship with what happened before the `spawn()` call and the new thread
	- Similarly joining a thread creates a *happens-before* relationship with joined thread and what happens after the `join()` call
- ## Relaxed Ordering
	- This is the most permissive and performant type of memory ordering.
	- It does not create any *happens-before* relationship and thus doesn't provide any gaurantee about the relative ordering of operations on different atomic variable
	- It **does** gaurantee that all modifications of the same atomic variable happen in an order that is same from the perspective of every threads
- ## Release and Acquire Ordering
	- `Release` and `Acquire` ordering work in pair to create a *happens-before* relationship. `Release` ordering apply to store operations while `Acquire` apply to load operations.
	- When an *acquire-load* operation observes the value of *release-load* operation, a *happens-before* relation is formed between the *store* operation and everything before it and the *load* operation and everything after it.
	- With the operations like *fetch-and-modify* or *compare-and-exchange*, The `Release` ordering applies to store part only while `Acquire` applies to load part only. We can use `AcqRel` to apply both `Release` and `Acquire` to store and load part respectively.
	- ### Example: Locking
		- ```rust
		  static mut DATA: String = String::new();
		  static LOCKED: AtomicBool = AtomicBool::new(false);
		  fn f() {
		      if LOCKED.compare_exchange(false, true, Acquire, Relaxed).is_ok() {
		          // Safety: We hold the exclusive lock, so nothing else is accessing DATA.
		          unsafe { DATA.push('!') };
		          LOCKED.store(false, Release);
		      }
		  }
		  fn main() {
		      thread::scope(|s| {
		          for _ in 0..100 {
		              s.spawn(f);
		          }
		      });
		  }
		  ```
	- ### Example: Lazy Initialization of Arbitrary Data
		- ```rust
		  use std::sync::atomic::AtomicPtr;
		  fn get_data() -> &'static Data {
		      static PTR: AtomicPtr<Data> = AtomicPtr::new(std::ptr::null_mut());
		      let mut p = PTR.load(Acquire);
		      if p.is_null() {
		          p = Box::into_raw(Box::new(generate_data()));
		          if let Err(e) = PTR.compare_exchange(
		              std::ptr::null_mut(), p, Release, Acquire
		          ) {
		              // Safety: p comes from Box::into_raw right above,
		              // and wasn't shared with any other thread.
		              drop(unsafe { Box::from_raw(p) });
		              p = e;
		          }
		      }
		      // Safety: p is not null and points to a properly initialized value.
		      unsafe { &*p }
		  }
		  ```
- ## Sequentially Consistent Ordering
	- all the guarantees of Acquire and Release ordering for load and store and also guarantees a globally consistent order of operations
	- every single operations using `SeqCst` ordering is part of a single total order that all threads agree on
	- almost never necessary
	- ### Example
		- ```rust
		  use std::sync::atomic::Ordering::SeqCst;
		  static A: AtomicBool = AtomicBool::new(false);
		  static B: AtomicBool = AtomicBool::new(false);
		  static mut S: String = String::new();
		  
		  fn main() {
		      let a = thread::spawn(|| {
		          A.store(true, SeqCst);
		          if !B.load(SeqCst) {
		              unsafe { S.push('!') };
		          }
		      });
		  
		      let b = thread::spawn(|| {
		          B.store(true, SeqCst);
		          if !A.load(SeqCst) {
		              unsafe { S.push('!') };
		          }
		      });
		  
		      a.join().unwrap();
		      b.join().unwrap();
		  }
		  ```