---
layout: post
title: "素数筛"
date:   2024-03-30
tags: [algorithm]
comments: true
author: Gnonymous
---

~~~c++
#include <iostream>
#include <vector>

using namespace std;

vector<int> sieveOfEratosthenes(int n) {
    vector<bool> isPrime(n+1, true); // 初始化所有数为素数
    vector<int> primes; // 用于存储素数的向量

    // 从2开始遍历，直到根号n
    for (int p = 2; p * p <= n; p++) {
        // 如果 p 是素数
        if (isPrime[p]) {
            // 将 p 的倍数标记为非素数
            for (int i = p * p; i <= n; i += p)
                isPrime[i] = false;
        }
    }

    // 将剩余的素数添加到结果向量中
    for (int p = 2; p <= n; p++) {
        if (isPrime[p])
            primes.push_back(p);
    }

    return primes;
}

int main() {
    int n;
    cout << "Enter the upper limit: ";
    cin >> n;

    vector<int> primes = sieveOfEratosthenes(n);

    cout << "Prime numbers up to " << n << " are: ";
    for (int prime : primes) {
        cout << prime << " ";
    }
    cout << endl;

    return 0;
}

~~~

