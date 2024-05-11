# rit_dyn_arr

## A dynamic array implementation written in C  

This is an example project trying to implement **dynamic array** in C, mimicking
`std::vector` from C++.

### Features
- **Supports custom allocators**  
If you prefer to use your own custom allocator for heap allocations, you can tell
this library to use it. In fact, it is required to specify what allocator you
want to use for this library to work.

- **Similar to `std::vector`**  
This library is inspired from `std::vector`. Because C lacks support for **Generics**,
there are clear differences between this library and `std::vector`. But C++ users should
feel at home with this library.

### How to use a custom allocator
I will spare you the details and will instead show you the allocator interface 
implementation and some usage code.

**Allocater interface implementation**:
```cpp
/// @brief Custom allocator interface.
/// Functions allocating memory takes a custom allocator based off this
/// interface as a parameter.
typedef struct {
  void *(*alloc)(void *, size_t);
  void (*free)(void *, void *);
  void *(*realloc)(void *, void *, size_t, size_t);
  void *m_ctx;  // The arena, stack or etc where the memory would be allocated,
                // NULL if none
} rit_dyn_arr_allocator;
```

**Example usage**:
```cpp
#include <stdlib.h>
#include "rit_dyn_arr.h"
#define ARENA_ALLOCATOR_IMPLEMENTATION
#include "arena_allocator.h"

void *libc_malloc(void *t_ctx, size_t t_size_in_bytes) {
  (void)t_ctx;
  return malloc(t_size_in_bytes);
}

void libc_free(void *t_ctx, void *t_ptr) {
  (void)t_ctx;
  free(t_ptr);
}

void *libc_realloc(void *t_ctx, void *t_old_ptr, size_t t_old_size_in_bytes,
                   size_t t_new_size_in_bytes) {
  (void)t_ctx;
  (void)t_old_size_in_bytes;
  return realloc(t_old_ptr, t_new_size_in_bytes);
}

int main() {
  rit_dyn_arr_allocator ctx_allocator = {libc_malloc,
                                         libc_free,
                                         libc_realloc, NULL};
  rit_dyn_arr(int, arr, 10, &ctx_allocator);
}
```
Only thing here to explain is maybe what is `t_ctx` doing here. `t_ctx` points
to a context which is an arena, a stack etc(depending on what allocator you are
using) where the memory will be allocated . For the generic heap allocator 
avalaible in C, there is no context. So, we are not doing anything with `t_ctx` 
and also initialzing `m_ctx` as NULL.
 
### How to use this library
Just copy and past the file `rit_dyn_arr.h` in your project and include it. 
Nothing else is needed to be done.

### Known issues
I don't know of any, file a issue to inform me about such.

Feel free to open a new GitHub issue or pull request to suggest any changes!!
