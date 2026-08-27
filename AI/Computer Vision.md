A CNN's purpose is to Detect high level features with as low as possible spatial resolution.

Torch vision which is a companion library to pytorch gives you access to common CV models

```python
from torchvision.models import resnet50

model = resnet50(weights="DEFAULT")
```

## Layers

Convolution layer 1 - Pooling Layer 1 - Convolution layer 2 - Pooling Layer 2 ..... Fully Connected - Fully Connected(Does Classification)

### Layer 1

**1 input with C channels**  
→ **K₁ filters** work on it  
→ **K₁ output feature maps**

### Layer 2

The **K₁ feature maps are stacked depth-wise** to become **1 input with K₁ channels**.  
→ **K₂ filters** work on it  
→ **K₂ output feature maps**

### Layer 3

The **K₂ feature maps are stacked depth-wise** to become **1 input with K₂ channels**.  
→ **K₃ filters** work on it  
→ **K₃ output feature maps**

### General Pattern

**C channels → K₁ feature maps/channels → K₂ feature maps/channels → K₃ feature maps/channels → ...**

**Key rule:** Number of filters in a layer = number of output feature maps = number of input channels to the next layer.

## Preprocessing

You generally use some open source tool to preprocess images.

### OpenCV

OpenCV is the standard OpenSource Computer Vision Library.

It loads the images as numpy arrays. If you are using pytorch down the line you would convert it to a pytorch tensor for enabling GPU operations and automatic differentiation useful for deep learning.

```python
import cv2

# Load image
image = cv2.imread("image.jpg")

# Resize
image = cv2.resize(image, (224, 224))

# Convert BGR -> RGB
image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

# Normalize pixel values from [0, 255] -> [0, 1]
image = image.astype("float32") / 255.0

# Grayscale conversion
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# Gaussian blur / noise reduction
blurred = cv2.GaussianBlur(image, (5, 5), 0)

# Edge detection
edges = cv2.Canny(gray, 100, 200)

# Crop
cropped = image[100:400, 200:500]

# Rotate
rotated = cv2.rotate(image, cv2.ROTATE_90_CLOCKWISE)

# Thresholding
_, thresholded = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

```
