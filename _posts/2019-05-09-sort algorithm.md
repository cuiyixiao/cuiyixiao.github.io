# 瞎整理排序算法

-  ##### 归并排序

```cpp
#include<iostream>
//归并排序

int assist[105];

void merge_sort(int* a, int left, int right)
{
	if(left == right)
	{
		return;
	}
	int mid = (left + right) / 2;
	merge_sort(a, left, mid);
	merge_sort(a, mid + 1, right);
	int x = left;
	int y = mid + 1;
	int loc = left;
	while (x <= mid || y <= right)
	{
		if (x <= mid && (y > right || a[x] <= a[y]))
		{
			assist[loc] = a[x];
			x++;
		}
		else
		{
			assist[loc] = a[y];
			y++;
		}
		loc++;
	}
	for (int i = left; i <= right; i++)
	{
		a[i] = assist[i];
	}
}
```
- ##### 快排

```cpp
#include<iostream>
//快速排序

void quick_sort(int* a, int left, int right)
{
	if (left > right)
	{
		return;
	}
	int flag = a[left];
	int low = left;
	int high = right;
	while (low < high)
	{
		while (low < high && a[high] >= flag)
		{
			high--;
		}
		a[low] = a[high];
		while (low < high&&a[low] <= flag)
		{
			low++;
		}
		a[high] = a[low];
	}
	a[low] = flag;
	quick_sort(a, left, low - 1);
	quick_sort(a, low + 1, right);
}
```

- ##### 堆排

```cpp
#include<iostream>
//大根堆排序

class Heap
{
private:
	int *data,size;
public:
	Heap(int length)
	{
		data = new int[length];
		size = 0;
	}
	~Heap()
	{
		delete[] data;
	}
	void push(int value);
	void update(int pos, int n);
	void heap_sort();
	void output();
};

void Heap::heap_sort()
{
	for (int i = size-1; i >= 0; i--)
	{
		std::swap(data[i], data[0]);
		update(0, i);
	}
}

void Heap::output()
{
	for (int i = 0; i < size - 1; i++)
	{
		std::cout << data[i] << std::endl;
	}
}

void Heap::push(int value)
{
	data[size] = value;
	int cur = size;
	int father = (cur - 1) / 2;
	while (data[cur] > data[father])
	{
		std::swap(data[father], data[cur]);
		cur = father;
		father = (cur - 1) / 2;
	}
	size++;
}

void Heap::update(int pos, int n)
{
	int lchild = 2 * pos + 1;
	int rchild = 2 * pos + 2;
	int cur = pos;
	if (lchild < n && data[lchild] > data[cur])
	{
		cur = lchild;
	}
	if (rchild < n && data[rchild] > data[cur])
	{
		cur = rchild;
	}
	if (cur != pos)
	{
		std::swap(data[pos], data[cur]);
		update(cur, n);
	}
}
```

- ##### 多路归并

```cpp
#include<iostream>
#include<vector>
//多路归并

class kwaymerge
{
public:
	static kwaymerge* Instance();
	kwaymerge(){}
	void Init(std::vector<std::vector<int> > num)
	{
		m_number = num; //假设为读入的数据
	}
	void sort()
	{
		for (int i = 0; i < k; i++)
		{
			read(i, base[i], 0); //把数据读出来，并且加到败者树叶节点中
		}
		createlosertree();  //创建败者树
		while (base[lose[0]] != MAXKEY)              //MAXKEY其实即败者树最大容量，可以理解最多分了几路
		{
			int i = lose[0];              
			sort_number.push_back(base[i]);  //将结果存起来
			pos[i]++;                       //非叶节点+1
			read(i, base[i], pos[i]);      //这个与上一步合起来就是代表当前败者已经排好，从这个败者所在的文件里取出第二个树加入败者树
			update(i);                      //加入新的数后，败者树更新操作
		}
	}
	void print()
	{
		for (int i = 0; i < sort_number.size(); i++)
		{
			std::cout << sort_number[i] << " ";
			std::cout << std::endl;
		}
	} 
private:
	void read(int i, int &base, int pos)
	{
		base = m_number[i][pos];    //read 可以理解为多路归并里的read file ，为了测试写成这样
	}
	void createlosertree()    //创建败者树
	{
		base[k] = MINKEY;
		for (int i = 0; i < k; i++)
		{
			lose[i] = k;
		}
		for (int i = k - 1; i >= 0; --i)
		{
			update(i); //每加入一个节点不断的更新
		}
	}
	void update(int i)
	{
		int t = (i + k) / 2;
		while (t > 0)
		{
			if (base[i] > base[lose[t]])
			{
				std::swap(i, lose[t]);
			}
			t = t / 2;
		}
		lose[0] = i;
	}//败者树更新，不断和父节点进行比较
private:
	static const int k = 4;
	static const int MINKEY = -1;
	static const int MAXKEY = 100;
	int base[k + 1]; // 败者树叶节点
	int lose[k]; //败者树其余节点
	std::vector<std::vector<int> >m_number;
	int pos[4] = {0,0,0,0};
	std::vector<int> sort_number;
	static kwaymerge* instance;
};
kwaymerge* kwaymerge::instance = new kwaymerge();
kwaymerge* kwaymerge::Instance()
{
	return instance;
}
/*
int main()
{
	std::vector<int>v1;
	std::vector<int>v2;
	std::vector<int>v3;
	std::vector<int>v4;
	v1.push_back(1);
	v1.push_back(5);
	v1.push_back(7);
	v1.push_back(100);
	v2.push_back(3);
	v2.push_back(8);
	v2.push_back(9);
	v2.push_back(100);
	v3.push_back(2);
	v3.push_back(4);
	v3.push_back(6);
	v3.push_back(100);
	v4.push_back(0);
	v4.push_back(10);
	v4.push_back(100);
	std::vector<std::vector<int> >test;
	test.push_back(v1);
	test.push_back(v2);
	test.push_back(v3);
	test.push_back(v4);
	kwaymerge::Instance()->Init(test);
	kwaymerge::Instance()->sort();
	kwaymerge::Instance()->print();
	getchar();
	return 0;
}
//https://www.cnblogs.com/iyjhabc/p/3141665.html
//败者树讲解
//可应用于数据多的情况下，无法全部载入到内存，采取外排序
*/
```

ps ：前三个比较熟悉，不详细记录，主要写一下第四个k路归并，主要应用于，比如一个大文件里很多数据，无法全部载入到内存中进行处理这种情况，多路归并利用了以中叫败者树的算法，继续说如何解决的上述问题，首先，将大文件分割成有序的小文件，甚至可以用多线程去处理，分完后就是用多路归并算法，也叫外排序，首先我们从每个文件里读出第一个数据，可以来组成败者树，之后每次得到的败者，我们将他从该文件中去除，之后再拿出一个新的数字顶替上来，更新败者树，最后可以得到有序的数据，详细见代码又明确的注释。
