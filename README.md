# Classical-Image-Segmentation-Pipeline-Mango-Leaf-Disease-Spot-Extraction

# Classical Image Segmentation Pipeline: Mango Leaf Disease & Spot Extraction

A small, fully classical (no deep learning) image-processing project that isolates a mango leaf from its background and extracts **anthracnose lesions** using color-based thresholding. The project compares:

1. **Otsu vs. GrabCut** for leaf isolation
2. **HSV vs. CIELAB (L\*a\*b\*)** thresholding for lesion segmentation

> **Scope note:** this is a learning project built around a single sample image to understand how color spaces behave on diseased tissue. The conclusions are qualitative. 
---

## Pipeline

<img width="256" height="610" alt="image" src="https://github.com/user-attachments/assets/615ec480-f37b-49f5-8e29-52addf2e3008" />



## 1. Leaf Isolation: Otsu vs. GrabCut

| | Otsu | GrabCut |
|---|---|---|
| Input | Grayscale intensity | Color image + initial rectangle |
| Idea | One global threshold from the histogram | GMM color models for foreground/background, refined by iterative graph cuts |
| Uses spatial context | No | Yes |
| Behavior on this image | Mask is sensitive to intensity; dark lesions and shadows can be confused with background, giving ragged boundaries | Follows the leaf contour closely and keeps lesions inside the leaf region |

**Result:** GrabCut produced the more precise leaf mask, so it is used as the first stage of the pipeline.

<!-- Replace with your own figure -->
<img width="1188" height="410" alt="image" src="https://github.com/user-attachments/assets/a26fa591-17ff-40df-917b-a957a823c1eb" />


Notes on the Otsu baseline:
- `THRESH_BINARY_INV` is needed if the leaf is darker than the background, otherwise the mask comes out inverted.
- Applying the mask with `cv2.bitwise_and` on the original image (instead of showing the binary mask) makes the comparison with GrabCut directly comparable.

---

## 2. Lesion Segmentation: HSV vs. CIELAB

After isolating the leaf, all color-space thresholding is done **only inside the GrabCut mask**, so the color space is the only variable being compared.

### CIELAB

- CIELAB Thresholding uses both chromatic channels, a* and b*:
- a* runs from green (low) to red (high). Healthy leaf tissue has low a*; necrotic brown tissue has higher a*.
- b* runs from blue (low) to yellow (high). Brown lesions are shifted toward yellow relative to green tissue, which adds a second cue to separate lesion from healthy leaf.
- Brightness is separated into L*, so very dark lesion centers still carry usable chromatic information.
- Result: a complete, contiguous lesion mask with smooth contours.

### HSV

- Thresholding on **Hue** failed to produce a coherent lesion region. In the Hue channel, lesion pixels appear as scattered, inconsistent values rather than one blob.
- Likely causes:
  - **Hue wrap-around:** red-brown colors sit at both ends of the hue circle (near 0 and near 179 in OpenCV), so a single `inRange` captures only part of the lesion.
  - **Hue instability at low V:** in very dark pixels, hue is not reliable.
- Result with hue-based thresholds: lesion **edges are detected but interiors are hollow**, with fragmented boundaries.
- The **Saturation** and **Value** channels show clearer lesion contrast than Hue, so the lesion information exists in HSV, just not in the hue channel.

<!-- Replace with your own figures -->
<img width="592" height="307" alt="image" src="https://github.com/user-attachments/assets/f7b041cb-ac6e-45eb-b068-b3f0fa1ecf8d" />

<img width="1178" height="380" alt="image" src="https://github.com/user-attachments/assets/45683294-13ff-4a97-9ff7-ef201e9d397c" />


### Summary

| Aspect | HSV (H,S,V) | CIELAB (a\* b\*) |
|---|---|---|
| Lesion interior | Hollow / incomplete | Filled |
| Boundary quality | Fragmented, noisy | Smooth, coherent |
| Dark lesion centers | less unreliable | Handled via a\* b\* independent of L\* |
| Red-brown wrap-around issue | Yes | No (a\*  b\*is linear, not circular) |


**Conclusion:** on this sample, CIELAB was the more practical color space for anthracnose lesions. This does not mean HSV is unusable: thresholding on S or V, or handling the hue wrap-around, may improve it.

---

## Repository Structure

```
.
├── README.md
├── segmentation_thresholding.ipynb         
├── sample_leaf.jpg 

           
```


## Requirements

```
python >= 3.8
opencv-python
numpy
matplotlib
jupyter
```

```bash
pip install opencv-python numpy matplotlib jupyter
jupyter notebook notebook.ipynb
```

