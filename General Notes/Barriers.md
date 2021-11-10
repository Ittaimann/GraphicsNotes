# Barriers

Most this info is from a work colleague so it's unsourced sadly but worth it

In dx12 and vulkan it's possible to force a barrier for gpu execution. Generally there are 3 times you want to do this.

- to flush cache on a shader execution. When you are writing to something. Shaders when complete don't necessarily immediately flush whatever they wrote to global memory, so putting a barrier forces a flush to occur.
- to force a wait for another resource. Essentially if we need a resource to be done while we start executing we can place a barrier to wait for that to be completed.
- to change formats of a resource. In gcn we have different formats in hardware support that are supposed to be written to and read from in different ways depending on compression.
