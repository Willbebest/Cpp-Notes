####  函数printf 的实现

***

**函数声明**

```c
#include <stdio.h>
int printf(const char* fmt, ...)
```

参数:

- 第一个参数: `fmt` 是一个字符串指针, 包含格式占位符
- 第二个参数: `...`是可变参数. 是占位符的实际数值 

返回值:
- 	当格式化字符串输出成功时，返回格式化字符的个数，不包括结尾得‘\0’
- 	如果格式化字符串输出错误，返回一个负值

***

#### printf 的具体实现
```c
int printf(const char*fmt, ...)
{
	char printf_buf[1024];
	va_list args;
	int printed;
	
	va_start(args, fmt);
	printed =vsprintf(printf_buf, fmt, args);
	va_end(args);
	
	puts(printf_buf);
	
	return printed;
}
```

逐行分析：

- 第一行是函数头
- 第三行 `printf_buf` 是一个大小为1024 的字符数组，用于存储输出结果。
- 第四行定义一个 `va_list` 类型的变量 `args`，是一个void类型指针，用于指向可变参数的列表
- 第五行`printed`用于记录返回格式化字符的个数
- 第七行 使用 `va_start` 初始化 `args` ，根据第一个参数 `fmt`，获取可变参数的地址传递给args
-  第八行 调用 `vsprintf`分析 `fmt` 中的格式化占位符，并使用可变参数替换格式化占位符，结果存储在 `printf_buf`。返回值为输出字符的个数。
- 第九行 使用 `va_end` 释放`args`，将其初始化为 0
- 第十一行 输出 `printf_buf` 中的内容

***

#### vsprintf函数原型：

```c
#include <stdio.h>
int vsprintf(char* buf, const char* fmt, va_list args);
```

在 vsprintf 有三个参数，第一个是字符数组，用于接收要输出的类型。第二个参数是 printf 的第一个参数。第三个参数是指向可变参数的指针。返回值为第一个参数中字符的个数。

vsprintf 通过遍历 fmt 来判断确认参数个数和类型。如果不是%，就添加到缓存中（第一个参数）。如果是%则跳过判断其后的字符：
- 如果是数字则代表宽度，如果是\*，则通过`va_args`获取可变参数数据，则个数据代表宽度。
- 然后在查看宽度后面，如果是"."，则后边的可能存在代表精度的字符。如果数字或\*，则存在精度。"\*" 需要通过 va_arg 获取精度数字。
- 然后再获取继续遍历，字符代表的是数据的类型以及进制。
- 然后返回第一步继续判断。

总结：printf函数借助于 va_list 类型变量和三个宏 va_start、va_args、以及 va_end 进行实现。

>va_list 是一个指针类型。用于标记可变参数
>va_start 两个参数：va_lis t和 const char*或void*。作用是通过 printf 的第一个参数，获取可变参数的首地址。
>va_arg有两个参数：va_list类类型变量和参数类型。作用是获取可变参数，并移动指针指向下一个参数。
>va_end有一个参数是va_list类型。用于置空。

***

实现一个可变参数函数 sum，计算多个数的和：

```c
long sum(long num, ...)
{
	va_list ap;
	long sum = 0;
	
	va_start(ap, num);
	while(num--) {
	sum += va_arg(ap, long);
	}
	va_end(ap);
	
	return sum;
}
```


