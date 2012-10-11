---
layout: post
category: test
---
{% include JB/setup %}

This is my first blog article.
你好，Github Page

##Code
	#include <iostream>
	#include <cstdio>

	int main(int argc, char *argv[]) {
		int b = 0;
		auto func = [&b](int a) -> void {std::cout<<a<<" "<<b++<<std::endl;};
		func(1);
		func(10);
		b++;
		func(0);
		return 0;
	}
