####

## 变量赋值情况
- 单引号：表示不解析，保留原始数据,所见即所得，即将单引号内的内容原样输出
- 双引号：会尝试解析,如果内容中有命令、变量等，会先把变量、命令解析出结果，然后在输出最终内容
- 无引号：和双引号一致，会把里面的变量替换为变量的内容
- 反引号：会将里面的命令执行，并把结果赋值给变量，等同于 $() 的效果

```shell
# 双引号
martin@yds-server:~/shell$ name="wudi"
martin@yds-server:~/shell$ echo $name
wudi
# 双引号：替换变量
martin@yds-server:~/shell$ name2="1${name}23"
martin@yds-server:~/shell$ echo $name2
1wudi23
# 单引号：不解析
martin@yds-server:~/shell$ name2='$name'
martin@yds-server:~/shell$ echo $name2
$name
martin@yds-server:~/shell$ name2=a${name}a
martin@yds-server:~/shell$ echo $name2
awudia
# 反引号 ：执行命令
martin@yds-server:~/shell$ name3=`sadfas`
sadfas: command not found
martin@yds-server:~/shell$ name3=`ls`
martin@yds-server:~/shell$ echo $name3
hell.sh
```

- $# 脚本文件后面的参数个数

- 当 $* 和 $@ 不被双引号" "包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。

- 但是当它们被双引号" "包含时，就会有区别了：

    - "$*"会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。
    - "$@"仍然将每个参数都看作一份数据，彼此之间是独立的。
执行结果比较：
```
    #!/bin/bash 
    echo "print each param from \"\$*\""
    for var in "$*"
    do
        echo "$var"
    done
    echo "print each param from \"\$@\""
    for var in "$@"
    do
        echo "$var"
    done
```
```shell
[mozhiyan@localhost demo]$ . ./test.sh a b c d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d

```
### 返回最大的数字
```shell
#!/bin/bash
echo "你好"
max=$1
for a in "$@"; do
    echo $a
    if (("$a" > "$max")); then
        max = $a
    fi
done
echo 最大的是：$max
```