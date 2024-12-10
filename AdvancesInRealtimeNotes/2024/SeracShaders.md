# Serac shader authoring
- Frostbite has a variety of different games running in production with it. Various shading models, platforms, requirements. No one size fits all solution, so the solution must be flexible enough to handle all the possiblilties
- HLSL at scale
	- can't just use hlsl
	- no encapsulation no safety
	- centralised definitions that limit usage of things
	- no link between the cpu and the shader meaning its very iffy how things go from cpu to shader. Resources require string or register index.
- Background
	- lots of handwritten c++ code printing code.
	- endless if else and switches, enums and just tons of branching and weirdness.
	- hard to see whats going on, iterations were slow, very limited extensibility.
	- by and large no longer sustainable.
- Inspired by spire which is a new shader language, defining the full pipeline.
	- had concept of interfaces which let shader be swapped in and out as desired.
- other inspiration was TFX by bungie which wraps hlsl. 

## serac
- At the top level there are "components" where the code lives
	- HLSL that can then reference the components and references as if they were member variables. 
	- other component references
	- externs to allow data imports
- Techniques which define a complete shader 
- Everything gets put into a dag which takes the externs in the technique, each component they refence becomes nodes. Then the components string along until ultimately they write to the exports.
- The shaders are partitioned into domains determining pixel, or vertex seeded by the export, once that has been determined we can walk the graph.
- Functions being determined in hlsl makes it sorta annoying so use the dxc to generate the ast and extract info from that

### Component replacement

- Components can be defined with an interface to inherit from. this defines a specific functionality but a different implmentation. The implmenetation just asks for the override and then inline replaces the interface version with the specified version.
- Inputs to the graph are externs identified via the domain.
	- Access and writing from the cpu is done via generated code (cool!)
	- Lots of machinery allowing for tiny changes to allow for mass changes in the underlying machinery of the engine. Very interesting...

### node graph interaction
- Trivial attribute based system for interacting with shader graph system!
- hlsl parsed in @{ @} blocks to define where serac and the graph start and end.

## Conclusion
- limited use case shipped in bf2042
- liked by artists and engineers
- semantics are non-obvious, the interfaces are not classes. auto interpolation is useful but easy to add large amount of interpolators. code generated tends to be verbose and difficult to follow