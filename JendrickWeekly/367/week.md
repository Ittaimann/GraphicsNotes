# Topics
[gpu driven rendering in anki](https://www.youtube.com/watch?v=QNN3JOI551A)

## extra


[Frames in flight](https://erfan-ahmadi.github.io/blog/Nabla/fif)

- Essentially the stuff where you have a frame on the cpu and the gpu work in parallel. There is some stuff about getting the next image from the swap chain (look into vkAquireNextImageKHR, as the docs imply it is only necessary with a specific spawnChain create flag but the entire present section seems to be technically an extension).

[AA analysis](https://blog.frost.kiwi/analytical-anti-aliasing/)
Just an AA history, may be good for text based. Good little itinerary of techniques.

[Slang](https://shader-slang.com/blog/2024/11/20/theres-a-lot-going-on-with-slang/)
Updates to slang, just press about it being run by khronos now.

[whats next for webgpu](https://developer.chrome.com/blog/next-for-webgpu)

- Allow for cross thread intrinsics, texel buffers, Unified memory buffer mapping, bindless, multidraw indirect, 64 bit atomics 

[Cpu perf optimizations](https://gpuopen.com/learn/cpu-performance-guide/cpu-performance-guide-part2/)

- don't use array of pointers, just use a raw ass array as it will gurantee hardware pre-fetch and less memory read misses.