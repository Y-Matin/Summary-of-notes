### tar

#### tar命令的打包操作

-  创建tar文件
```
tar -cvf 打包文件的名字.tar 文件名字或路径
```
> -c：创建新的tar文件(create 表示建包的意思)  
-v：显示操作详细信息 （verbose 冗长的，详细，表示命令执行时会显示更多的信息）  
-f：指定要创建的tar文件的文件名（file 指定tar包的文件名）
- 可以使用以下命令创建一个名为archive.tar的tar文件，其中包含目录/ home / user /和文件/home/user/file1和/home/user/file2：
    ```
    tar -cvf archive.tar /home/user/ /home/user/file1 /home/user/file2
    ```

#### tar命令的解包操作
- 解压
```
tar -xvf 需要解包文件的名字.tar  [-C   解压文件的存储路径]
```
> -x：提取文件（extrac 提取 取出 表示解包的意思）  
-v：显示操作详细信息  
-f：指定要提取的tar文件的文件名  
-C: 指定解压文件的存储路径


- 要将tar文件解压缩到当前目录，可以使用以下命令
```
tar -xvf archive.tar

```



#### tar命令的建立压缩包操作

