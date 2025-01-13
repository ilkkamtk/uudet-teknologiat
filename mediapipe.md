# Local AI

- artificial intelligence models and systems that run directly on a local device (e.g., a smartphone, laptop, or
  embedded system)
- no constant connection to external servers or cloud platforms.
- leverages on-device resources to perform computations and make predictions.
- example frameworks: [Google AI Edge](https://ai.google.dev/edge), [ONNX](https://onnxruntime.ai/docs/tutorials/web/),
  or [Executorch](https://pytorch.org/executorch-overview).
- [Built-in AI in Chrome](https://developer.chrome.com/docs/ai/built-in-apis)

---

## Key Characteristics of Local AI

1. **On-Device Processing**
    - AI computations are performed locally, reducing dependence on external servers.

2. **Privacy-Focused**
    - Data remains on the device, ensuring better protection of sensitive information.
    - Reduces the risk of data breaches or unauthorized access during transmission to remote servers.

3. **Real-Time Performance**
    - Local AI minimizes latency since data does not need to travel over the network.
    - Ideal for applications requiring instantaneous responses, like augmented reality (AR), speech recognition, and
      real-time video analysis.

4. **Offline Functionality**
    - Enables the use of AI systems in areas with limited or no internet connectivity.
    - Applications like language translation, image recognition, or navigation can function seamlessly without external
      dependencies.

5. **Cost-Effective**
    - Reduces the need for continuous cloud server usage, making it more affordable for large-scale deployments.
    - Ideal for resource-constrained environments where cloud services may be impractical or expensive.

---

## Challenges of Local AI

1. **Resource Constraints:**
    - Limited processing power, memory, and storage on edge devices.
    - Requires efficient optimization techniques to deploy models.

2. **Model Compression and Optimization:**
    - AI models need to be compressed or pruned to fit device constraints without significant loss in accuracy.

3. **Development Complexity:**
    - Developers must account for hardware variability across devices.
    - Balancing accuracy with performance on constrained devices.

---

### **Applications of Local AI**

1. **Computer Vision:**
    - Real-time facial recognition, object detection, and hand tracking using frameworks like MediaPipe.

2. **Voice and Natural Language Processing:**
    - Speech recognition, language translation, and chatbots that operate offline.

3. **Healthcare:**
    - Wearable devices that monitor health metrics locally, such as heart rate or glucose levels.

4. **Augmented and Virtual Reality (AR/VR):**
    - Real-time interactions and overlays without cloud dependency.

5. **Smart IoT Devices:**
    - Devices like security cameras, smart thermostats, and industrial robots.

---

## Google MediaPipe Solutions

**[MediaPipe Solutions](https://ai.google.dev/edge/mediapipe/solutions/guide)** provides a suite of libraries and tools
for building cross-platform artificial intelligence (AI) and machine learning (ML) pipelines.

MediaPipe Suite includes:

- [MediaPipe Tasks](https://ai.google.dev/edge/mediapipe/solutions/tasks): Core programming interfaces for building ML
  solutions.
- MediaPipe Models: Pre-trained models for common tasks.
- [MediaPipe Model Maker](https://ai.google.dev/edge/mediapipe/solutions/model_maker): Tools for training custom models.
- [MediaPipe Studio](https://ai.google.dev/edge/mediapipe/solutions/studio): A visual tool for building and testing ML
  pipelines.

---

### Getting Started with MediaPipe in React. Gesture Detection Example

1. **Install MediaPipe:**
    - Use the [MediaPipe JavaScript API](
    - Add the MediaPipe library to your project using npm: `npm install @mediapipe/tasks-vision`
    - Load
      the [MediaPipe model](https://storage.googleapis.com/mediapipe-models/gesture_recognizer/gesture_recognizer/float16/latest/gesture_recognizer.task)
      for the desired task and save to `assets/models`.
2. Example Code:
    ```typescript
    import React, { useEffect, useRef, useState } from 'react';
    import { GestureRecognizer, FilesetResolver } from '@mediapipe/tasks-vision';
    
    const DetectGesture: React.FC = () => {
    const [gesture, setGesture] = useState<string>('No gesture detected'); // Current gesture
    const videoRef = useRef<HTMLVideoElement>(null); // Reference to the video element
    const timer = useRef<ReturnType<typeof setTimeout> | null>(null); // Timer for frame processing
    
    useEffect(() => {
      let gestureRecognizer: GestureRecognizer | null = null;

      // Initialize the GestureRecognizer
      const initializeGestureRecognizer = async () => {
        try {
         const filesetResolver = await FilesetResolver.forVisionTasks('/wasm');
         gestureRecognizer = await GestureRecognizer.createFromOptions(filesetResolver, {
           baseOptions: { modelAssetPath: '/models/gesture_recognizer.task', delegate: 'GPU' },
            runningMode: 'VIDEO',
            numHands: 1,
          });
        } catch (error) {
          console.error('Error initializing GestureRecognizer:', error);
        }
      };

     // Process video frames for gesture detection
     const processVideoFrames = async () => {
        if (videoRef.current && gestureRecognizer) {
          const nowInMs = Date.now();
          const results = gestureRecognizer.recognizeForVideo(videoRef.current, nowInMs);

          // Update gesture state
          if (results.gestures.length > 0) {
            const detectedGesture = results.gestures[0][0]?.categoryName || 'No gesture detected';
            setGesture(detectedGesture);
          }
        }
        timer.current = setTimeout(processVideoFrames, 100); // Process frames periodically
      };

      // Main initialization function
      const main = async () => {
        try {
          if (!videoRef.current) return;

          // Wait for the video element to be ready
          await new Promise<void>((resolve) => {
            if (videoRef.current.readyState >= 2) resolve();
            else videoRef.current.oncanplay = () => resolve();
          });

          await initializeGestureRecognizer();
          timer.current = setTimeout(processVideoFrames, 100); // Start processing frames
        } catch (error) {
          console.error('Error in main initialization:', error);
        }
      };

      main();

      // Cleanup on unmount
      return () => {
        if (gestureRecognizer) {
          gestureRecognizer.close();
          gestureRecognizer = null;
        }
        if (timer.current) {  
          clearTimeout(timer.current);
        }
      };
    }, []);
    
    return (
      <div style={{ textAlign: 'center', marginTop: '20px' }}>
        <h1>Gesture Detection</h1>
        <video
          ref={videoRef}
          autoPlay
          playsInline
          muted
          style={{ width: '80%', border: '1px solid black', marginBottom: '20px' }}
        />
        <p>
          <strong>Detected Gesture:</strong> {gesture}
        </p>
      </div>
      );
    };
    
    export default DetectGesture;

   ```

#### How It Works:

- Video Feed:
    - The `<video>` element captures input from the device's camera.
    - The videoRef connects the MediaPipe recognizer to this feed.
- Gesture Recognition:
    - MediaPipe processes the video feed and updates the detected gesture every 100ms.
- Display Gesture:
    - The gesture is displayed dynamically on the screen.

---

## Face Detection with face-api.js

**[face-api.js](https://github.com/justadudewhohacks/face-api.js)** is a JavaScript API for face detection, recognition,
and landmark detection in the browser and Node.js. It provides pre-trained models for face detection, face recognition,
and facial landmark detection.

### Usage Example in React

1. Install face-api.js:
    ```bash
    npm i face-api.js
    ```
2. Load Model (tiny_face_detector_model-shard1, tiny_face_detector_model-weights_manifest.json) [from the repository](https://github.com/justadudewhohacks/face-api.js/tree/master/weights) and save to `public/models`.
3. Example Code:
    ```typescript
    import React, { useEffect, useRef, useState } from 'react';

    import * as faceapi from 'face-api.js';
    
    const DetectFace: React.FC = () => {
    const videoRef = useRef<HTMLVideoElement>(null); // Reference to the video element
    const [detection, setDetection] = useState<faceapi.FaceDetection | null>(null); // Detected face
    
    useEffect(() => {
    let timer: ReturnType<typeof setTimeout> | null = null;
    
        // Load the face detection models
        const loadModels = async () => {
          try {
            await faceapi.nets.tinyFaceDetector.loadFromUri('/models');
            console.log('Models loaded');
          } catch (error) {
            console.error('Error loading models:', error);
          }
        };
    
        // Detect face from video frames
        const detectFace = async () => {
          if (videoRef.current) {
            const result = await faceapi.detectSingleFace(
              videoRef.current,
              new faceapi.TinyFaceDetectorOptions()
            );
    
            setDetection(result || null); // Update detection state
          }
    
          // Schedule the next detection
          timer = setTimeout(detectFace, 100);
        };
    
        // Initialize the video feed and start detection
        const startDetection = async () => {
          try {
            if (videoRef.current) {
              // Wait for the video element to be ready
              await new Promise<void>((resolve) => {
                if (videoRef.current!.readyState >= 2) resolve();
                else videoRef.current!.oncanplay = () => resolve();
              });
    
              detectFace(); // Start detecting faces
            }
          } catch (error) {
            console.error('Error initializing video feed:', error);
          }
        };
    
        loadModels().then(startDetection); // Load models and start detection
    
        // Cleanup on unmount
        return () => {
          if (timer) clearTimeout(timer);
        };
    
    }, []);
    
    return (
        <div style={{ textAlign: 'center', marginTop: '20px', position: 'relative' }}>
           <h1>Face Detection</h1>
            <video
                ref={videoRef}
                autoPlay
                muted
                playsInline
                style={{
                    width: '80%',
                    border: '1px solid black',
                }}
            />
            {detection && (
                <div
                    style={{
                        position: 'absolute',
                        top: detection.box.y,
                        left: detection.box.x,
                        width: detection.box.width,
                        height: detection.box.height,
                        border: '2px solid red',
                        pointerEvents: 'none',
                    }}
                ></div>
            )}
        </div>
        );
    };
    
    export default DetectFace;
    ``` 

#### How It Works:

- Video Feed:
    - The `<video>` element captures input from the device's camera.
    - The videoRef connects the face-api.js detector to this feed.
- Face Detection:
    - The detectSingleFace function uses the Tiny Face Detector model to detect a single face in the video frames.
- Bounding Box:
    - If a face is detected, the bounding box coordinates (detection.box) are used to display a red frame over the face.
- Detection Loop: 
    - The face detection runs every 100ms using setTimeout for continuous processing. One could use requestAnimationFrame for more accurate timing but that uses more resources.
- Model Loading:
    - The face-api.js models are loaded from the specified directory (/models) before starting the detection process.


