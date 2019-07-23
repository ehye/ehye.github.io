---
title: 机器学习编程作业1 - Linear Regression
date: 2019-01-19 18:57:23
categories: MOOC
mathjax: true
tags:
	- Linear Regression
	- Gradient Descent
---

实现线性回归并可视化其对数据起作用<!-- more -->

[必做] warmUpExercise.m  -  Octave / MATLAB中的简单示例函数
[必做] plotData.m  - 显示数据集的函数
[必做] computeCost.m  - 计算线性回归成本的函数
[必做] gradientDescent.m  - 运行梯度下降的函数
[选作] computeCostMulti.m  - 多个变量的成本函数
[选作] gradientDescentMulti.m  - 多个变量的梯度下降
[选作] featureNormalize.m  - 规范化功能的功能
[选作] normalEqn.m  - 计算正规方程的函数

---

- warmUpExercise.m
输出5x5的单位矩阵，作为热身练习，PDF中给出了解答，当作作业提交的测试
```matlab
A = eye(5,5);
```

- plotData.m
绘制数据点，并给出数字轴的人口和利润标签。PDF中给出了解答
```matlab
plot(x, y, 'rx', 'MarkerSize', 10); % Plot the data
ylabel('Profit in $10,000s'); % Set the y−axis label
xlabel('Population of City in 10,000s'); % Set the x−axis label
```

- computeCost.m
    
    设计代价函数 $$J(\theta_0,\theta_1)=\frac{1}{2m}\displaystyle\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2$$
```matlab
predictions = X*theta;
sqrErrors = (predictions-y).^2;
J = 1/(2*m) * sum(sqrErrors);

% J = sum((X*theta-y).^2) / (2*m);
```

- gradientDescent.m
梯度下降算法的实现，训练集的第一列已全为1，注意要同时更新θ。
```matlab
    temp1=theta(1)-alpha*sum((X*theta-y).*X(:,1))/m;
    temp2=theta(2)-alpha*sum((X*theta-y).*X(:,2))/m;
    theta(1)=temp1;
    theta(2)=temp2;    
```

- featureNormalize.m

    > 参考讲义4-P12

    $x_1=\frac{x_1-\mu_1}{s_1}$
```matlab
for i=1:size(X,2)
    mu(i)=mean(X(:,i));
    sigma(i)=std(X(:,i));
end
X_norm=(X_norm - mu) ./ sigma;
```

- computeCostMulti.m

    > 参考讲义4-P7 

    $$J(\theta)=\frac{1}{2m}\displaystyle\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2$$
```matlab
J = sum((X*theta-y).^2/(2*m)); % 4-P7
```

- gradientDescentMulti.m
参考讲义4-P8
```matlab
temp=theta;
for t = 1:length(theta)
    temp(t,1)=theta(t,1)-alpha*sum((X*theta-y).*X(:,t))/m;
end
theta=temp;
```

- normalEqn.m

    > 参考讲义4-P27 

    $\theta=(X^TX)^{-1}X^Ty$
```matlab
theta=pinv(X'*X)*X'*y
```