# Achieveing scalable performances for large scale game components with CBTS

- Dealing with large scale game components like terrain, ocean, space/planets
- How dow we render these? These problems have two parts
	- Data generation, aka how we make textures, instances, sprites etc...
	- Triangulation/rendering category aka how do we render this
- This paper only handles the second one.
- CBT is concurrent binary tree, this talk is v2 (v1 presented 3 years earlier)
- At full res the demo they have is an exabyte of data, which would prevent LOD systems like nanite, which require full res mesh as input (does it? the whole point is that its clustered and not actually globally full res right?)

## what is a CBT?
- a concurrent binary tree is a full binary tree (each node has two children except for leaves) with two main parts 
	- A bit field at the bottom level of the tree
	- a sum reduction tree of this bitfield at the upper levels
- This mean the root gives the total number of green bits (what those are I don't knnow)
	- The are allocated spaces
- We track allocation and used spaces from the top of the tree
- We can go down the tree into the bitfields that may be set to unallocated and then allocate to them and update the sum reduction. Same with deallocation
- We then allocate triangles and use the bitmask at the bottom as a heapID, and store the neighboring tris as their heapids. This is queit difficult to describe in words, i'd recommend re-reading the talk if you want to do anything with this.
- Having the neigbhors we now have the ability when bisecting a tri to propogate the bisection as to not have cracks in the mesh.

## Parallel update routine for CBT 
- code is open source, and availble if we want it... do we? lmao?
- We want to dump the cbt into group shared memory, but the limitation is 32kb in dx12
- Each level is defined as
	- The number of nodes within the level
	- the range of values each node can represent
	- the minimal size of each node to represent that range
	- The final size used to represent each node
- Each level has some statement about memory locations but I don't quiet understand how or why its behaving the way it is? seems somewhat strange tbh?
	- top level is atomic friend subtree of uint32 in group memory
	- then a thread safe subtrees with min size aligned on powers of two in group shared.
	- virtual trees that don't exist
	- Raw bitfield buffer of uint64_t that is used to intuit the virtual trees
- How does that end up in implmenetaion
	- two structured buffers one stores uint32_t of the top level and threadsafe
	- a second one stores uint64_t which is the bitfield
	- an allocations buffer in uint32_t
- The way the memory manager is used evey frame is:
	- book the worst case memory requirement
	- allocate a bunch of bits that will depend on the subdivion we are processing
	- return the memory manager memory space that we didn't use.
- with this we can decode the bits we've allocated.
	- Load the CBT into shared memory for our searches
	- Descend the tree
	- atomically set bits to 1 for allocations we've done and 0 for those we've deallocated.
	- Do a sum reduction logging the amount of bits in the lower level trees.

### Memory pool
- Per triangle we need the heapID buffer, neighbor buffers, camera position, the position of the parent triengle for split/merge decisions, and some temp data.
- At each update loop, a bisector can be in one of 5 states, unchanged, single split, double split (two cases), or triple split. We encode the state and store via atomics. Each bisection creats some number of allocations so we need to store that, as well as flags and propogations. 

## Example routine 
- page 90
1. Single thread dispatch that prepares allocations for the frame, resets bisector queues for indirect dispatches
2. classify where each thread will run for one active bisector, classifying them in one of 3 states, keep, merge, or split. Critieria for that changes depending on what you want, but for this demo the you ndotv test and flag if failed. if it intersects one or multiple frustums it flags it for merge. Then compare the current size and parent size to determine if it needs to be merged back to parent (smaller triangle size) or split (larger tri size)
3. Split pass which they don't explain for some reason?
4. Decode runs the bisectors that have been flagged for subdivision. based on the subdivision patter of the current bisector and neighbors we modify the heap id and adjust neighbor pointers
5. Propogate bisect, which will adjust and problematic interfaces and change the neighbors when required. if the neighbor was split we have to adjust the neighbors.
6. Prepare simplify which runs on bisectors marked for merge/simplify. we take the smallest heap id and free the others.
7. propage simplify which adjustst the neighbores data for those that have been simplified.
8. Reduction we modified the bit field in the previous step and now need to re-run the sum reductions
9. Evaluate LEB by going from heap id to triangle multiplications into the appropriate view space... I think thats what this is doing?
10. apply some deformation to make use of the enhanced amount of geo.

## conclusion uh... ya I didn't get it. Smart bitmasking of verts to do tesselation but it seems like its not for normal games?



