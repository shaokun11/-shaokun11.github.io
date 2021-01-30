---
title: 常用linux/vim命令(不定时更新)
date: 2021-01-31 00:09:09
---

#### 目标
-  常用操作的记录,不定时更新,方便查找

#### linux
```bash
- 查看末尾的的100行 tail 100 
- 查看少量文本  cat
- 查看大文件	less -mN text.txt (-mN显示进度与行号, 搜索 ? / ,从后往前查看 shif + G 移动到最后  space 向下翻页 b向上翻页),极力推荐
- 递归现实当前目录下的所有文件 ls -R
- 不断输出查看末尾的的100行 tail 100 -f
- 查看磁盘信息 df -h
- 查看当前目录的信息 du -h -d 1
- 移动到命令行首 ctrl + a
- 移动到命令行尾 ctrl + e
- 删除当前位置到行尾的数据 ctrl + k
- 删除当前位置到行首的数据 ctrl + u
- 清屏 ctrl + l
- 改名 mv test.txt test1.txt
- 复制 cp text.txt text1.txt (只能复制文件,可配合压缩来处理)
- 压缩gizp tar -zcvf test1.txt.tar.gz text.txt
- 解压gizp tar -zxvf test1.txt.tar.gz 
- 删除 rm -rf test.txt
```

#### vim
```javascript
- 跳转到页面顶部  gg
- 跳转到页面底部  G
- 向上半屏查看 ctrl + b
- 向下半屏查看 ctrl + f
- 显示行号 set nu
- 删除当前行 dd
- 拷贝1,4行到5行开始 1,4 copy 5 
- 移动1,4行到第5行开始 1,4 move 5
```


##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  



