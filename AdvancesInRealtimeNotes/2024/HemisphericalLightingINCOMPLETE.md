# Hemispherical Lighting insights
- Backed lighting split between simple objects that use lightmaps, objectsthat are more difficul use local light probes, while dynamic objects resample a local volume from a grid of probes
## Spherical Harmonics and least squares encoding
- Say we want to approximate some signal for all directions on a sphere or hemisphere. We define a function that will give us some value for some direction and compute a coefficient vector that gives weights for those functions. To reconstruct the signal we just sum the weighted basis functions for the direction we're querying.

Ok. I have no clue whats happening in this talk. Its way more math heavy than I can comprehend at all. This talk is unfortunately math heavy,i'd recommend returning to it at a later date with a better vocab. Once you have a grip on the math of spherical harmonics return to it.