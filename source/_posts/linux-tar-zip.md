---
title: Linux 下压缩与解压缩命令 zip 和 tar
date: 2017-10-12 23:42:00
description: 总结常用的压缩与解压缩命令 zip 与 tar
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

# zip

1. 在 `/home/blog`目录下，压缩 `source` 目录，命令为 `source.zip`

  ```
  zip -r source.zip source
  ```
  ![][1]
2. 将 `source.zip` 文件解压缩至指定目录；不加 `-d` 参数，默认是在当前目录下

  ```
  unzip source.zip -d source_dir
  ```
  ![][2]
  
3. 查看 `source.zip` 文件的完整性

  ```
  unzip -t source.zip
  ```
4. 查看 `source.zip` 文件的内容

  ```
  unzip -v source.zip
  ```
5. 将 `source.zip` 中的所有文件扁平解压至 `zip` 文件所在目录

  ```
  unzip -j source.zip
  ```
# tar

1. 压缩

  ```
  tar -cvf source.tar source　　　　 <== 仅打包，不压缩！
  tar -zcvf source.tar.gz source　　 <== 打包后，以 gzip 压缩
  tar -jcvf source.tar.bz2 source　　<== 打包后，以 bzip2 压缩
  ```
2. 解压缩

  ```
  tar -xvf source.tar         <== 解压缩 打包文件
  tar -zxvf source.tar.gz     <== 解压缩 以 gzip 压缩的打包文件
  tar -jxvf source.tar.bz2    <== 解压缩 以 bzip2 压缩的打包文件
  ```

3. 查看

  ```
  tar -tvf source.tar
  ```


## 主选项
- `-c`： 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。相当于打包。
- `-x`：从档案文件中释放文件。相当于拆包。
- `-t`：列出档案文件的内容，查看已经备份了哪些文件。

## 辅助选项
- `-z`： 是否同时具有 `gzip` 的属性？亦即是否需要用 `gzip` 压缩或解压？一般格式为 `xx.tar.gz` 或 `xx. tgz`
- `-j`： 是否同时具有 `bzip2` 的属性？亦即是否需要用 `bzip2` 压缩或解压？一般格式为 `xx.tar.bz2`
- `-v`：压缩的过程中显示文件！这个常用
- `-f`：使用档名，请留意，在 f 之后要立即接档名喔！不要再加其他参数！
- `-p`：使用原文件的原来属性（属性不会依据使用者而变）
- `--exclude FILE`：在压缩的过程中，不要将指定 `FILE` 打包！可使用正则 `*`，要排除多个文件，则多次使用本参数；



  [1]: http://owk2q4gs5.bkt.clouddn.com/linux-zip.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/unzip.png
