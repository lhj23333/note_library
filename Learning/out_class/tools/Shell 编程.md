### 1. 变量

#### 1.1 变量命名规则
- 只包含字母、数字和下划线
- 不能以数字开头
- 避免使用 `shell` 关键字：`if` 、`then` 、`if`、`else` .....
- 使用大小写表示常量
- 避免使用特殊符号：@、!、#、$ .....
- 避免使用空格：空格通常用于分割命令和参数

#### 1.2 变量类型
- 字符型变量：

``` shell
str_name="xxxx" 	# 双引号里可以有变量和转义字符
str_name='xxxx'		# 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
```

- 整数变量：

``` shell
declare -i num_integr=10
```

- 数组变量：

```shell
num_arry=(1 2 3 4 5 6)
```

或者关联数组：

```shell
declare -A associative_arry
associative_arry["name"]="John"
associative_arry["age"]=30
```

- 环境变量：

```shell
$PATH
```


#### 1.3 变量的使用：

使用一个定义过的变量时，通过在变量名前面添加美元符号 `$` 来进行调用，变量名外添加花括号（可选，目的是为了帮助解释器识别变量边界）

- 打印变量：

```shell
echo ${variable_name}
```

- 只读变量：
	- 使用 `readonly` 命令将变量设为只读变量，只读变量的值不能被改变

```shell
readonly variable_name 
```

- 删除变量
	- 变量被删除后不能被再次使用，且 `unset` 命令不能删除只读变量

```shell
unset variable_name
```

- 字符串拼接：
