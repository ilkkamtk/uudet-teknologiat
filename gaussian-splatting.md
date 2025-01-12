# Gaussian Splatting

[Example](https://projects.markkellogg.org/threejs/gaussian_splats_3d_demos/truck.html?mode=0)

Gaussian splatting is a novel technique for rendering 3D scenes using continuous 3D Gaussians instead of traditional mesh-based geometry. This method is particularly useful for rendering point clouds, volumetric data, and other complex 3D structures.

Key idea is to represent 3D objects as a set of continuous Gaussian functions, which are then rendered using a ray-marching algorithm. This approach allows for smooth and accurate rendering of complex 3D structures, with minimal artifacts and high performance.

## Gaussian splatting vs NeRF

Gaussian splatting is similar to Neural Radiance Fields (NeRF) in that both techniques use continuous functions to represent 3D geometry. However, Gaussian splatting is more focused on rendering point clouds and volumetric data, while NeRF is designed for general 3D scene rendering.

## Advantages of Gaussian splatting

- **Smooth rendering**: Gaussian splatting produces smooth and continuous surfaces, which can be useful for visualizing complex 3D structures.
- **High performance**: By using continuous functions instead of mesh-based geometry, Gaussian splatting can achieve high rendering speeds.
- **Minimal artifacts**: Gaussian splatting minimizes rendering artifacts such as aliasing and discretization errors.
- **Flexible representation**: Gaussian splatting can represent a wide range of 3D structures, including point clouds, volumetric data, and implicit surfaces.
- **Scalability**: Gaussian splatting can scale to large datasets and complex scenes without sacrificing performance.

## Disadvantages of Gaussian splatting

- **Limited support**: Gaussian splatting is a relatively new technique and may not be supported by all 3D rendering engines and frameworks.
- **Performance trade-offs**: While Gaussian splatting can achieve high rendering speeds, it may not be as efficient as traditional mesh-based rendering for certain types of scenes.
- **Learning curve**: Gaussian splatting may require a steep learning curve for developers who are not familiar with continuous functions and ray-marching algorithms.

## Getting started with Gaussian splatting

To get started with Gaussian splatting, you can explore open-source implementations and research papers on the topic. There are several libraries and frameworks that support Gaussian splatting, including PyTorch, TensorFlow, and OpenGL.
