##	C++ 智能指针 笔记

*	几种智能指针

	auto_ptr, unique_ptr, weak_ptr, shared_ptr, scoped_ptr

	第一个已经被c++11弃用

	后三个是c++11支持

*	> 从较浅的层面看, 智能指针是利用了一种叫做RAII（资源获取即初始化）的技术对普通的指针进行封装, 这使得智能指针实质是一个对象, 行为表现的却像一个指针。
	>
	> by [柴小喵](https://www.zhihu.com/question/20368881/answer/25835250)

*	基本作用/为什么

	应对两种情况: i) 忘记delete, ii) 程序出现异常的时候也可以回收内存

	c++ 需要手动进行内存管理, new --- delete

	但是我们不能避免程序还未执行到 delete 时就跳转了或者在函数中没有执行到最后的 delete 语句就返回了, 如果我们不在每一个可能跳转或者返回的语句前释放资源, 就会造成内存泄露 memory leak

	使用智能指针可以很大程度上的避免这个问题, 因为`智能指针就是一个类`, 当超出了类的`作用域`时, 类会`自动调用析构函数`, 析构函数会自动`释放资源`

*	其他作用

	TODO: 理解详情

	value 语义转化为 reference 语义

*	所有权

	> 对于特定的对象, 只能有一个智能指针可拥有, 这样只有拥有对象的智能指针的构造函数会删除该对象。然后让赋值操作转让所有权。这就是用于 `auto_ptr` 和 `unique_ptr` 的策略, 但 `unique_ptr` 的策略更严格。
	>
	> [Alexia](https://www.cnblogs.com/lanxuezaipiao/p/4132096.html)

	下面会有例子, 简单理解就是 p2 = p1 后, p1 变成空指针, 防止两个指针共同管理一个对象的情况。如果此时再用 p1->foo(), 那么 auto_ptr 会在运行时报错, 而 unique_ptr 将在编译时报错

*	引用计数

	与 auto_ptr/unique_ptr 不同, shared_ptr 可以多个指针共同管理一个对象, 但是如何避免刚才的问题呢？

	首先, 智能指针不需要手动 delete

	> ... 引入了引用计数的概念。...在还没有被开辟的时候, 该块内存的引用计数为0, 在第一次用p1开辟的时候引用计数+1变成1, 如果有其他指针也需要这块内存, 比如一个潜copy操作, 比如p2, 那么这时候就有两个指针指向m1, 引用计数变成2, 当p1用完了, 就用一个操作切断p1和m1的关系, m1的引用计数变成1。当p2也用完了, 那么通过一个操作引用计数再次减去 1, 引用计数变成 0. 当智能指针发现它管理的内存的引用计数变成 0 的时候, 对 m1 做一个 delete 操作, 使之释放。
	> 
	> [郑斌](https://www.zhihu.com/question/20368881/answer/14918243)

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

	调用拷贝构造函数或者赋值函数后, 原有指针会被置 NULL

	> ... 这跟通常理解的 copy 行为是不一致的, 而这样的行为要有些场合下不是我们希望看到的...
	> 例如参考《Effective STL》第8条, sort的快排实现中有将元素复制到某个局部临时对象中, 但对于auto_ptr, 却将原元素置为null, 这就导致最后的排序结果中可能有大量的null
	> ... 这也就是用 unique_ptr 代替 auto_ptr 的原因, 本质上来说, 就是 unique_ptr 禁用了 copy, 而用 move 替代
	>
	> [Sen Zhang](https://www.zhihu.com/question/37351146/answer/83379043)

	> 总而言之，auto_ptr 最大的弊端在于允许很多几乎没有实际用处又不符合常规认知的行为，用 scoped_ptr 和 unique_ptr 可以利用静态检查早早地指出这些无意义操作的存在。
	>
	> [丁冬](https://www.zhihu.com/question/37351146/answer/71576902)

*	TODO

	其他智能指针的用法和特点

	<br>

##	参考资料

*	[c++ 智能指针用法详解, JustDoIT](https://www.cnblogs.com/TenosDoIt/p/3456704.html)

*	[selfboot/CS_Offer/C++/11_SmartPoint.md](https://github.com/selfboot/CS_Offer/blob/master/C%2B%2B/11_SmartPoint.md)

*	[C++智能指针简单剖析](https://www.cnblogs.com/lanxuezaipiao/p/4132096.html)

*	[如何理解智能指针？](https://www.zhihu.com/question/20368881)

*	[auto_ptr的缺陷在哪里？为什么不应该用？](https://www.zhihu.com/question/37351146)

*	[auto_ptr, cplusplus](http://www.cplusplus.com/reference/memory/auto_ptr/)

*	[智能指针, wikipedia](https://zh.wikipedia.org/wiki/智能指针)