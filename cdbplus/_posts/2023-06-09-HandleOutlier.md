---
layout: post
title: (c++) Outlier Handling
subtitle: c, c++, outlier, filtering, smoothing
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, outlier, filtering, smoothing]
comments: true
---

# Outlier Handling (Data fitting)
- 최초 작성일: 2023년 6월 9일 (금)

## 목차

[TOC]

<br/>

## 내용

1차원 데이터를 최대한 Outlier를 제거하여 부드럽게 피팅하고자 여러 가지 방법을 사용해보았다.

<br/>

## 코드

### 선형 보간법 

```c++
std::vector<float> CRbf::handleOutliers(const std::vector<float>& values, int windowSize, float threshold) {
    int size = values.size();
    std::vector<float> smoothedValues = values;

    for (int i = 0; i < size; ++i) {
        float sum = 0;
        int count = 0;

        for (int j = std::max(0, i - windowSize); j <= std::min(size - 1, i + windowSize); ++j) {
            sum += values[j];
            ++count;
        }

        float mean = sum / count;

        if (std::abs(values[i] - mean) > threshold) {
            // Outlier를 찾았으므로 선형 보간법으로 대체
            float prevValue = (i > 0) ? values[i - 1] : values[i];
            float nextValue = (i < size - 1) ? values[i + 1] : values[i];
            smoothedValues[i] = (prevValue + nextValue) / 2.0f;
        }
    }

    return smoothedValues;
}
```

<br>

###  이동 평균

```c++
std::vector<float> CRbf::handleOutliers(const std::vector<float>& values, int windowSize, float threshold) {
    int size = values.size();
    std::vector<float> smoothedValues = values;
    
    std::vector<float> movingAverages(size);
    for (int i = 0; i < size; ++i) {
        float sum = 0;
        int count = 0;

        for (int j = std::max(0, i - windowSize); j <= std::min(size - 1, i + windowSize); ++j) {
            sum += values[j];
            ++count;
        }

        movingAverages[i] = sum / count;
    }

    for (int i = 0; i < size; ++i) {
        if (std::abs(values[i] - movingAverages[i]) > threshold) {
            // Outlier를 찾았으므로 이동평균으로 대체
            smoothedValues[i] = movingAverages[i];
        }
    }

    return smoothedValues;
}
```

<br/>

