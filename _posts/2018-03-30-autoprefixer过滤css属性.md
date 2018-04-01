---
layout: post
section-type: post
title: autoprefixer过滤“-webkit-box-orient：vertical”属性
category: actual_problem
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

###  在处理多行文本溢出变省略号时出现的这个问题。

<pre><code>
<span><</span><span>style></span>
  -webkit-box-orient: vertical;
  overflow: hidden;
  -webkit-line-clamp: 2;
  height: .72rem;
  line-height: .36rem;
  display: -webkit-box;
<span><</span><span>/style></span>
</code></pre>

这几个属性是用来处理多行文本溢出的，但是现在webpack编译以后，-webkit-box-orient: vertical检查元素样式并没有这个属性。并且打包好之后的css样式文件里也没有这个属性。

###解决办法

在使用-webkit-box-orient: vertical属性的位置加下以下代码。参考<a href="https://github.com/postcss/autoprefixer/issues/776">https://github.com/postcss/autoprefixer/issues/776</a>

<pre><code>
<span><</span><span>style></span>
  /* autoprefixer: off */
  -webkit-box-orient: vertical;
   /* autoprefixer: on */
<span><</span><span>/style></span>
</code></pre>

上诉代码在less中还是无效，需要简单修改一下。

<pre><code>
<span><</span><span>style></span>
  /*! autoprefixer: off */
  -webkit-box-orient: vertical;
   /* autoprefixer: on */
<span><</span><span>/style></span>
</code></pre>
