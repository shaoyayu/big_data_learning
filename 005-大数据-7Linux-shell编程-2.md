# Linux的变量

## 作用域：

1. 本地
2. 局部
3. 位置
4. 特殊
5. 环境

## 本地变量

- 当前的shell拥有

- 生命周期随shell
- name=gob

```shell
[root@MDNode01 shell]# echo $$
1091
[root@MDNode01 shell]# syy=123
[root@MDNode01 shell]# echo $syy
123
[root@MDNode01 shell]# 
```

如果当前的进程结束以后，变量就会被销毁。



## 局部变量

- 只能local用于函数
- val = 10

```shell
[root@MDNode01 shell]# syy=123
[root@MDNode01 shell]# echo $syy
123
[root@MDNode01 shell]# fun(){
> echo $syy
> syy=12
> echo $syy
> local mgs=111
> echo $mgs
> }
[root@MDNode01 shell]# fun
123
12
111
[root@MDNode01 shell]# echo $syy
12
[root@MDNode01 shell]# echo $mgs

[root@MDNode01 shell]# 
```


可以看出在函数fun()中修改syy的值会对本地变量产生影响，

## 位置

- $1,$2,${11}
- 脚本
- 函数