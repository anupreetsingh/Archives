CNN's

purpose is to Detect high level features with as low as possible spatial resolution.

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
