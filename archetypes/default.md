---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags = [""]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
categories = [""] # 文章类型
toc = true # 用于开启文章目录
preserveTaxonomyNames = true
disablePathToLower = true
---

