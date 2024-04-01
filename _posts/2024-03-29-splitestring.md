---
layout: post
title: "c++下实现按照特定字符分割字符串"
date:   2024-03-29
tags: [algorithm]
comments: true
author: Gnonymous
---

#### 高级语言中有能直接按照特定字符串分割的函数，那么在C++中如何实现呢？

思路是将string变为输入流string stream，后利用getline(输入流，存储变量，结束符（默认为空格）)，进行分割。

~~~c++
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

using namespace std;

vector<string> splitString(const string& str, char delimiter) {
    vector<string> tokens; // 保存分割后的单词
    stringstream ss(str); // 使用字符串流分割字符串
    string token; // 临时保存单词的字符串

    // 使用 getline() 函数和指定的分隔符分割字符串
    while (getline(ss, token, delimiter)) {
        tokens.push_back(token); // 将单词保存到向量中
    }

    return tokens; // 返回保存所有单词的向量
}

int main() {
    string input = "Hello,World!This,is,a,test."; // 输入的字符串
    char delimiter = ','; // 指定分隔符为逗号

    // 分割字符串
    vector<string> words = splitString(input, delimiter);

    // 打印分割后的单词
    cout << "分割后的单词：" << endl;
    for (const auto& word : words) {
        cout << word << endl;
    }

    return 0;
}
~~~

to_lowercase
