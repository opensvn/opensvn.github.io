---
layout: post
title: Project Euler Problem 1
date: 2015-10-23 16:30:00
categories: ProjectEuler
excerpt: ProjectEuler第一题
---

# Multiples of 3 and 5

If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

Find the sum of all the multiples of 3 or 5 below 1000.

{% highlight python %}
sum = 3 * (1 + 999 / 3) * (999 / 3) / 2 + \
      5 * (1 + 999 / 5) * (999 / 5) / 2 - \
      15 * (1 + 999 / 15) * (999 / 15) / 2
print sum

{% endhighlight %}

# 3和5的倍数

如果我们列出所有小于10且为3或5的倍数的自然数，我们得到3，5，6和9。这些数的和是23。找出所有低于1000且为3或5倍数的和。
