+++
date = '2025-04-03T09:54:30+08:00'
title = 'String字符串使用简录'
+++
# string 记录
## c风格字符串的缺陷

c风格字符串的缺陷：不检查边界，不会自动扩展缓冲区。
### 不检查边界
``` cpp
char buffer[10];  				// 仅能存放 9 个字符 + '\0'
strcpy(buffer, "This is a very long string!");  // 溢出！  
printf("%s\n", buffer);  			// 可能导致未定义行为
return 0;
```

缺少边界检查
``` cpp
char buffer[8];
gets(buffer);					// 用户的输入可能超过8个字符
fgets(buffer, sizeof(buffer), stdin);
```

### 长度陷阱
``` cpp
#include <cstring>
char *copyString(const char *str)
{
	char *result { new char[strlen(str)] };	//strlen返回的字符串的长度没有包含结尾的'\0'
	strcpy(result, str);	//Bug! Off by one!
	return result;
}
```

### char* 和 char[] 的差异
不要通过C风格的字符串获取长度。
第二种写法就会出现警告ISO C++11 does not allow conversion from string literal to 'char *'。
``` cpp
char text[] {"abcdefabcdefabcdef"};
std::cout << "sizeof(): " << sizeof(text) << std::endl;		// 输出数组大小
std::cout << "strlen(): " << strlen(text) << std::endl;

char *text_1 {"abcdefabcdefabcdef"};
std::cout << "sizeof(): " << sizeof(text_1) << std::endl;	// 输出指针大小
std::cout << "strlen(): " << strlen(text_1) << std::endl;
```

字符串字面量，字符串字面量存储在只读的数据段（Read-Only Data Segment），通常不可修改。编译器可重用等效字符串字面量的引用，从而优化内存的使用。
只读变量存储区位于 .rodata 中。

| 写法 			| 存储位置	| 可修改?	| 推荐 |
| ---			| ---           | ---           | --- |
|char* p = "Hello"; 	| 只读数据段	| ❌ 不可修改	| ❌ 不推荐 |
|const char* p = "Hello";|只读数据段	| ❌ 不可修改	| ✅ 推荐 |
|char arr[] = "Hello";	| 栈上（本地数组）|✅ 可修改	| ✅ 推荐（如需修改）|

``` cpp
char *ptr = {"hello"};
ptr[1] = 'a';			// Undefined behavior

// 比较安全的做法
const char *ptr {"hello"}
ptr[1] = 'a';			// Error! Write to read-only memory
```

放在数组中，编译器会创建一个新的字符串数组，并将只读区域的字符串值复制到字符串数组中
``` cpp
char arr[] = {"hello"};
arr[1] = 'a';			// Ok
```

### 原始字符串常量
raw string literal
``` cpp
const char* str {"Hello \"World\""};
const char* str {R"(Hello "World")"};
const char* str {R"(Line 1
Line 2)" };
```
## std::string 细节
### 三向比较运算符
``` cpp
#include <compare>
#include <iostream>
int main() {
    int a = 5, b = 10;
    auto result = a <=> b;

    if (result < 0) std::cout << "a < b\n";
    else if (result > 0) std::cout << "a > b\n";
    else std::cout << "a == b\n";
}
```

### 获取C风格的字符串
此种方式获取到的C风格的字符串是需要立即被使用的。
``` cpp
const char *c = str.c_str();
```

此种方式返回的也是危险行为
``` cpp
const char* getString() {
    std::string str = "hello world";
    return str.c_str(); // 返回的是一个指向局部变量内容的指针
}
```

### 标准字面量s
``` cpp
auto s = {"This is const char *"};
auto str {"This is const char *"s};
std::cout << "tpye: " << typeid(s).name() << std::endl;
std::cout << "type: " << typeid(str).name() << std::endl;
```

命名空间使用的是内联函数
``` cpp
namespace std {
	inline namespace literals {
		inline namespace string_literals {
			// ...
		}
	}
}
```

### 类型推导陷阱
vector会被推导为 const char[],而不是 string
``` cpp
vector names{"Jon", "Sam", "Sue"};
vector names_string {"Jon"s, "Sam"s, "Sue"s };
```

### 数值转化
``` cpp
string val_str = to_string(4.12)
```

to_chars,from_chars需要自己指定string大小，但是有更高的效率
```cpp
const size_t BufferSize { 50 };
std::string out(BufferSize, ' ');
auto result { std::to_chars(out.data(), out.data() + out.size(), 12345) };
if (result.ec == std::errc()) { std::cout << out << std::endl; }

double value1 { 0.314 };
std::string out_val(BufferSize, ' ');
auto [ptr1, error] = std::to_chars(out_val.data(), out_val.data() + out_val.size(), value1);

if (error == std::errc()) {
	double value2;
	auto [ptr2, error2] = std::from_chars(out_val.data(), out_val.data() + out.size(), value2);
        if (error2 == std::errc()) {
            if (value1 == value2) {
                std::cout << "Perfect roundtrip" << std::endl;
            }
        }
}
```

## std::string_view
使用场景：函数型参类型，const char * 或者 const std::string &
如果传入的参数是字面值常量，字面值常量会被转化为 const std::string &，会增加一些额外的开销。

string_view使用的一些陷阱
std::string("temp.txt")是一个临时变量，语句执行完成后，便会被释放。ext指向的便是一个空悬指针。
``` cpp
auto ext = extractExtension(std::string("temp.txt"));
```

str + "World"其实也是产生了一个临时变量
``` cpp
std::string str{"hello"};
std::string_view sv { str + "World" };
```
