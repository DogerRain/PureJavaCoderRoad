## shell编程的一些习惯

1. 开头指定使用什么shell，例如：bash，ksh，csh等
2. 脚本功能描述，使用方法，作者，版本，日期等
3. 变量名，函数名要有实际意义，函数名以动名词形式，第二个单词首字母要大写。例如：updateConfig()
4. 缩进统一用4个空格，不用TAB



## 1、if

```bash
if 判断条件 1 ; then
  条件为真的分支代码
elif 判断条件 2 ; then
  条件为真的分支代码
else
  以上条件都为假的分支代码
fi
```

demo：

```bash
#!/bin/bash
# 用一个age变量接收输入的行的值
read -p "Please input your age: " age
# ~表示非，表示是否有除了0~9的以外的字符
if [[ $age =~ [^0-9] ]] ;then
    echo "please input a int"
    exit 10
    # 大于
elif [ $age -ge 150 ];then
    echo "your age is wrong"
    exit 20
    # 小于
elif [ $age -gt 18 ];then
    echo "good good work,day day up"
else
    echo "good good study,day day up"
fi
```



exit 退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。

## 2、case

```bash
case $name in;
PART1)
  cmd
  ;;
PART2)
  cmd
  ;;
*)
  cmd
  ;;
esac
```

注意，case支持一些通配符：

```bash
  *: 任意长度任意字符
  ?: 任意单个字符
  [] ：指定范围内的任意单个字符
  a|b: a 或b
```

demo：

```bash
#判断yes or no
#!/bin/bash
read -p "Please input yes or no: " anw
case $anw in
[yY][eE][sS]|[yY])
    echo yes
    ;;
[nN][oO]|[nN])
    echo no
    ;;
*)
    echo false
    ;;
esac
```

## 3、循环

```bash
# 语法一
for name in 列表 ;do
  循环体
done

# 语法二
for (( exp1; exp2; exp3 )) ;do
  循环体
done
```

demo：

```bash
#求出（1+2+...+n）的总和
#!/bin/bash
sum=0
read -p "Please input a positive integer: " num
if [[ $num =~ [^0-9] ]] ;then
    echo "input error"
elif [[ $num -eq 0 ]] ;then
    echo "input error"
else
	# 使用sed形成一个数组
    for i in `seq 1 $num` ;do
    # 加减乘除需要这样写，否则是拼接
        sum=$[$sum+$i]
    done
    echo $sum
fi
```

如果不想用sed，可以这样写：

```bash
#求出（1+2+...+n）的总和
#!/bin/bash
sum=0
read -p "Please input a positive integer: " num
if [[ $num =~ [^0-9] ]] ;then
    echo "input error"
elif [[ $num -eq 0 ]] ;then
    echo "input error"
else
 	for (( i=0; i<$num; i++ ));do
        sum=$[$sum+$i]
	done
    echo $sum
fi
```

