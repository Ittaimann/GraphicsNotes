# Seamless Rendering on Mobile
## Background
- Cluster based rendering where occlusion is done on a per meshlet basis.
- Visbility buffer for rendering fine trinagles.
## Redesign expression format of mesh
- Like nanite, except they add a merge coefficient function to have higher controllability of assets and manually generated mesh LODs. For each cluster they store a bounding box and a normal cone for efficient occlusion and backface culling. Clusters are 128 tris (max 384 verts) so they can store indices with 8 bits. 
## The render pipeline
- Instances in an instance buffer, clusters in a cluster buffer and IDs of successfully cullsed clusters in a visible cluster buffer. 
- Culling starts with a instance level culling using HZB, frustum culling, occlusion culling, and a normal cone based backface culling. 
- Categorize objects in the visbuffer. 
- Tile the screen and each material has its list of tiles. Extract the materialID from the cluster info in the visbuf and compare it with the current shadingID
- For skinned meshes dynamically build bounding box info and culling based on clusters in the main bone space. Offline compute each clusters main bone, normal cone, and conservative bounding box in the main bone space. THe main bone is the one with the highest vertex weight in the cluster. During cluster culling read in the bone transform, transform the conservative bounding box into world space and do the cluster culling.
## GI for seamless rendering?
- Divide the scene into blocks and voxelize the blocks. These blocks and voxels are used for rapid raymarching. 
## Conclusion
- this seems to all work on mobile? There seems to be a few omissions from the original nanite ideas, but the deferred material rendering, lod generation, loding, and everything else is there. 