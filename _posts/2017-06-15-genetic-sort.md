---
layout: post
title: test
permalink: /:title
---

Recently my friend has been working on a course scheduling algorithm for the
university we went to. The algorithm has to schedule courses in such a way that it
satisifies several types of constraints, such as matching professors' schedule
preferences and preventing conflicts between courses that are a prerequisite for
a common course. The problem is NP-Complete, but through some clever optimizations
and dynamic programming he was able to generate passing schedules for moderate
problem sizes... sometimes. 

Apparently the success rate of his algorithm is largely dependent on the order in 
which it applies the constraints. Certain constraint permutations yeilded fantastic 
results while others lead to schedules with unworkable conflicts. 


{% highlight python %}
print('Hello World!')
{% endhighlight %}
