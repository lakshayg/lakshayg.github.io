Recently I had the need to debug issues in a codebase that did not produce backtraces. I was able to get the backtrace by manually triggering a segfault and using `catchsegv`.

```cpp
#include <signal.h>

void foo()
{
  // ...
  // Need to get the backtrace at this point
  raise(SIGSEGV);      // manually trigger a segfault
  // ...
}
```

Now run this binary under `catchsegv`. I was using `bazel` as my build system so this could be done using:

```sh
bazel run --run_under=catchsegv :target
```

This produces a register dump and backtrace in the output. The backtrace contains mangled symbol names which can be demangled using `c++filt`
