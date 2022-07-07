---
title: 三个线程顺序打印ABC
date: 2022-03-14 20:55:14
tags:
     - c++
categories: 
     - 多线程/多进程
     - 三个线程顺序打印ABC
comments: true
---

> ​                                                                  三个线程顺序打印ABC

<!-- more -->

```c++
#include<iostream>
#include<vector>
#include<thread>
#include<condition_variable>
using namespace std;
mutex m; //互斥锁
std::condition_variable cond_var; //创建条件变量对象
int num=0;
void func(char ch)
{
	int n=ch-'A';
	for(int i=0;i<10;i++)
	{
		std::unique_lock<std::mutex> mylock(m);
		cond_var.wait(mylock,[n]{ //调用wait函数，先解锁mylock，然后判断lambda的返回值
            return n==num;});
		cout<<ch;
		num=(num+1)%3;
		mylock.unlock();
		cond_var.notify_all(); //唤醒所有等待条件变量的线程。
	}
}
int main()
{
	vector<thread> pool;
	pool.push_back(thread(func,'A'));
	pool.push_back(thread(func,'B'));
	pool.push_back(thread(func,'C'));
	for(auto it = pool.begin(); it != pool.end(); it++)
	{
		it->join();
	}
	return 0;
}
/*
调用wait时，会执行以下步骤：1、对绑定的unique_lock对象解锁；2、判断调用对象的返回值。如果调用对象返回值为false，则wait就一直阻塞，一直等待被notify唤醒；如果调用对象为true，则会对绑定的unique_lock对象重新上锁，然后wait函数返回，继续执行后续的程序。当wait被notify唤醒时，会先重新对绑定的unique_lock对象上锁，然后执行上面的1、2步骤
*/
```

