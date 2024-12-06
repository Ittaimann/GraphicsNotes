# NEURAL LIGHT GRID
*ReRead in a month or two*
- Something about trying to get something useful out of ML in production lmao

## Can we use ML for pre-computed lighting? But first a primer on ML
- Functionally layers of matrix multiplication in steps to derive some output based on some input with care for history and training.

## The things that didn't work
### Idea 1: Train and grab
- Train on high res lighting (1 inch) and at runtime grab a normal and request a lighting value from the network 
	- Model was relatively simple... but the middle was 128x128 mat muls... so 16k multiplies at runtime.
- Results were good, but performance was impractical (multiple ms on ps4). Maybe create a lower res mapping by volume and in case we need it generate next frame. 
	- use spherical harmonics as the input, but seems to generate subpar results.
	- However perf was quite good and looked good. Very little popping from the frame delay and relatively looked nice
		- however, voxel-ification introduced discontinuties which created a lot of artifacts. Also lots of far froxels being triggered, introducing cascades introduced more artifacts and light leaking.
- Ok even with those problems its accceptable and can be engineered but... Bake time was 1-2 hours, training networks took 16 hours.
  
### Idea 2: Latent Spaces
- Use ML or modernize the entire lighting pipeline.
- Use of latent space which means there is some intrinsic meaning in a system for similar objects.
- Essentially train against old maps that have already got good enough lighting and let the ML determine the intrinsic outputs from those
- Use of deep mind paper [Neural Scene representation and rendering](). Given training data on a scene from multiple perspectives, the network could approximate the view from a novel viewpoint.
- Some of this is quiet dense, I don't understand it entirely. Essentially train on pre-existing data and then use that to approximate light bakes in a new position?
- ** A mess, didn't work at all **
	- First part of generating latent space seemed ok but decoder would be really bad, couldn't tell if the encoder was doing something wrong and the decoder can't deal with it?
- Eventually determined that decoder size wwas not large enough but by the time it was large enough it was taking 3ms on a ps4 gpu doing 4x4x4 brick, for visual stability it would need to decocde 128 (300ms). Needed way more training. Also didn't look good.
### Idea 3-n
- Adjusted values in the models and everything in betweenn, anytime it'd be promising it fall apart for either runtime cost or bake time.

## Pivot, What do we have that could be improved?
### Light Grids
- Take the 3d light grid that is sampled from and generate a radiance map based on that. Input is only 4 possible points interpolated. Results actually ended up pretty good!
- But the original system was linear, this is non-linear and can't be easly compposed, at its heart its still irradiance probes, but the sampling is ML based?
- SO sorta not used

## Reconstrucction
- TLDR use proper weighting functions, Per sample look to the lights in the area with their weighting functions and blend appropriately.
- Problem is that the space is global, so the weighting functions are not isolated and thus difficult. So generate the weighting function using ML? 
- Generated learned outputs is very very similar to generated baked ones?
- This weighting function is turned into a very small neural network at runtime it takes xyz pos and returns the weight at the given position. For training these they wrote a custom implmenetation as pytorch was 100x slower and gpu version was only 15% faster than custom cpu.
- Training took a while... but the meta trainer aka a hypernetwork seemed to work to immprove bake time.
- 4tb of training data took 18 hours  to build network. When lighting artists put down something the new raddiance is computed in a couple seconds
- Essentially have a bunch of radiance probes and they are relative to each other in a grid, weighting for the points in the grid is done via training? Look up is ok-ish? 
