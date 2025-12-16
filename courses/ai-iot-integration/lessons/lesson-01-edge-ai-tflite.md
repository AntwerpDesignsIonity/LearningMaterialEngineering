# Lesson 1: Edge AI with TensorFlow Lite

## Introduction
Learn to deploy machine learning models on resource-constrained IoT devices using TensorFlow Lite for Microcontrollers.

## What is TensorFlow Lite?
TensorFlow Lite (TFLite) is an optimized framework for running machine learning models on mobile and embedded devices. TensorFlow Lite for Microcontrollers extends this to run on microcontrollers with just a few kilobytes of memory.

## Key Concepts

### 1. Model Optimization
Techniques to reduce model size and computational requirements:
- **Quantization**: Reduce precision from 32-bit float to 8-bit integer
- **Pruning**: Remove unnecessary connections
- **Knowledge Distillation**: Train smaller models to mimic larger ones

### 2. TinyML Constraints
- Limited RAM (typically 256KB or less)
- Limited Flash storage (1-2MB)
- Low processing power
- Power efficiency critical

### 3. Use Cases
- Keyword spotting (wake word detection)
- Gesture recognition
- Anomaly detection
- Predictive maintenance
- Image classification

## TensorFlow Lite Architecture

### Training Phase (Cloud/Desktop)
```python
import tensorflow as tf

# Create and train model
model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation='relu', input_shape=(10,)),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dense(3, activation='softmax')
])

model.compile(optimizer='adam', loss='categorical_crossentropy')
model.fit(train_data, train_labels, epochs=10)

# Convert to TensorFlow Lite
converter = tf.lite.TFLiteConverter.from_keras_model(model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_model = converter.convert()

# Save the model
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
```

### Inference Phase (Microcontroller)
```cpp
#include "tensorflow/lite/micro/micro_interpreter.h"
#include "tensorflow/lite/micro/micro_mutable_op_resolver.h"
#include "model.h"  // Your converted model

// Set up TFLite
constexpr int kTensorArenaSize = 10 * 1024;
uint8_t tensor_arena[kTensorArenaSize];

tflite::MicroMutableOpResolver<5> micro_op_resolver;
micro_op_resolver.AddFullyConnected();

tflite::MicroInterpreter interpreter(
    model, micro_op_resolver, 
    tensor_arena, kTensorArenaSize);

interpreter.AllocateTensors();

// Get input/output tensors
TfLiteTensor* input = interpreter.input(0);
TfLiteTensor* output = interpreter.output(0);

// Run inference
interpreter.Invoke();
```

## Quantization Deep Dive

### Post-Training Quantization
```python
converter = tf.lite.TFLiteConverter.from_keras_model(model)

# Dynamic range quantization (weights only)
converter.optimizations = [tf.lite.Optimize.DEFAULT]

# Full integer quantization (weights and activations)
def representative_dataset():
    for data in sample_dataset:
        yield [data]

converter.representative_dataset = representative_dataset
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.int8
converter.inference_output_type = tf.int8

tflite_model = converter.convert()
```

### Model Size Comparison
- Original: 500KB (float32)
- Dynamic quantization: 125KB (int8 weights, float32 activations)
- Full quantization: 125KB (int8 weights and activations)
- Additional optimizations: 50-80KB

## Example: Image Classification on ESP32

### Training the Model
```python
import tensorflow as tf
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# Load and preprocess data
train_datagen = ImageDataGenerator(rescale=1./255)
train_generator = train_datagen.flow_from_directory(
    'dataset/train',
    target_size=(96, 96),
    batch_size=32,
    class_mode='categorical'
)

# Build a small CNN
model = tf.keras.Sequential([
    tf.keras.layers.Conv2D(16, (3,3), activation='relu', input_shape=(96,96,3)),
    tf.keras.layers.MaxPooling2D(2, 2),
    tf.keras.layers.Conv2D(32, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dense(3, activation='softmax')
])

model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
model.fit(train_generator, epochs=20)

# Convert and quantize
converter = tf.lite.TFLiteConverter.from_keras_model(model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_model = converter.convert()
```

### Deploying to ESP32
```cpp
#include <TensorFlowLite_ESP32.h>
#include "model.h"
#include "esp_camera.h"

// Capture image
camera_fb_t * fb = esp_camera_fb_get();

// Preprocess image (resize, normalize)
for(int i = 0; i < 96*96*3; i++) {
    input->data.int8[i] = (fb->buf[i] - 127);  // Normalize to [-128, 127]
}

// Run inference
TfLiteStatus invoke_status = interpreter.Invoke();

// Get results
int8_t max_score = -128;
int max_index = 0;
for(int i = 0; i < 3; i++) {
    if(output->data.int8[i] > max_score) {
        max_score = output->data.int8[i];
        max_index = i;
    }
}

Serial.printf("Prediction: Class %d (confidence: %d)\n", max_index, max_score);

esp_camera_fb_return(fb);
```

## Performance Metrics

### Inference Time
- Float32 model: ~500ms
- Quantized model: ~50ms (10x faster)

### Memory Usage
- Float32: 600KB RAM
- Quantized: 100KB RAM

### Power Consumption
- Continuous inference: 200mW
- Wake-on-event: 5mW average

## Best Practices

1. **Start Simple**: Begin with small models, optimize later
2. **Test Early**: Validate accuracy after quantization
3. **Profile Performance**: Measure inference time and memory
4. **Use Representative Data**: Calibration dataset should match real data
5. **Consider Edge Cases**: Test boundary conditions
6. **Optimize Preprocessing**: Can be bottleneck on microcontrollers

## Common Challenges

### Challenge 1: Accuracy Loss After Quantization
**Solution:** Use quantization-aware training
```python
import tensorflow_model_optimization as tfmot

quantize_model = tfmot.quantization.keras.quantize_model
q_aware_model = quantize_model(model)
q_aware_model.compile(optimizer='adam', loss='categorical_crossentropy')
q_aware_model.fit(train_data, train_labels, epochs=5)
```

### Challenge 2: Out of Memory
**Solution:** 
- Reduce model complexity
- Increase tensor arena size
- Use streaming for large inputs

### Challenge 3: Slow Inference
**Solution:**
- Enable hardware acceleration (if available)
- Optimize operations
- Reduce input size

## Hands-On Exercise

Create a gesture recognition system:
1. Collect accelerometer data for 3 gestures
2. Train a small neural network
3. Convert to TFLite
4. Deploy to ESP32
5. Test real-time classification

## Additional Resources
- TensorFlow Lite Micro documentation
- Edge Impulse tutorials
- TinyML book by Pete Warden
- Arduino TensorFlow Lite examples

## Next Steps
- Experiment with different architectures
- Try advanced optimization techniques
- Explore hardware accelerators
- Build a complete edge AI project
