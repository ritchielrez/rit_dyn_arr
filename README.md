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

**Example usage:**
```cpp
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>

#include "rit_dyn_arr.h"

void *libc_malloc(void *t_ctx, size_t t_size_in_bytes) {
 (void)t_ctx;
 return malloc(t_size_in_bytes);
}

void libc_free(void* t_ctx, void *t_ptr) {
  (void)t_ctx;
  free(t_ptr);
}

void *libc_realloc(void *t_ctx, void *t_old_ptr, size_t t_old_size_in_bytes, size_t t_new_size_in_bytes) {
  (void)t_ctx;
  (void)t_old_size_in_bytes;
  return realloc(t_old_ptr, t_new_size_in_bytes);
}

int main() {
  rit_dyn_arr_allocator ctx_allocator = {libc_malloc,
                                         libc_free,
                                         libc_realloc, NULL};

  rit_dyn_arr(int, arr, 10, &ctx_allocator);

  for (int i = 0; i < 10; ++i) {
    rit_dyn_arr_set(arr, i, i + 1);
    printf("%d\n", rit_dyn_arr_at(arr, i));
  }

  rit_dyn_arr_push_back(&arr, 11, &ctx_allocator);
  rit_dyn_arr_pop_back(&arr);
  rit_dyn_arr_insert(&arr, 2, 69, &ctx_allocator);
  rit_dyn_arr_erase(&arr, 1);
  rit_dyn_arr_resize(&arr, 3, 0, &ctx_allocator);

  for (int *it = rit_dyn_arr_begin(arr); it < rit_dyn_arr_end(arr); it++) {
    printf("%d\n", *it);
  }
  printf("1st element: %d, last element: %d\n", rit_dyn_arr_front(arr),
         rit_dyn_arr_back(arr));

  rit_dyn_arr_clear(arr);
  printf("arr.empty() = %d\n", rit_dyn_arr_empty(arr));

  printf("size: %zu, capacity: %zu\n", rit_dyn_arr_size(arr),
         rit_dyn_arr_capacity(arr));

  rit_dyn_arr(int, arr2, 10, &ctx_allocator);
  for (int *it = rit_dyn_arr_begin(arr2); it < rit_dyn_arr_end(arr2); it++) {
    *it = 0;
  }

  for (int *it = rit_dyn_arr_begin(arr); it < rit_dyn_arr_end(arr); it++) {
    printf("%d ", *it);
  }

  rit_dyn_arr_resize(&arr, 30, 1, &ctx_allocator);
  printf("\n[Before swap]\n1st array: \n");
  for (int *it = rit_dyn_arr_begin(arr); it < rit_dyn_arr_end(arr); it++) {
    printf("%d ", *it);
  }

  printf("\n2nd array: \n");
  for (int *it = rit_dyn_arr_begin(arr2); it < rit_dyn_arr_end(arr2); it++) {
    printf("%d ", *it);
  }

  rit_dyn_arr_swap(&arr, &arr2);
  printf("\n[After swap]\n1st array: \n");
  rit_dyn_arr_for_each(it, arr) {
    printf("%d ", *it);
  }

  printf("\n2nd array: \n");
  rit_dyn_arr_for_each(it, arr2) {
    printf("%d ", *it);
  }

  rit_dyn_arr(int, arr3, 0, &ctx_allocator);
  rit_dyn_arr_copy(&arr, &arr3, &ctx_allocator);
  printf("\n3rd array(copy of the 1st one): \n");
  rit_dyn_arr_for_each(it, arr3) {
    printf("%d ", *it);
  }

  rit_dyn_arr_free(arr, &ctx_allocator);
  rit_dyn_arr_free(arr2, &ctx_allocator);
  rit_dyn_arr_free(arr3, &ctx_allocator);
}
```

### Known issues
I don't know of any, file a issue to inform me about such.

Feel free to open a new GitHub issue or pull request to suggest any changes!!
