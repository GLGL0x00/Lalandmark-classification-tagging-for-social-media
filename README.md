# 🏛️ Landmark-classification&tagging-for-social-media 🔍

A deep learning project to classify landmark images using a custom CNN architecture and explain model predictions with Grad-CAM visualizations.

---

## 📌 Project Overview

This project was part of the **AWS Machine Learning Engineer Nanodegree** by **Udacity and AWS**, focusing on building an image classifier for world landmarks.

The goal was to design a Convolutional Neural Network (CNN) from scratch and then enhance its performance using transfer learning. A key part of the project involved experimenting with different architectures to boost accuracy and generalization.

💡 My standout contributions:
- Implemented **residual connections** in the custom CNN to improve training stability and depth.
- Used **Grad-CAM** to generate heatmaps that visualize which parts of the image most influenced the model’s predictions, making the model more interpretable.

---

## 📂 Dataset

- **Source**: A curated subset of the [Google Landmarks Dataset](https://github.com/cvdfoundation/google-landmark) provided by Udacity as part of the AWS Machine Learning Engineer Nanodegree program.
- **Classes**: 50 world landmarks
- **Images**: ~100 images per class (~5,000 total)
- **Balanced**: Yes (approximately equal images per class)
- **Purpose**: Educational — designed for hands-on learning and model experimentation.

### 🧪 Preprocessing & Augmentation

Data augmentation pipeline was used to simulate real-world image variations and improve model robustness:

```python
transforms.Compose([
    transforms.Resize(256),
    transforms.RandomCrop(224),
    transforms.RandomResizedCrop(size=(224, 224), scale=(0.8, 1.0)),
    transforms.RandomAffine(degrees=10, translate=(0.1, 0.1), scale=(0.9, 1.1), shear=5),
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.GaussianBlur(kernel_size=5, sigma=(0.1, 2.0)),
    transforms.ToTensor(),
    transforms.Normalize(mean, std)
])
```
💡 **Why this pipeline?**

- Input Size (224×224): Matches PyTorch pre-trained model expectations.

- Affine & Flip: Handle different angles, distances, and mirrored images.

- ColorJitter: Simulate lighting/camera variations.

- Blur: Mimic low-quality or out-of-focus images.

- RandomResizedCrop: Emulate zooming and framing differences.

---

## 🧠 Model Architecture
<details>
  <summary><strong>CNN From Scratch Architecture Summary</strong>
    <blockquote>
      <strong>Model Flow:</strong> Input → [Conv-BN-ReLU + MaxPool] ×5 → GAP → FC(50)
    </blockquote>
  </summary>

  <br>

  <table>
    <thead>
      <tr>
        <th>Stage</th>
        <th>Layers</th>
        <th>Output Shape</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>Input</strong></td>
        <td>—</td>
        <td><code>[3, 224, 224]</code></td>
      </tr>
      <tr>
        <td><strong>Block 1</strong></td>
        <td>Conv(64) → BN → ReLU → MaxPool</td>
        <td><code>[64, 112, 112]</code></td>
      </tr>
      <tr>
        <td><strong>Block 2</strong></td>
        <td>Conv(128) → BN → ReLU → MaxPool</td>
        <td><code>[128, 56, 56]</code></td>
      </tr>
      <tr>
        <td><strong>Block 3</strong></td>
        <td>Conv(256) ×2 → BN → ReLU → MaxPool</td>
        <td><code>[256, 28, 28]</code></td>
      </tr>
      <tr>
        <td><strong>Block 4</strong></td>
        <td>Conv(512) ×2 → BN → ReLU → MaxPool</td>
        <td><code>[512, 14, 14]</code></td>
      </tr>
      <tr>
        <td><strong>Block 5</strong></td>
        <td>Conv(512) ×2 → BN → ReLU → MaxPool</td>
        <td><code>[512, 7, 7]</code></td>
      </tr>
      <tr>
        <td><strong>Head</strong></td>
        <td>GAP → Flatten → Dropout → FC(50)</td>
        <td><code>[50]</code></td>
      </tr>
    </tbody>
  </table>

</details>



  
<details>
  <summary><strong>ResNet-style CNN Architecture Summary</strong>
    <blockquote><strong>Model Flow:</strong> Input → [ResBlock + MaxPool] ×5 → GAP → FC(50)</blockquote>
  </summary>
  <table>
    <thead>
      <tr>
        <th>Stage</th>
        <th>Layers</th>
        <th>Output Shape</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>Input</strong></td>
        <td>—</td>
        <td><code>[3, 224, 224]</code></td>
      </tr>
      <tr>
        <td><strong>Block 1</strong></td>
        <td>ResidualBlock(3→64)<br>→ Conv-BN-ReLU ×2 + SkipConv (1×1) + ReLU<br>+ MaxPool(2×2)</td>
        <td><code>[64, 112, 112]</code></td>
      </tr>
      <tr>
        <td><strong>Block 2</strong></td>
        <td>ResidualBlock(64→128) + MaxPool</td>
        <td><code>[128, 56, 56]</code></td>
      </tr>
      <tr>
        <td><strong>Block 3</strong></td>
        <td>ResidualBlock(128→256) ×2 + MaxPool</td>
        <td><code>[256, 28, 28]</code></td>
      </tr>
      <tr>
        <td><strong>Block 4</strong></td>
        <td>ResidualBlock(256→512) ×2 + MaxPool</td>
        <td><code>[512, 14, 14]</code></td>
      </tr>
      <tr>
        <td><strong>Block 5</strong></td>
        <td>ResidualBlock(512→512) ×2 + MaxPool</td>
        <td><code>[512, 7, 7]</code></td>
      </tr>
      <tr>
        <td><strong>Block 6</strong></td>
        <td>ResidualBlock(512→512) ×2 + MaxPool</td>
        <td><code>[512, 7, 7]</code></td>
      </tr>
      <tr>
        <td><strong>Head</strong></td>
        <td>GlobalAvgPool → Flatten → Dropout(0.5) → Linear(512→50)</td>
        <td><code>[50]</code></td>
      </tr>
    </tbody>
  </table>

</details>

You can inspect the final model with TorchScript:
```python
torch.jit.load("model_scripted.pt")
