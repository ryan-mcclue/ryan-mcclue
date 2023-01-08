<!-- SPDX-License-Identifier: zlib-acknowledgement -->

* Array:
  - Heap:
* Hashing:
Simplest hashing function `(x >> 4 + 12) & (size - 1)`
  - Set: just keys
  - Hashmap
* Singly Linked List: managing first and last pointers with node next pointer
  - Queue
  - Stack
* Doubly Linked List: managing first and last pointers with node next and prev pointers
* Tree:
  - BST, B-Tree, Red-Black tree, Splay Tree, AVL Tree?
hierarchical data, faster searching is added boon
Ordered use self-balancing red-black-tree yielding logarithmic time
* Graph:

* Sorting:
Important to keep in mind we are executing on a physical machine and that
Big-Oh is a 'zero-cost abstraction' world.
For example, the extra overhead of introducing a hashmap (memory allocations/copies) 
will result in this being slower for small lists (also no dynamic memory allocations in ISR)
This is why C++ STL uses hybrid introsort
  - O(n²) preferable for small lists
insertion/bubble sort
  - O(logn) divide-and-conquer  for medium
merge/quick
  - O(n) for large
radix

quick, merge, heap, bubble, insertion, selection, radix

## Matching Pairs (array, hashmap)
With array O(n), O(n²)
With hashmap O(2n), O(n)
A quadratically scaling solution is intuitive
However, as we know every match is unique, linearly scaling solution obtained with a hash map.


## MinHeap
// IMPORTANT(Ryan): For complete binary trees, better off storing it as an array
// In other tree structures, better of storing in nodes to account for holes, e.g. say root node only has left children
#define HEAP_CHILD0_INDEX(i) (2 * (i) + 1)
#define HEAP_CHILD1_INDEX(i) (2 * (i) + 2)
#define HEAP_PARENT_INDEX(i) math_floor_f32_to_u32((f32)(i - 1) / 2.0f)
#define HEAP_LAST_PARENT_INDEX(size) math_floor_f32_to_u32((f32)(size - 2) / 2.0f)

INTERNAL void
heap_min_sift_up(u32 index, u32 *array)
{

}

INTERNAL void
heap_min_sift_down(u32 index, u32 *array)
{

}


// remove root: swap with last, sift down (swap with minimum of child nodes)

// insert: append, walk up/sift up swapping with parent

// create: iteratively sift down on parent nodes starting from end
