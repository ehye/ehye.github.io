---
title: 机器学习编程作业2 - Logistic Regression
date: 2019-01-27 18:26:09
categories: MOOC
tags:
	- Machine Learning
	- Logistic Regression

---
实现线性回归并应用到两种不同的数据集里<!-- more -->

plotData.m - Function to plot 2D classification data
sigmoid.m - Sigmoid Function
costFunction.m - Logistic Regression Cost Function
predict.m - Logistic Regression Prediction Function
costFunctionReg.m - Regularized Logistic Regression Cost

--- 

- plotData.m
PDF第3页给出
```matlab
% Find Indices of Positive and Negative Examples
pos = find(y==1); neg = find(y == 0);
% Plot Examples
plot(X(pos, 1), X(pos, 2), 'k+','LineWidth', 2, 'MarkerSize', 7);
plot(X(neg, 1), X(neg, 2), 'ko', 'MarkerFaceColor', 'y', 'MarkerSize', 7);
```

- sigmoid.m
$g(z) = \dfrac{1}{1 + e^{-z}}$
数组也需适用，要用exp
```matlab
g = 1 ./ (1 + exp(-z));
```

- costFunction.m
```matlab
% p21
J = sum(-y'*log(sigmoid(X*theta))-(1-y)'*log(1-sigmoid(X*theta))) / m;
% p16
grad = (X'*(sigmoid(X*theta)-y)) ./ m;
```

- predict.m
```matlab
for i = 1:size(p)
    if (sigmoid(X*theta) >= 0.5)
        p(i) = 1;
    else
        p(i) = 0;
    end
end
```

- costFunctionReg.m
```matlab

```