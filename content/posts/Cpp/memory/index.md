+++
date = '2025-04-03T09:54:30+08:00'
title = 'C++内存管理'
+++
# 内存管理
在现代c++中，应尽量避免底层内存操作，而使用现代结构，如容器和智能指针。

局部变量在栈上自动分配，动态变量在自由存储区分配。

## new 和 delete 关键字
```cpp
void leaky()
{
	new int;	//BUG! leaks memory
}
```
可以在nullptr指针上调用delete关键字，它不会做任何事情。

## malloc函数
new 关键字不仅分配内存，还构造对象。使用free()时，不会调用对象的析构函数。使用delete时，将调用析构函数恰当的清理对象。

```cpp
Foo *myFoo { (Foo*)malloc(sizeof(Foo)) };
Foo *myOtherFoo { new Foo() };
```

## 内存分配失败时

int *ptr { new(nothrow) int };

## 自由存储区数组的初始化
```
int *myArrayPtr { new int[] { 1, 2, 3, 4, 5} };
delete [] myArrayPtr;
```

realloc()这个函数不要使用，很危险；

## 多维数组
### 栈上的多维数组
3*3的多维数组，在计算机中，将以一维数组表示。数组的大小是所有维度的乘积。
### 堆上的多维数组
堆上的多维数组是不连续的。
```
char **board { new char[i][j] };	//Bug，编译不通过
```

```
char ** allocateBoard(size_t xDimension, size_t yDimension)
{
	char **myArray { new char*[xDimension] };
	for (size_t i = 0; size_t < xDimension; ++size_t) {
		myArray[i] = new char[yDimension];
	}
	return myArray;
}

void releaseBoard(char **&myArray, size_t xDimension)
{
	for (size_t i = 0; size_t < xDimension; ++i) {
		delete [] myArray[i];
		myArray[i] = nullptr;
	}
	delete [] myArray;
	myArray = nullptr;
}
```
## 数组和指针的对偶性

## 智能指针
```
unique_ptr<Simple> ptr = make_unique<Simple>();
ptr.reset(make_unique<Simple>());	// 销毁之前的指针，并改变所有权
auto raw_ptr = ptr.release();		// 释放所有权，但不销毁资源

```

自定义析构操作
```
void close(FILE* filePtr)
{
	if (filePtr == nullptr) { return; }
	fclose(filePtr);
	cout << "File closed." << endl;
}

int main()
{
	FILE* f { fopen("data.txt", "w") };
	shared_ptr<FILE> filePtr { f, close}
}
```
weak_ptr的使用
```
void useResource(weak_ptr<Simple> &weakSimple)
{
	auto resource { weakSimple.lock() };
	if (resource) {
		cout << "Resource still alive." << endl;
	} else {
		cout << "Resource has been freed." << endl;
	}
}

int main()
{
	auto sharedSimple { make_shared<Simple>() };
	weak_ptr<Simple> weakSimple { sharedSimple };
	
	useResource(weakSimple);
	
	sharedSimple.reset();
	
	useResource(weakSimple);
}
```


