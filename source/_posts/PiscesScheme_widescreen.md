---
title: 使 next 主题适应移动设备
date: 2016-12-16 13:15:56
categories: Blog
tags: 
	- hexo
	- CSS
	- NexT
---
修改 PiscesScheme 的宽度，使 next 主题适应移动设备
> 对于Pisces Scheme ，需要同时修改 **header** 的宽度、**.main-inner** 的宽度以及 **.content-wrap** 的宽度。例如，使用百分比（Pisces 的布局定义在 source/css/_schemes/Picses/_layout.styl 中）：

```css
.header{ width: 90%; }
.container .main-inner { width: 90%; }
.content-wrap { width: calc(100% - 260px); }
```

> 查看Github上的[Issue](https://github.com/iissnan/hexo-theme-next/issues/759#issuecomment-202242848)

实际操作后发现此时移动端页面显示不完整，相应修改如下

```css
.header .container .main-inner {
  width: 90%;
  +tablet() { width: 100%; }
  +mobile() { width: 100%; }
}
.content-wrap {
  width: calc(100% - 260px);
  +tablet() { width: 100%; }
  +mobile() { width: 100%; }
}
.site-nav-toggle {
  +tablet() { top: 10px; }
  +mobile() { top: 11px; }
}
.site-meta {
  +tablet() { padding: 1px 0;}
  +mobile() { padding: 1px 0;}
}
.site-nav-toggle button {
  margin-top: 20px;
}
```

最新的 Gemini 主题适用
