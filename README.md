# ELiTNet model trained on IDRiD and US Nerve Dataset

## Architectural Changes

The baseline ELiTNet architecture was modified to improve segmentation 
performance while maintaining computational efficiency across two different 
medical imaging datasets (IDRiD retinal images and Ultrasound nerve images). 
The main changes include:
- A single shared ELiTNet model for both datasets with num_classes=6.
- Dataset-specific augmentation was applied, with heavier augmentations
  for IDRiD images and lighter augmentations for ultrasound images to
  better match each dataset's characteristics.
- Balanced multi-dataset training was implemented using a weighted random
  sampler with a 6:1 Ultrasound-to-IDRiD sampling ratio, ensuring the
  smaller retinal dataset remained well represented during optimization.
- In module.py, SeparableConv2d. was replace with nn.Conv2d due to it being
  absent in kaggle.
- Certain bugs were also fixed like changing MaxPool1d to MaxPool2d in
  the ConvBlock, and skip connections were reinstated by passing the value
  of down_activations, which was absent earlier.

## Loss Function

Training used a weighted hybrid loss combining three complementary objectives:

Dice Loss (weight = 0.4)
Focal Loss (weight = 0.3)
Cross-Entropy Loss (weight = 0.3)

This combination improves class balance, handles difficult samples, 
and optimizes segmentation overlap simultaneously.

## Training Hyperparameters

| Parameter               | Value                                                             |
| ----------------------- | ----------------------------------------------------------------- |
| Input size              | 128 × 128                                                         |
| Batch size              | 16                                                                |
| Number of classes       | 6                                                                 |
| Optimizer               | AdamW                                                             |
| Learning rate           | 3 × 10⁻⁴                                                          |
| Weight decay            | 1 × 10⁻⁴                                                          |
| Learning rate scheduler | CosineAnnealingWarmRestarts (T₀ = 30, Tmult = 2, ηmin = 1 × 10⁻⁶) |
| Epochs                  | 90                                                                |
| Gradient clipping       | 1.0                                                               |

## Final Performance

| Dataset          | Dice Score |        IoU |
| ---------------- | ---------: | ---------: |
| IDRiD            |   **0.64** |   **0.64** |
| Ultrasound Nerve | **0.8791** | **0.8666** |

Best average validation Dice score: 0.7867
