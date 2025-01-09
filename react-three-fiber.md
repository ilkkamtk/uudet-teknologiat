# React Three Fiber

React Three Fiber (R3F) is a React renderer for Three.js. It allows you to build Three.js scenes using React’s declarative style, which can make your 3D projects more intuitive and easier to maintain. With R3F, you write JSX to create 3D objects, manage their state, and interact with React's lifecycle and hooks.

---

### Why Use R3F?
1. **Declarative Syntax**: Define 3D objects in JSX instead of imperative Three.js code.
2. **Component Reusability**: Modularize your Three.js code into React components.
3. **React Ecosystem**: Leverage React features like state, props, and context in your 3D scenes.
4. **Reduced Boilerplate**: Automatically handles scene, camera, renderer, and animation loop.

---

### Getting Started
1. **Create React Project**
    - Start a new React project using Vite.

2. **Setup**
    - Install React Three Fiber via npm or yarn:
    ```bash
    npm install @react-three/fiber three
    ```

3. **Basic R3F App**
   - Here’s a simple example to render a rotating cube:

    ```tsx
    import { Canvas, useFrame } from '@react-three/fiber';
    
    function RotatingCube() {
      const ref = React.useRef();
    
      useFrame(() => {
        if (ref.current) {
          ref.current.rotation.x += 0.01;
          ref.current.rotation.y += 0.01;
        }
      });
    
      return (
        <mesh ref={ref}>
          <boxGeometry args={[1, 1, 1]} />
          <meshStandardMaterial color="orange" />
        </mesh>
      );
    }
    
    export default function App() {
      return (
        <Canvas>
          <ambientLight />
          <pointLight position={[10, 10, 10]} />
          <RotatingCube />
        </Canvas>
      );
    }
    ```

   - **Key Points:**
     - **`Canvas`**: R3F’s wrapper for the WebGL renderer.
     - **Geometry & Material**: Defined as JSX components (`<boxGeometry>`, `<meshStandardMaterial>`).
     - **Hooks**: Use `useFrame` for animations and updates.

---

### Comparison: Three.js vs R3F

| Feature                  | Three.js (Imperative)                           | R3F (Declarative)                                   |
|--------------------------|------------------------------------------------|----------------------------------------------------|
| Scene setup              | Manual scene, camera, and renderer setup       | `<Canvas>` handles setup automatically             |
| Object creation          | `new THREE.BoxGeometry()`                      | `<boxGeometry args={[width, height, depth]} />`    |
| Object hierarchy         | Manual `add()` to scene/parent objects         | Nested JSX                                        |
| Animation loop           | Manual `requestAnimationFrame` loop            | `useFrame` hook                                    |
| React integration        | Difficult to use React state/props directly    | Fully integrated with React                        |

---

### React Hooks in R3F
R3F integrates with React's hooks for interactivity and animations:
- **`useFrame`**: Runs on every animation frame (ideal for updates like rotations).
- **`useRef`**: Access and manipulate objects (e.g., meshes, cameras).
- **`useThree`**: Access Three.js internals like `scene`, `camera`, `gl`.

Example: Animate a camera position:
```tsx
import { useThree, useFrame } from '@react-three/fiber';

function CameraAnimation() {
  const { camera } = useThree();

  useFrame(() => {
    camera.position.z = Math.sin(Date.now() * 0.001) * 5 + 10;
  });

  return null;
}
```

---

### Extending with Drei
Install [Drei](https://drei.docs.pmnd.rs/getting-started/introduction), a helper library for common R3F components:
```bash
npm install @react-three/drei
```

Example: Add an orbit control and environment lighting:
```tsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Environment } from '@react-three/drei';

export default function App() {
  return (
    <Canvas>
      <ambientLight intensity={0.5} />
      <OrbitControls />
      <Environment preset="sunset" />
      <mesh>
        <boxGeometry args={[1, 1, 1]} />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
}
```

---

### Loading 3D Models

- [Loading GLTF models](https://r3f.docs.pmnd.rs/tutorials/loading-models#loading-gltf-models)
- [Loading GLTF models as JSX components](https://r3f.docs.pmnd.rs/tutorials/loading-models#loading-gltf-models-as-jsx-components)

---

### Exercise
Create a simple 3D UI using React Three Fiber:

1. **Objective**: Build a Sci-Fi control panel with interactive buttons.
    - Or any other thing that has something you can animate and/or interact with.
2. **Features**:
   - Use `useFrame` to animate antenna (or smth else) on the panel.
   - Implement a click event to toggle button states.
   - Add lighting and shadows for a realistic look.
3. You can create the control panel design using simple geometries and materials or import 3D models.
