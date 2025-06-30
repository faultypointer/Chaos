- used to avoid repeated calculations
- often the problems solved by dynamic programming are solved naturally using recursion
- in such cases, saving the repeated state in a lookup table to improve performance is basically top down dynamic programming with memoization
- ## An Example with Fibonacci
	- ```c++
	  int f(int n) {
	    if (n < 2) return n;
	    return f(n-1) + f(n-2);
	  }
	  ```
	- This code results in over a million call for `f(29)`
	- the function with same n is called multiple times. we can store the value of `f(n)` for the n we have calculated once for later use
	- ```c++
	  const int MAXN = 100;
	  bool found[MAXN];
	  int memo[MAXN];
	  
	  int f(int n) {
	      if (found[n]) return memo[n];
	      if (n == 0) return 0;
	      if (n == 1) return 1;
	  
	      found[n] = true;
	      return memo[n] = f(n - 1) + f(n - 2);
	  }
	  ```
	-
	-