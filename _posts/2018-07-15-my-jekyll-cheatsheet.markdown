---
layout: post
title:  "My Jekyll Cheatsheet!"
date:   2018-07-15 02:24:02 -0700
categories: jekyll update
---
To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.markdown`.

Jekyll also offers powerful support for code snippets:

{% highlight python %}
def print_sth(str):
    print(str)
#=> e.g., prints 'hello world' to STDOUT.
{% endhighlight %}

jekyll also offers powerful support for latex math fomulations:

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$

To enable this functionality, we first specify `markdown: kramdown` in the `_config.yaml` file and add the following snippet in the front of `</head>` tag of `/_include/head.html` file (if it is not in the repository, use `bundle show minima` to find the location of minima theme):

{% highlight html %}
<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
{% endhighlight %}

Use `bundle exec jekyll serve` to build and run the website natively.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
