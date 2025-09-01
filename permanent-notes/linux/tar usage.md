---
"type:": permanent-note
"title:": tar usage
"id:": 20250607135513
"created:": 2025-06-07T13:55:13
tags:
  - permanent-note
  - linux
  - cli/tar
related-context: 
related-notes:
---
`tar` 可以用于打包和压缩/解压缩文件。

## 常用选项

| 选项   | 含义                |
| ---- | ----------------- |
| `-c` | 创建压缩包 (create)    |
| `-x` | 解压压缩包(extract)    |
| `-v` | 显示过程(verbose)     |
| `-f` | 指定文件名(file)       |
| `-z` | 使用`gzip`压缩或解压     |
| `-C` | 指定解压目录(directory) |

## 常用命令组合

```shell
tar -cvf XXX.tar ./folder
tar -xvf XXX.tar

tar -zcvf XXX.tar.gz ./folder
tar -zxvf XXX.tar.gz

tar -zxvf XXX.tar.gc -C /path/to/dir
```
