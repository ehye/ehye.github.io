---
title: 机器学习编程作业3 - Multi-class Classification and Neural Networks
date: 2019-02-02 14:46:14
categories: MOOC
tags:
	- Machine Learning
	- Neural Networks
---

实现一对多逻辑回归和利用神经网络识别手写阿拉伯数字<!-- more -->

lrCostFunction.m - Logistic regression cost function
oneVsAll.m - Train a one-vs-all multi-class classifier
predictOneVsAll.m - Predict using a one-vs-all multi-class classifier
predict.m - Neural network prediction function

---

- lrCostFunction.m
```matlab
% 7 P21
h = sigmoid(X*theta);
% theta1 = [0 ; theta(2:end, :)];
temp = theta;
temp(1) = 0;
p = lambda*(temp'*temp)/(2*m);
J = ((-y)'*log(h) - (1-y)'*log(1-h))/m + p;

% 7 P22
grad = X'*(sigmoid(X*theta)-y)/m;
temp = theta;
temp(1) = 0;
grad = grad + lambda/m*temp;
```

- oneVsAll.m
```matlab
for c = 1:num_labels
    % Set Initial theta
    initial_theta = zeros(n + 1, 1);

    % Set options for fminunc
    options = optimset('GradObj', 'on', 'MaxIter', 50);

    % Run fmincg to obtain the optimal theta
    % This function will return theta and the cost 
    [theta] = fmincg (@(t)(lrCostFunction(t, X, (y == c), lambda)), initial_theta, options);
    all_theta(c, :) = theta';
end
```

- predictOneVsAll.m
```matlab
%  6 P31
[p_max, i_max] = max(sigmoid(X*all_theta'), [], 2);
p = i_max;
```

- predict.m
```matlab
% 8 P23
a1 = [ones(m, 1) X]; % mx1 +> X
z2 = a1*Theta1';
% z2rows x 1 col +> sigmoid
a2 = [ones(size(z2, 1), 1) sigmoid(z2)];
z3 = a2*Theta2';
a3 = sigmoid(z3);

[p_max, p] = max(a3, [], 2);
```