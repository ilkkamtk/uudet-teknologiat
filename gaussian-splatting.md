# Neural Radiance Fields and Gaussian Splatting

## Neural Radiance Fields (NeRF)

[Example: Explore NeRF-based scene reconstruction](https://nerf-w.github.io/). This example demonstrates NeRF's ability to render highly detailed 3D scenes from a sparse set of input images.

Neural Radiance Fields (NeRF) is a state-of-the-art technique for reconstructing and rendering 3D scenes using neural networks. Unlike traditional 3D modeling methods, which rely on explicit geometry (e.g., meshes or point clouds), NeRF represents scenes as continuous volumetric functions encoded in a neural network.

The key idea is to model the scene as a 5D function:
- Input: 3D spatial coordinates \((x, y, z)\) and a 2D viewing direction \((\theta, \phi)\).
- Output: The density (opacity) and emitted radiance (color) at that point.

NeRF uses **volume rendering techniques** to synthesize images by integrating these values along rays cast through the scene. This enables highly realistic image synthesis from novel viewpoints.

## Advantages of NeRF

- **High-quality 3D reconstructions**: NeRF can capture fine details and complex lighting effects, making it ideal for photorealistic scene modeling.
- **Few input images required**: Unlike traditional photogrammetry, NeRF can reconstruct scenes from a sparse set of images taken from different angles.
- **Continuous representation**: NeRF models scenes as a smooth, continuous function, avoiding the discretization issues of meshes or voxel grids.
- **Rich lighting effects**: Captures subtle visual phenomena like soft shadows, specular highlights, and translucency.
- **Versatile applications**: Used in fields like virtual reality, film production, and real-world scene digitization.

## Disadvantages of NeRF

- **Computational cost**: Training NeRF models requires significant computational resources, including GPUs or TPUs, and can take hours or even days for complex scenes.
- **Real-time rendering limitations**: While recent advancements have sped up NeRF rendering, interactive real-time performance remains a challenge compared to traditional rendering methods.
- **View-dependent artifacts**: Reconstructed scenes may exhibit artifacts when viewed from extreme angles due to incomplete or biased training data.
- **Data dependency**: The quality of the reconstruction depends heavily on the quality and coverage of the input images.

---

Let me know if you’d like a deeper dive into the math behind NeRF, or if you need this formatted for slides or presentations!



## Gaussian Splatting

[Example: Explore Gaussian splatting applied to a 3D truck model](https://projects.markkellogg.org/threejs/gaussian_splats_3d_demos/truck.html?mode=0).

Gaussian splatting is a technique for rendering 3D scenes using continuous 3D Gaussians instead of traditional mesh-based geometry.

The key idea is to represent 3D objects as a set of points, each blurred using a Gaussian function. These "splats" are then rendered using ray-marching algorithms to produce smooth and continuous surfaces.

Gaussian splatting shares similarities with Neural Radiance Fields (NeRF) in using continuous functions to represent 3D geometry. However, while NeRF is designed for general 3D scene reconstruction, Gaussian splatting focuses on point clouds and volumetric data, offering significant advantages in simplicity and rendering speed.

## Advantages of Gaussian Splatting

- **Smooth rendering**: Produces seamless, continuous surfaces, ideal for applications like medical imaging or architectural visualization.
- **High performance**: Leverages continuous functions to achieve high rendering speeds, especially for point-based datasets.
- **Minimal artifacts**: Reduces aliasing and discretization errors common in traditional rendering pipelines.
- **Flexible representation**: Can handle diverse data types, including point clouds, volumetric data, and implicit surfaces.
- **Scalability**: Efficiently scales to handle large datasets or complex scenes without major performance degradation.

## Disadvantages of Gaussian Splatting

- **Limited support**: As a newer technique, it may not yet be compatible with all 3D rendering engines and frameworks.
- **Performance trade-offs**: While fast for many scenarios, Gaussian splatting can be less efficient than traditional mesh-based rendering for highly structured objects, like architectural CAD models.
- **Learning curve**: Developers unfamiliar with ray-marching algorithms and continuous functions may find Gaussian splatting challenging to implement initially.

---

## Comparison

| **Aspect**             | **NeRF**                                         | **Gaussian Splatting**                             |
|-------------------------|-------------------------------------------------|---------------------------------------------------|
| Representation          | Continuous volumetric function (neural network) | Point-based representation using Gaussian splats  |
| Input Data              | Sparse 2D images                                | Point clouds or volumetric data                   |
| Rendering Method        | Volume rendering (neural integration)           | Ray-marching with Gaussian projections            |
| Strengths               | Photorealism, complex lighting                  | Speed, simplicity, and scalability                |
| Computational Efficiency| High training cost, slower rendering            | Faster rendering, lower training cost             |

---

## Resources

### NeRF
- https://docs.nerf.studio/
- https://youtu.be/YX5AoaWrowY?si=KN0pW8b8m3Q1xMFe

### Gaussian Splatting
- https://www.youtube.com/watch?v=Xn4h0vJ-wYQ
- https://www.youtube.com/watch?v=ERuRMOVO58Q
- https://www.jawset.com/ - postshot, generate splats from images or videos (Windows)
   - Installed in Media Lab, Unreal computer 
- [Supersplat](https://playcanvas.com/supersplat/editor/) - Gaussian Splat editor
- https://github.com/mkkellogg/GaussianSplats3D
- https://github.com/vincent-lecrubier-skydio/react-three-fiber-gaussian-splat
- [Splat embedded with Babylon.js](https://users.metropolia.fi/~ilkkamtk/babylon/)
