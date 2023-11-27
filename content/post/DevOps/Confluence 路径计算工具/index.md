---
title: Confluence 附件路径计算工具
date: 2023-11-27
tags: ['Confluence','附件','路径']  
categories: ['DevOps','Confluence']
---

## 背景

Confluence 的某一个版本存在问题，当我跨空间去迁移页面的时候就会出现附件丢失的情况

## 代码

```Golang
package main

import (
	"fmt"
	"os"
)

func main() {
	// 使用os.Args获取命令行参数
	args := os.Args[1:]
	spaceId := args[0]
	pageId := args[1]
	attachmentPath := "ver003"

	for i := len(spaceId); i >= 0; i -= 3 {
		start := i - 3
		if start < 0 {
			start = 0
		}
		substr := spaceId[start:i]
		result := modulo(substr, 250)
		attachmentPath = attachmentPath + "/" + fmt.Sprintf("%d", result)
		if start == len(spaceId)-6 {
			attachmentPath = attachmentPath + "/" + spaceId
			break
		}
	}

	for i := len(pageId); i >= 0; i -= 3 {
		start := i - 3
		if start < 0 {
			start = 0
		}
		substr := pageId[start:i]
		result := modulo(substr, 250)
		attachmentPath = attachmentPath + "/" + fmt.Sprintf("%d", result)
		if start == len(pageId)-6 {
			attachmentPath = attachmentPath + "/" + pageId
			break
		}
	}
	fmt.Println(attachmentPath)
}

func modulo(substr string, module int) int {
	num := 0
	for _, ch := range substr {
		num = num*10 + int(ch-'0')
		num %= module
	}
	return num
}
```