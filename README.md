# sindre-imgaug

[![PyPI version](https://img.shields.io/pypi/v/sindre-imgaug.svg)](https://pypi.org/project/sindre-imgaug/)
[![Python Versions](https://img.shields.io/pypi/pyversions/sindre-imgaug.svg)](https://pypi.org/project/sindre-imgaug/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Fork 说明**: 本项目是 [imgaug](https://github.com/aleju/imgaug) 的一个 fork，主要目标是支持最新的 NumPy 版本（NumPy 2.x），同时保持向后兼容。

**sindre-imgaug** 是一个图像增强（data augmentation）库，帮助你在机器学习项目中扩充图像数据集。它可以将一组输入图像转换为大量经过轻微变化的图像，支持多种增强方式及标注数据的同步增强。

---

## 安装

```bash
# 通过 pip 安装（推荐）
pip install sindre-imgaug

# 或从源码安装最新版
pip install git+https://github.com/SindreYang/sindre-imgaug.git
```

## 使用

```python
import imgaug as ia
import imgaug.augmenters as iaa

# 注意：导入时使用 imgaug，不是 import sindre-imgaug
```

### 快速开始

```python
import numpy as np
import imgaug.augmenters as iaa

# 创建随机测试图像
images = np.random.randint(0, 255, (4, 128, 128, 3), dtype=np.uint8)

# 定义一个简单的增强序列
seq = iaa.Sequential([
    iaa.Fliplr(0.5),                 # 50% 概率水平翻转
    iaa.GaussianBlur(sigma=(0, 2.0)), # 高斯模糊
    iaa.AdditiveGaussianNoise(scale=(0, 0.05*255)),  # 高斯噪声
    iaa.Affine(rotate=(-20, 20)),    # 随机旋转 -20° ~ 20°
])

# 对图像进行增强
images_aug = seq(images=images)
```

### 同时增强图像与标注

```python
import imgaug.augmenters as iaa

seq = iaa.Sequential([
    iaa.Fliplr(0.5),
    iaa.Affine(translate_px={"x": (-10, 10)}),
])

# 同时增强图像和关键点/边界框/分割图
images_aug, kps_aug = seq(images=images, keypoints=keypoints)
images_aug, bbs_aug = seq(images=images, bounding_boxes=bbs)
images_aug, segmaps_aug = seq(images=images, segmentation_maps=segmaps)
```

---

## 相比原始仓库的特性与修复

### NumPy 2.x 兼容性
- 替换已弃用的 `np.bool` 为 `bool`，`np.complex` 为 `np.complex_`
- 替换已移除的 `np.sctypes` 为硬编码的 dtype 集合（`NP_FLOAT_TYPES`、`NP_INT_TYPES`、`NP_UINT_TYPES`）
- 修复因 NumPy 2.0 移除 `np.float128` 导致的兼容性问题
- 修复 `np.sctypes` 在 NumPy 2.0 中被完全移除导致的 `AttributeError`
- 更新最低 NumPy 版本要求至 `>=1.21`

### 构建与依赖
- 修复 `setuptools >= 82` 下 `pkg_resources` 被移除导致的构建失败
- 将元数据迁移至 `pyproject.toml`，支持现代 Python 打包标准（PEP 517/518）
- 新增 `opencv-python` 作为备选依赖
- 更新 `scikit-image` 最低版本至 `>=0.17`
- 移除过时的 `imagecorruptions-imaug` 依赖

### 功能改进
- `OneOf` 支持离散概率分布，提供更灵活的增强选择
- 分割图（Segmentation Maps）和热力图（Heatmaps）支持更多填充模式
- 改进 RNG 采样性能，大幅提升随机数生成效率
- 修复 `BoundingBoxesOnImage` 的赋值 bug
- 修复 `LUT_CACHE` 为 `None` 时的运行时错误
- 修复 `geometric.py` 中的变量误用 bug
- 支持多张图像的归一化处理
- 添加了 `RandAugment` 支持

### Python 版本支持
- 支持 Python 3.8 ~ 3.14
- 移除了对 Python 2.7 / 3.4 ~ 3.7 的过时支持声明
- 全面适配现代 Python 类型提示

---

## 支持的增强功能

| 类别 | 增强器 |
|------|--------|
| **元操作** | `Sequential`、`SomeOf`、`OneOf`、`Sometimes`、`WithChannels` 等 |
| **几何变换** | `Affine`、`PiecewiseAffine`、`PerspectiveTransform`、`ElasticTransformation`、`Rot90`、`Jigsaw` |
| **颜色变换** | `Add`、`Multiply`、`AddToHueAndSaturation`、`Grayscale`、`ChangeColorTemperature` |
| **对比度** | `GammaContrast`、`SigmoidContrast`、`LogContrast`、`CLAHE`、`HistogramEqualization` |
| **模糊/噪声** | `GaussianBlur`、`AverageBlur`、`MedianBlur`、`MotionBlur`、`AdditiveGaussianNoise`、`Dropout` |
| **翻转** | `Fliplr`、`Flipud` |
| **裁剪/填充** | `Crop`、`Pad`、`CropAndPad`、`CropToFixedSize`、`PadToFixedSize`、`Resize` |
| **分割** | `Superpixels`、`Voronoi`、`UniformVoronoi`、`RegularGridVoronoi` |
| **池化** | `AveragePooling`、`MaxPooling`、`MinPooling`、`MedianPooling` |
| **天气效果** | `FastSnowyLandscape`、`Clouds`、`Fog`、`Snowflakes`、`Rain` |
| **艺术效果** | `Cartoon` |
| **卷积** | `Sharpen`、`Emboss`、`EdgeDetect`、`DirectedEdgeDetect` |
| **腐蚀/噪声** | `SaltAndPepper`、`CoarseDropout`、`Cutout`、`Invert`、`Solarize`、`JpegCompression` |
| **集合** | `RandAugment` |

## 支持的标注类型

- 图像（uint8 全支持，其他 dtype 参见文档）
- 热力图（float32）
- 分割图（int32）
- 掩码（bool）
- 关键点/地标（int/float 坐标）
- 边界框（int/float 坐标）
- 多边形（int/float 坐标）
- 线串（int/float 坐标）

---

## 文档

更多详细文档和示例，请访问原始项目的文档站点：
- [原项目文档](https://imgaug.readthedocs.io/)
- [API 参考](https://imgaug.readthedocs.io/en/latest/source/api.html)
- [增强器概览](https://imgaug.readthedocs.io/en/latest/source/overview_of_augmenters.html)

---

## 许可

本项目基于 MIT 协议开源。

---

**原始仓库**: [https://github.com/aleju/imgaug](https://github.com/aleju/imgaug)

