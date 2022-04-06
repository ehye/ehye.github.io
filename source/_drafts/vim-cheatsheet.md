---
title: git-cheatsheet
tags:
	- vim
---

- 查找

```vim
/\<word>\
```

- 初始化 .vimrc

```vim
set nu
syntax on
colorscheme desert
```

- [复制，剪切和粘贴](https://vim.fandom.com/wiki/Copy,_cut_and_paste)

1. 将光标定位在要开始切割的位置
2. 按v选择字符，或按大写V选择整行，或按Ctrl-v选择矩形块（vG全选）
3. 将光标移到要剪切的末尾
4. 按d剪切（或y进行复制）
5. 移至您要粘贴的位置
6. 按P粘贴在光标之前，或p粘贴在光标之后

- 全选（高亮显示）：按esc后，然后ggvG或者ggVG