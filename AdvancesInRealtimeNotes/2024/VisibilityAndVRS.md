# Variable Rate Shading with visibility buffer rendering

- Key idea, we don't see pixels, rather we see gradients. With variable rate shading strong edges (aka intense gradients) can let us reduce the sample rate within the geometry edge
- The goal of this talk is to use a visibility buffer to seperate geo and shading rate to sample whatever pattern we like, then reconstruct it. Essentially use the vis buffer to say which samples we want to process and intuit the rest
- page 12 lots of references (good read through)

## quad utilization
- quick review on quads. Idea that when we raster a triangle we are rastering at 4 pixels at at a time. If the pixel is outside the tri it becomes a helper lane and is discarded
- With VRS the 2x2 quad can be a larger size and we need to invoke more pixels at once. This section seems awkwardly done? Feels like the video would've explained this better?

## Vbuffer Details
- a quick review
	- forward rendering: caluclate light at draw
	- deferred write lighting in second pass
	- Visibilty buffer splits it into 
		- triangle ID
		- gBuffer write (now needs to fetch interpolators and evaulate material)
		- evaluate lighting
- with single pixel tries with vbuffer there is a 1x cost for rendering, but its a lot of extra overhead. vs Forward where the entire pass is 4x cost and deffered where material is 4x and lighting is 1x

 ### Vbuffer rendering
- how do we get primitive ID, fetch interpolators, calculate derivatives for mips, and (for this one) split the pixels sort them and put the in a final image with vrs

## Primitive ID
- how do we get it?
	- SV_PrimitiveID or gl_PrimitiveId but not supported globally and has perf penalties
	- NonIndexed draw call where we render 3 verts and use a color for the tri, downside no vertex reuse and requires 3 unique vert invokes per tri
	- Leading vertex, where we use a noninterpolate or flat shading where the interpolator will use one vertex for the color.
	- Mesh shaders, which has peak perf but not supported globally.
	- software rasterizer which is the fastest option on some platforms but requires 64bit atomics and generally requires a fallback for large tris
	- geometry shader could work but is inconsistent.
- Based on Seb altonen testing (that he obviously posted to twitter) it seems like leading vert is a 1x on all platforms (that we care for), with all others being slightly less so.
- But how do we get the proper leading vert? If the verts a laid out in a way where the first vert of ever tri is equivalent to the tris index we can just get the vertexID and call it. But it requires some work
	1. count the number of tris per vertex. First pass is a greedy loop where each trinagle simply chooses the best vertex, where the best one is the one with the lowest valence, aka onces with the least possibility of being used by other tris. If nothing is availble wait until the next pass.
	2. Rotate tris. If you have a triangle with no free verts to be the provoking index, then take one of the neighbors and see if they have a free vert. Do a depth first serach around the tri. We then can rotate our other triangles provoking vert to allow us to use one. We just need to keep rotating from our neighbors until we eventually find a free vert and that will be the last rotation.
- failure case, triangle with multiple 1 valence verts, which may result in tris that can't be rotated.
- Depending on the size of the cluster you can get away with only 64 indicies and only need 6bits per index, aka 5.5 bytes per triangle 
- if you want to support multiple paths compress the mesh to be mesh shader friendly and at load re-org for leading vert or non-indexed???

## Mesh Pool
- in order to make this work, you want to put all your mesh data in one big buffer. You can't index into a vert if all the meshes are tiny buffers that are isolated.
- The issue is that vertex animations can be quiet rough, the way around that is by doing them in animations via compute before hand. If you do compute skinning this already sorta is the way?

## Derivatives
- the 2x2 pixel quads exist for derivative sampling, so now you have to do it manually. you have to get the barycentric coordinates relative to x,y screen space.
- annoying but you can't use sample, you must you sample_grad

## Reconstructions (the part i don't get)
- When you render you can determine the density to render at in the grid. 2x diagonal vrs, 4x grid, 25% noise or the 1x reference (aka the normal way)
- The trick is accurate edge detection, using instanceID, depth, and geo normals you can get goo edge detector.
- for each pixel look in four surrounding regions, and choose one pixel from the region, we don't want to cross any edges so you need to use a search algo to find the best neighbor.
- you can actually do this via hardcoded search paths, and have it be a binary expression.
- to be a valid path to the neighbor it must:
	- not have any valid pixels on the way.
	- not have any horizontal borders aka we have horizontal edges are legal? I think he messed up here and should've said that edges are NOT legal 
	- not hit any vertical borders on the path aka vertival edges are legal? I think he messed up here and should've said that edges are NOT legal 
- after evaluating the giant boolian expression for all the search locations, we have a result between -4 and 4 in each dimension and a value for no hit. we can pack that into a single uint32
- We can then interpolate using the best 4 pixels we hit not crossing borders. A handful of pixels will not find a valid neighbor to interpolate to and the taa must handle those.

## perf of theory
- big tris: vbuffer is a slight loss
- tiny tris vbuffer is a massive winner.
- we pay functinoally 1.52x pixel call for material and 1.06x for lighting 
- Tiny tris with vrs 2x2 is even better at 3.72 ms and the per pixel cost is .42 for material and .32 for lighting

## conclusion
- vbuffer will be win eventually... maybe

## OIT?
- uh somethiing about chooseing good samples and interpolating, I don't really get it to be honest.