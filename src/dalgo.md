<!-- SPDX-License-Identifier: zlib-acknowledgement -->

* **Two arrays, find matching pairs between them**
Intuitive solution with array: O(n²), O(n)
Knowing uniqueness, can obtain linear solution with hashmap: O(n), O(2n)


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


* Array: search O(n)
  - Heap:
* Hashing: insert O(1)
  Hashing function different for strings and integers.
  Fixed array of buckets, where the index of an element is obtained by modulo'ing hash value.
  Each bucket a linked list of slots following separate chaining collision resolution.
  Each slot will store the key. 
  The key will also contain the value of the key prior to hashing to allow for collision resolution
  - Set: just keys
  - Hashmap
* Singly Linked List: search O(n)
  managing first and last pointers with node next pointer
  - Queue (priority queue?)
  - Stack
* Doubly Linked List: deletion O(1) 
  managing first and last pointers with node next and prev pointers
* Tree: search O(logn)
  Store standalone parent node pointer
  Child nodes stored in a doubly linked list
  Therefore, store managing first and last pointers with child node next and prev pointers 
  Complete tree has all 'h-1' levels are filled, and last level filled from left to right
  Balanced tree has each node's subtree's differing in height by no more than 1 (O(logn)) 
  Full/perfect tree has all nodes filled 
  - BST:
    Each node at most degree 2.
    Nodes of left subtree have values less than node, so definitionally require distinct values.
  - B-Tree: 
    Self balancing multiway search tree
    Nodes store a number of keys, i.e. values based on order of tree
    Order 'm' has each node with at most 'm' children, i.e. 'm' comparison points and 'm-1' keys
    If on insertion into leaf node violates order, than will create new parent node based on median comparison point.
    Used in databases to speed up disk access, e.g. with say order 500
    This is because of large number of children results in lower tree height, so less disk accesses
    So, use with huge number of items or items that are grouped together
  - Red-Black tree:
    Most popular search balanaced binary search tree implementation
    Has concept of external nodes, which are just left or right pointers that are null (so different to child node)
    Root node is black
    Each red node has black child nodes
    The black depth is same for each leaf node. It's the number of black nodes (including external node, excluding root node) 
  - AVL Tree:
    Another popular search balanaced binary search tree implementation
    Faster lookups than red-black as stricter rules around rebalancing, however results in slower insertions and removals
  - Splay Tree, 2-3-4 Tree
hierarchical data, faster searching is added boon
Ordered use self-balancing red-black-tree yielding logarithmic time
* Graph:
  Adjacency list has each vertex store a linked list of all the edges it connects to
  Therefore, better for sparse graphs than adjacency matrix

* Sorting:
Important to keep in mind we are executing on a physical machine and that
Big-Oh is a 'zero-cost abstraction' world.
For example, the extra overhead of introducing a hashmap (memory allocations/copies) 
will result in this being slower for small lists (also no dynamic memory allocations in ISR)
This is why C++ STL uses hybrid introsort
  - O(n²) preferable for small lists
insertion/bubble/selection
  - O(nlogn) divide-and-conquer  for medium
merge/quick/heap (qsort() is quick-sort)
  - O(n) for large
radix

* Traversing: (seems to be mainly graph or tree orientated?)
BFS, DFS, Djisktra


TODO: what is dynamic programming?

TODO: multithreading!!!

For embedded:
* buffering techniques: zero-copy buffer, circular/ring buffer
* bit-twiddling tricks: 
* Usage and problems with DMAs, DMAs with and without cache coherency
* persistent logs  
* Identify if it is HW or FW problem
* debugging techniques for various peripherals
* data structure dumps
* TLB with MMU
* state machines


TODO: understanding how to benchmark algorithm.
know about various cache misses
