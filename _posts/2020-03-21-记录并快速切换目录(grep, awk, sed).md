---

使用Shell实现快速跳转到经常访问的路径，类似`zsh`的`d` 命令可以显示多条最近访问的路径并快速跳转到目标路径，与其不同的是，保存的不是访问路径的历史记录而是自己认为需要保存以待下次使用的路径，且终端关闭后保存的路径不会被清除，下次打开时还能查看保存的路径并直接跳转，于是，下次电脑重启或者新打开终端时就可以快速切换到自己想去的路径而不需要借助终端命令的历史记录、文件浏览器或者命令补全了。将此命令命名为`mjump`。

当处于某一工作路径要从此路径退出或切换到其它路径，但是下次打开终端可能还要访问此路径时，可以直接输入`mjump -s`以将当前工作路径作持久化保存；下次打开终端想要查看以前保存了哪些路径时输入`mjump -o`即可；要切换到保存的某条路径时，输入`mjump -c n`，其中“n“表示目标路径的序号；在输入`mjump`而不带任何参数时会直接跳转到最后一次保存的路径；保存数量有限的路径，淘汰不经常访问的路径，且不重复保存。

可以将路径保存到文件使其持久化；保存的路径应按LRU思想排序，即第一条路径应是最后一次保存的路径，逐渐淘汰最近使用次数少的路径，每次保存新路径时直接插入到文件第一行，再删除文本最后一行，路径记录在文本中的行号表示路径的访问权重，行号越小权重越高，当要插入的新路径已经存在于文件时应先删除原先的路径再插入新的路径到第一行。

由于每次只向文件操作一条数据，可以用`sed`命令操作文件，对于macOS，`gnu-sed`相对系统默认的`sed`好用；  
对于保存路径功能：
1. 首先要获取当前路径；  
==> `pwd`
2. 判断文件中是否已经存在此路径；    
==> `grep -q $p $file`
3. 若不存在此路径则直接插入在第一行且删除最后一行；     
==> `gsed -i 1i/$p $file`        
==> `gsed -i 'n, nd' $file`   
4. 若存在此路径则先获得旧的路径的行号；    
==>   
```shell
line=`grep -n "$p$" $file  | awk -F ':' '{print $1}'` 
```
5. 删除该行    
==> `gsed -i "$line, ${line}d" $file`   
本来可以直接使用`sed`的搜索并删除功能，但是由于路径保存在$p，直接使用`gsed -i "/$p/d" $file`或类似命令会出现各种问题。      

切换路径：
1. 获取输入的参数即保存的路径的行号；    
==> `$#`表示参数个数，`$0`表示函数名，`$1`表示第一个参数，`$2`表示第二个参数
2. 根据行号取文件中的路径；    
==> `gsed -n 'n, np' $file`
3. 切换到目标路径；    
==> cd `gsed -n '1, 1p' $file`；   
==> 直接在终端执行此脚本会创建子线程运行，所以终端的工作目录不会改变，所以运行时要适用`source mjump -c n`，使其在shell父进程运行。

注意：
1. `sed`/`gsed`是“stream editor”，无法修改空文件，因此创建的文件不能是空文件，即只使用`touch`创建此文件是不够的，可以直接用`echo $HOME > $file`创建一个默认含有用户家目录的文件；
2. 每个目录字符串都是前缀码，故可简化在文件中判断某路径是否存在的表达式为`grep -q "$p$" $file `，即忽略正则表达式中的`^/`，其在`grep`的匹配中和变量引用的`$`在一起会产生混淆以致无法得到正确的搜索结果；
3. 存储的路径开头为两个`/`，这是导致各种匹配问题的根源；
4. `==`或`=`均可用于字符串比较，两边都要有空格，变量赋值的`=`两边不能有空格；
5. `[]`等同`test`，其中不能使用`||`，`&&`等，而`[[  ]]`可以

Code:

```shell
#! /bin/zsh


file="/Users/me/tools/rwp/path"
backup="/Users/me/tools/rwp"

if [ -e $file ]
then
else
    echo $HOME > $file
fi

if [[ $1 = "-s" || $1 = "--save" ]]
then
    p=`pwd`
    grep -q "$p$" $file 
    if [ $? -ne 0 ]
    then 
        gsed -i 1i/$p $file 
        file_lines=`wc -l $file | awk -F ' ' '{print $1}'`
        if [ $file_lines -eq 7 ]
        then
            gsed -i '7, 7d' $file
        fi
    else
        # gsed -i "s#/$p##g" $file
        # gsed -i "/^$/d" $file
        line=`grep -n "$p$" $file  | awk -F ':' '{print $1}'`
        gsed -i "$line, ${line}d" $file
        # gsed -i \\$p\\d $file
        gsed -i 1i/$p $file 
    fi
    echo "current path saved to $file"
elif [[ "$1" = '-o' || "$1" = '--out' ]] 
then 
    cat -n $file 
elif [[ "$1" = '-c' ||  "$1" = '--cd' ]]
then
    if [[ $2 == "" || $2 == "1" ]]
    then 
        cd `gsed -n '1, 1p' $file`
    elif [ $2 -eq 2 ]
    then 
        cd `gsed -n '2, 2p' $file`
    elif [ $2 -eq 3 ]
    then 
        cd `gsed -n '3, 3p' $file`
    elif [ $2 -eq 4 ]
    then 
        cd `gsed -n '4, 4p' $file`
    elif [ $2 -eq 5 ]
    then 
        cd `gsed -n '5, 5p' $file`
    elif [ $2 -eq 6 ]
    then 
        cd `gsed -n '6, 6p' $file`
    else
        echo "out of index"
    fi
elif [[ $1 == "-h" || $1 == "--help" ]]
then
    echo "file location: "
    ls -lT $file
    echo "-- use '-s' to save path;\n-- use '-o' to display saved path;\n-- use '-c' to change directory.\n"
elif [ $# -eq 0 ]
then 
    cd `gsed -n '1, 1p' $file`
else
    echo "mjump: wrong arguments.\n-- use '-s' to save path;\n--use '-o' to display saved path;\n-- use '-c' to change directory.\n"
fi
```

运行:
<div align="left">
    <img src="https://errorbeep.github.io/verbose-engine.io/images/shell01.png" width="50%">
</div>
使用了如下别名：
```shell
alias jump="source /Users/me/tools/rwp/mjump"
alias jo="jump -o"
alias jc="jump -c"
alias js="jump -s"
```





