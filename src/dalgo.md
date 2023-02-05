<!-- SPDX-License-Identifier: zlib-acknowledgement -->

IMPORTANT: In practice, you can solve efficiency by leaning on existing libraries/work.
More important to write robust code with tests.
Have knowledge of these rather than memorising implementations.
You can investigate specific implemenations on a per-problem basis

Euclidean geometry are a set of rules laid out by Euclid that follow for geometry on a flat surface

Big Oh is worse case.
Amortised is average over a series of operations, i.e. worse case average over a series of operations
It's an indication of how an algorithm scales, i.e. asymptotic limiting behaviour
It operates in a zero-cost abstraction world, e.g. memory allocations involved in small input number for linear is slower than logarithmic (why C++ STL uses hybrid introsort)
Furthermore, ignores constants, e.g. Fibonacci heaps have large constant in their amortised linear cost, so often not actually better insertions than binary heap



## Data Structures
* Array: search O(n)
  - Heap:
    The parent node is greater/less than all its child nodes
    Binary has 2 children, binomial has many
    Binary heap used to implement priority queues to allow for logarithmic insertion/deletion
    - Fibonacci Heap:
      Mergeable heap
      Consists of unioned min-binomial-heaps
      Called fibonacci as each tree of order 'n' will have at least fib('n + 2') nodes in it
* Hashing: insert O(1)
  Hashing function different for strings and integers.
  Fixed array of buckets, where the index of an element is obtained by modulo'ing hash value.
  Each bucket a linked list of slots following separate chaining collision resolution.
  Each slot will store the key. 
  The key will also contain the value of the key prior to hashing to allow for collision resolution
  - Set: just keys
      - Bloom Filter
        Probalistic set, in that it can only say if an element probably exists
        This is because the storage of different keys may overlap with one another 
        Greatly reduces storage space 
  - Hashmap
* Singly Linked List: search O(n)
  Constant time merging
  Managing first and last pointers with node next pointer
  - Queue
  - Stack
  - Skip List:
    Consists of a series of linked lists
    On creation, each list will move along the lower list and randomly select a node to copy over or skip
    Search path starts from the top of these lists in a greedy fashion, i.e. if cannot go further will move down a list
    Yields searches of linked lists in O(logn)
* Doubly Linked List: deletion O(1) 
  managing first and last pointers with node next and prev pointers
* Graph:
  Adjacency list has each vertex store a linked list of all the edges it connects to
  Therefore, better for sparse graphs than adjacency matrix
  Isomorphic means same number of vertices with same degrees
* Tree: search O(logn)
  Type of undirected unweighted connected acyclic graph. 
  Cannot have cycles
  Store standalone parent node pointer
  Child nodes stored in a doubly linked list
  Therefore, store managing first and last pointers with child node next and prev pointers 
  Complete tree has all 'h-1' levels are filled, and last level filled from left to right
  Balanced tree has each node's subtree's differing in height by no more than 1 (O(logn)) 
  Full/perfect tree has all nodes filled 
  For complete binary trees, better off storing it as an array
  In other tree structures, better of storing in nodes to account for holes, e.g. say root node only has left children
  Inverting is a notorious procedure, however demonstrates recursive solutions that arise with trees
  Specifically, inverting is swapping left and right subtrees from bottom up
  - Trie:
    Prefix matcher
    Each node will contain a flag that indicates if it's a prefix or whole
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
    A 2-3-4 Tree is a B-Tree of order 4
  - Red-Black tree:
    Most popular search balanaced binary search tree implementation
    Has concept of external nodes, which are just left or right pointers that are null (so different to child node)
    Root node is black
    Each red node has black child nodes
    The black depth is same for each leaf node. It's the number of black nodes (including external node, excluding root node) 
  - AVL Tree:
    Another popular search balanaced binary search tree implementation
    Balance threshold calculated by `left-subtree-height - right-subtree-height`.
    So, if left heavy, i.e. > 1, will do a right rotation.
    Rotations change 2 node positions, and then swap around children to adhere to BST invariants
    Faster lookups than red-black as maintains balance threshold of 1.
    However results in slower insertions and removals
  - KD Tree:
    Binary tree
    Breaks up space for 'k' dimensions
    'k' represents number of elements per node
    Each depth alternates what node element the BST invariant is applied to
    This has the effect of dividing into partial spaces.
    Therefore, will have to unwind to find actual nearest neighbour
    Useful for calculating nearest neighbour for static structures that don't update frequently
  - Quadtree/Octree 
    Perform spatial partitioning of 2D or 3D space respectively
    Multiway search tree, so number of direct children is 2ⁿ (where 'n' is number of dimensions) 
  - Splay Tree
    Inbalanced binary search tree, i.e. no extra operations performed on insertion and deletion
    On a search, the accessed node is rotated to the root node. 
    This makes for recently accessed nodes to retrieved fast.
  - Fenwick tree
    Also known as binary index tree, as uses bits of each index to determine how many elements to count up
    For example, indices with 0th bit 1 will add only 1 element, indices with 1st bit 1 will add 2 elements, etc.
    Considered a tree due to this indexing, although implemented as array
    Used for range checks

## Algorithms
greedy algorithms memory efficient however not optimal, so balancing act
greedy chooses next best option from what is available at the moment

dynamic programming is breaking a problem into subproblems and working way up
This involves storing the results of smaller problems and using them for results on bigger problems 

Informed search is when we have a way to estimate how far away are we from our goal, i.e. have domain knowledge

* Sorting:
  - O(n²) preferable for small lists or lists that are almost sorted
    - Insertion
      Used most often
      Working from left-to-right, develop an increasing 'sorted partition' as we move up an index 
      Each iteration, compare the element when elements to its left and swap if required
    - Selection
      Working from left-to-right as base element, find next element that is absolute minimum compared to this.
      Swap items. Repeat on next index. Less memory swaps than insertion
    - Bubble
      Working from left-to-right, compare two elements and swap if out-of-order. 
      Do so until reach end. Repeat on next index.
  - O(nlogn) divide-and-conquer (typically recursive) for medium
    - Merge
      Out-place
      Divide into sub-arrays until 1 element sized.
      Sort each sub-arrays and combine.
      Logarithmic as dividing into sub-arrays creates a binary tree structure 
    - Quick (qsort() is quick-sort)
      In-place meaning sorted items occupy same storage as original ones
      Involves pivot point, i.e. a point 
      When sorting, pivot point at end. 
      Moving from left, find element larger than pivot.
      Moving from right, find element smaller than pivot.
      Swap them. When elements overlap, place pivot back where it started.
      Can yield quadratic time if pivot point chosen poorly.
      Pivot point normally chosen via median-of-3, i.e. sort 1st/middle/last element 
    - Heap 
      First create max-heap and extract max.
      Then adjust existing max-heap, which as almost ordered, is logarithmic time.
      Repeat
  - O(n) for large
    - Radix
      Sort on element radix/base, e.g. ones unit, then tens unit etc.
      On each pass, copy elements into buckets and then copy back out
      Cost of memory operations reason for applicibility

* Search/traversing:
for each algorithm, does it work on DAG?

BFS, DFS (uninformed greedy memory efficient), Djisktra (BFS with weights), A* (Djikstra with heuristic), Knuth-Morris, Prims, Floyd-Warshall
Bellman-Ford (non-greedy), topological sort (for things with dependencies), Myers-Diff (dynamic programming),
Disjoint-Set-Union-Find (minimum spanning trees), 
Each A* node has local and global goals, i.e. how close to end result

(simple hash, simple checksum, Huffman encoding, greedy, non-greedy, dynamic programming, backtracing?)

TODO: multithreading!!!

For embedded:
* buffering techniques: zero-copy buffer (direct to peripheral buffer), circular/ring buffer, bipartite buffer
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
