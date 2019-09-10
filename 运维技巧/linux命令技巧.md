## 一些linux下的命令技巧

### 提取某一列(如ip)并排序

文件内容

```
test 1.1.1.1 xxxx
test1 2.2.2.2 xxx
......
```
提取Ip列表：

`cat 1.txt | akw {print $2} | sort`