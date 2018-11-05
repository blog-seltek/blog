---
title: linux命令-cat
categories: FE
author: zhouang
tags:
  - linux
  - cat
---

# cat命令主要功能：
	1.打开文件
	2.创建文件
	3.合并文件

	
## 1.一次性打开文件：
	参数：
	-n 或 --number 由 1 开始对所有输出的行数编号
	-b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号
	-s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
	-v 或 --show-nonprinting
	
	cat -n zhouang.text // 打开名为zhouang.text，并在控制台中打印出行号（含空白行）
	cat -b zhouang.text // 打开名为zhouang.text，并在控制台中打印出行号（不含空白行）
	
## 2.创建文件
	cat > zhouang.text << EOF
	> 输入文件内容
	> EOF
	
	对文件进行追加:
	cat >> zhouang.text << EOF
	> 追加文件内容
	> EOF
	
## 3.合并两个文件
	cat zhouang1.text zhouang2.text
	
	合并两个文件到第三个文件（第三个文件的原始内容会被清空）
	cat zhouang1.text zhouang2.text > zhouang3.text
	
	
	