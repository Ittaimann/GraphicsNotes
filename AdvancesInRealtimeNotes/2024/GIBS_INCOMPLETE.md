# Shipping dynamic global illumination in frostbite 
## NOTE: PLEASE READ THE PREVIOUS PAPER BEFORE THIS ONE (the 2021 one)
- Frostbite has multipe diffuse GI systems availble ranging from preomcputed, parital precompute, and streaming irradiance volumes
- This has hit limits in large open worlds, dynamic environments with destruction, procedural and UGC.
## Dynamic Global illumination
- All inputs fully dynamic
- no authoring required
- high quality indirect diffuse lighting
- uses hardware ray tracing.
- original system presented at advances 2021.
	- ran on current gen, but no ships, average time is 6ms.
- Optimiazations have been done and now GIBS is being used in multiple shipped games include sports and open world action games.
## system overview
- Inputs:
	- gbuffer used to spawn surfels
	- tlas for raytracing
- Componenets
	- surfels persistent and spawn ray traces
	- probes clipmap amortized raytrace
- outputs
	- irradiance buffer for deferred lighting
	- probes for all other lighting

## improvements
- Instead of using hardware interpolation of spherical harmonic coefficients, they now use octahedral representation and software interpolation of the probes.
- Probe system
	- PAGE 16