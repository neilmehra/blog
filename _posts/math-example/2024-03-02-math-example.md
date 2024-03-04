---
layout: post
title:  Post Example with MathJax
date:   2024-03-02
tags: cp
usemathjax: true
---

Below is an example of math using MathJax. 

Any page needing maths should start with the frontmatter:
{% highlight markdown %}
usemathjax: true
{% endhighlight %}

$$ 
\begin{align*}
y = y(x,t) &= A e^{i\theta} \\
&= A (\cos \theta + i \sin \theta) \\
&= A (\cos(kx - \omega t) + i \sin(kx - \omega t)) \\
&= A\cos(kx - \omega t) + i A\sin(kx - \omega t)  \\
&= A\cos \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big) + i A\sin \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big)  \\
&= A\cos \frac{2\pi}{\lambda} (x - v t) + i A\sin \frac{2\pi}{\lambda} (x - v t)
\end{align*}
$$

Inline math can be written with the `$` and `$` characters, producing inline equations
such as $\delta(t) \xrightarrow{\mathscr{F}} 1$.

The above is accomplished with thanks to [Bodun Hu](https://www.bodunhu.com/blog/posts/add-mathjax-support-to-jekyll-and-hugo/).
