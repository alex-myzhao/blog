---
title: vgg learning 1
tags:
---
## ARCHITECTURE

### Input

- 224 * 224 RGB Images
- Preprocessing:
  - subtract the mean RGB value, pixel-wise
- Conv
  - 3 * 3 filters
  - sometimes 1 * 1 filters
  - stride fixed to 1 pixel
  - preserved spatial resolution
- Max pooling
