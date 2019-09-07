##	C++ 智能指针 笔记

*	四种智能指针

	auto_ptr, shared_ptr, weak_ptr, unique_ptr

	第一个已经被c++11弃用

	后三个是c++11支持

*	为什么

	c++ 需要手动进行内存管理, new --- delete

	但是我们不能避免程序还未执行到 delete 时就跳转了或者在函数中没有执行到最后的 delete 语句就返回了

	如果我们不在每一个可能跳转或者返回的语句前释放资源, 就会造成内存泄露 memory leak

	使用智能指针可以很大程度上的避免这个问题, 因为`智能指针就是一个类`, 当超出了类的`作用域`时, 类会`自动调用析构函数`, 析构函数会自动`释放资源`

	<br>

##	`auto_ptr`

*	`auto_ptr` 成员函数

	`get()`, `reset()`, `release()`

*	`auto_ptr` 用法

	```cpp
	class Test{
	public:
		Test(string s){
			str = s;
			cout<<"create Test: "<<str<<'\n';
		}
		~Test(){
			cout<<"delete Test: "<<str<<'\n';
		}
		void print(){
			cout<<str<<endl;
		}
	private:
		string str;
	};

	int main(){
		auto_ptr<Test> p1(new Test("123"));    // 输出 create Test: 123

		p1->print();    // 输出 123

		// 注意 auto_ptr 像一个实例对象一样可以用 . 来访问自己的成员函数 get()
		p1.get()->print();    // 输出 123

		// 注意 auto_ptr 像一个实例对象一样可以用 . 来访问自己的成员函数 reset()
		p1.reset(new Test("hhh"));    // 先输出 create Test: hhh
		// 然后输出 delete Test: 123
		// 表明原来的 123 对象的内存已经被释放了

		p1->print();    // 输出 hhh

		// 最后在退出时, 由于超出了类的作用域, auto_ptr 会自动调用析构函数释放资源
		// 因此还会输出 delete Test: hhh
		return 0;
	}
	```

*	`auto_ptr` 赋值操作

	当进行 p2 = p1 操作时, p2 会接管 p1 原来的内存管理权, p1 会变为空指针, 如果 p2 原来不为空, 则它会释放原来的资源

	基于这个原因, 应该避免把 `auto_ptr` 放到容器中, 因为算法对容器操作时, 很难避免 STL 内部对容器实现了赋值传递操作, 这样会使容器中很多元素被置为 NULL, 判断一个智能指针是否为空不能使用 `if(p1 == NULL)`, 应该使用 `if(p1.get() == NULL)`

	```cpp
	int main(){
		auto_ptr<Test> p1(new Test("123"));    // 输出 create Test: 123
		auto_ptr<Test> p2(new Test("456"));    // 输出 create Test: 456
		p2 = p1;    // 输出 delete Test: 456, 释放 p2 原来管理的资源
		p2->print();    // 输出 123
		if(p1.get()==NULL){
			// 此时 p1 已经指向 NULL
			cout << "p1 is NULL\n";
		}
		// 析构, 输出 delete Test: 123
		return 0;
	}
	```

*	release

	还有一个值得我们注意的成员函数是 release, 这个函数只是把智能指针赋值为空, 但是它原来指向的内存并没有被释放, 相当于它只是释放了对资源的所有权

	```cpp
	int main(){
		auto_ptr<Test> p1(new Test("123"));
		p1.release();
		if (p1.get() != NULL){
			// 不会进入, 因为此时 p1 已经指向 NULL
			p1.get()->print();
		}
		// 值得注意的是, 最后不会输出 delete Test: 123
		// 也就是说, 析构函数没有被调用
		// 因为此时 123 这个对象已经不归 p1 管控了
		return 0;
	}
	```

*	为什么抛弃 auto_ptr

	调用拷贝构造函数或者赋值函数后, 原有指针会被置 NULL, 所以 C++ 才建议使用 shared_ptr

	<br>

##	参考资料

*	[c++ 智能指针用法详解, JustDoIT](https://www.cnblogs.com/TenosDoIt/p/3456704.html)

*	[selfboot/CS_Offer/C++/11_SmartPoint.md](https://github.com/selfboot/CS_Offer/blob/master/C%2B%2B/11_SmartPoint.md)

*	[C++智能指针简单剖析](https://www.cnblogs.com/lanxuezaipiao/p/4132096.html)

*	[如何理解智能指针？](https://www.zhihu.com/question/20368881)

*	[auto_ptr的缺陷在哪里？为什么不应该用？](https://www.zhihu.com/question/37351146)

*	[auto_ptr, cplusplus](http://www.cplusplus.com/reference/memory/auto_ptr/)