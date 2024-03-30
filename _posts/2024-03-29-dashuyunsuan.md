---
layout: post
title: "大数运算——加减乘除"
date:   2024-03-29
tags: [algorithm]
comments: true
author: Gnonymous
---



#### 大数运算

转化为字符串后处理（细节后续更新）

~~~c++
#include <iostream>
#include <string>
#include <algorithm> // 使用 reverse() 函数

using namespace std;

// 函数用于执行大数加法
string add(string num1, string num2) {
    string result;
    int carry = 0; // 进位标志

    // 将两个字符串反转，从个位开始相加
    reverse(num1.begin(), num1.end());
    reverse(num2.begin(), num2.end());

    // 对两个字符串进行相加，直到其中一个字符串的所有位数都相加完毕
    size_t len = max(num1.size(), num2.size());
    for (size_t i = 0; i < len; ++i) {
        int n1 = (i < num1.size() ? num1[i] - '0' : 0); // 获取 num1 中的当前数字
        int n2 = (i < num2.size() ? num2[i] - '0' : 0); // 获取 num2 中的当前数字
        int sum = n1 + n2 + carry; // 计算当前位的和
        result.push_back(sum % 10 + '0'); // 将当前位的结果添加到结果字符串中
        carry = sum / 10; // 更新进位标志
    }

    // 如果最高位还有进位，需要额外添加一个进位位
    if (carry > 0) {
        result.push_back(carry + '0');
    }

    // 反转结果字符串，得到最终的加法结果
    reverse(result.begin(), result.end());

    return result;
}

// 函数用于执行大数减法
string subtract(string num1, string num2) {
    string result;
    int borrow = 0; // 借位标志

    // 将两个字符串反转，从个位开始相减
    reverse(num1.begin(), num1.end());
    reverse(num2.begin(), num2.end());

    // 对两个字符串进行相减，直到其中一个字符串的所有位数都相减完毕
    size_t len = max(num1.size(), num2.size());
    for (size_t i = 0; i < len; ++i) {
        int n1 = (i < num1.size() ? num1[i] - '0' : 0); // 获取 num1 中的当前数字
        int n2 = (i < num2.size() ? num2[i] - '0' : 0); // 获取 num2 中的当前数字
        int diff = n1 - n2 - borrow; // 计算当前位的差值
        if (diff < 0) { // 如果当前位小于 0，则需要向高位借位
            diff += 10;
            borrow = 1;
        } else {
            borrow = 0;
        }
        result.push_back(diff + '0'); // 将当前位的结果添加到结果字符串中
    }

    // 去除结果字符串开头的多余的 0
    while (result.size() > 1 && result.back() == '0') {
        result.pop_back();
    }

    // 反转结果字符串，得到最终的减法结果
    reverse(result.begin(), result.end());

    return result;
}

// 函数用于执行大数乘法
string multiply(string num1, string num2) {
    string result(num1.size() + num2.size(), '0'); // 结果字符串初始化为全 0
    for (int i = num1.size() - 1; i >= 0; --i) {
        int carry = 0;
        for (int j = num2.size() - 1; j >= 0; --j) {
            int temp = (num1[i] - '0') * (num2[j] - '0') + (result[i + j + 1] - '0') + carry;
            result[i + j + 1] = temp % 10 + '0';
            carry = temp / 10;
        }
        result[i] += carry;
    }
    size_t pos = result.find_first_not_of('0');
    if (pos != string::npos) {
        return result.substr(pos);
    }
    return "0";
}

// 函数用于执行大数除法
string divide(string dividend, string divisor) {
    string quotient;
    string remainder = dividend;
    while (remainder.size() >= divisor.size()) {
        int factor = 0;
        while (compare(remainder, multiply(divisor, to_string(factor + 1))) >= 0) {
            ++factor;
        }
        quotient.push_back(factor + '0');
        remainder = subtract(remainder, multiply(divisor, to_string(factor)));
    }
    return quotient;
}

// 比较两个大数的大小
int compare(string num1, string num2) {
    if (num1.size() < num2.size()) {
        return -1;
    } else if (num1.size() > num2.size()) {
        return 1;
    } else {
        return num1.compare(num2);
    }
}

int main() {
    string num1 = "123456789012345678901234567890";
    string num2 = "987654321098765432109876543210";

    // 执行大数加法
    string sum = add(num1, num2);
    cout << "大数加法结果：" << sum << endl;

    // 执行大数减法
    string difference = subtract(num1, num2);
    cout << "大数减法结果：" << difference << endl;

    // 执行大数乘法
    string product = multiply(num1, num2);
    cout << "大数乘法结果：" << product << endl;

    // 执行大数除法
    string quotient = divide(num1, num2);
    cout << "大数除法结果：" << quotient << endl;

    return 0;
}
~~~

