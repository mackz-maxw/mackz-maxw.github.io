---
title: 算法笔试 | acm模式输入输出指南
categories: interview
lang: zh-CN
---

使用c++作为笔试语言
```cpp
#include <iostream>
#include <vector>
#include <string>
#include <sstream>
#include <algorithm>
#include <climits>
#include <cmath>
using namespace std;

int main() {
    // ====== 基础输入输出 ======

    // 读取多个整数
    int a, b, c;
    cout << "输入三个整数(用空格分隔): ";
    cin >> a >> b >> c;
    cout << "三个整数分别是: " << a << ", " << b << ", " << c << endl;
    
    // 清除输入缓冲区
    cin.ignore(INT_MAX, '\n');
    
    // ====== 读取整行输入 ======
    cout << "\n整行输入示例:" << endl;
    string line;
    cout << "输入一行文本: ";
    getline(cin, line);
    cout << "你输入的文本是: " << line << endl;
    
    // ====== 处理大数字 ======
    cout << "\n大数字处理示例:" << endl;
    
    // 32位大整数 (使用long)
    long big32;
    cout << "输入一个32位大整数: ";
    cin >> big32;
    cout << "32位大整数: " << big32 << endl;
    
    // 64位大整数 (使用long long)
    long long big64;
    cout << "输入一个64位大整数: ";
    cin >> big64;
    cout << "64位大整数: " << big64 << endl;
    
    cin.ignore(INT_MAX, '\n');
    
    // ====== 字符串流处理 ======
    cout << "\n字符串流处理示例:" << endl;
    cout << "输入多个用空格分隔的整数: ";
    getline(cin, line);  // 使用逗号作为分隔符：getline(ss, line, ',')
    stringstream ss(line);
    vector<int> numbers;
    int num;
    while (ss >> num) {
        numbers.push_back(num);
    }
    cout << "提取的数字: ";
    for (int n : numbers) {
        cout << n << " ";
    }
    cout << endl;
    
    // ====== 多组输入数据 ======
    cout << "\n多组输入数据示例:" << endl;
    cout << "输入多行数据，每行两个整数(输入0 0结束):" << endl;
    int x, y;
    while (cin >> x >> y) {
        if (x == 0 && y == 0) break;
        cout << "和: " << (x + y) << endl;
    }
    
    // 清除状态并忽略剩余内容
    cin.clear();
    cin.ignore(INT_MAX, '\n');
    
    // ====== 文件结束处理 ======
    cout << "\n文件结束处理示例(输入Ctrl+Z或Ctrl+D结束):" << endl;
    cout << "输入多个整数:" << endl;
    vector<int> eofNumbers;
    int input;
    while (cin >> input) {
        eofNumbers.push_back(input);
    }
    
    cin.clear();
    cin.ignore(INT_MAX, '\n');
    
    cout << "输入的数字: ";
    for (int n : eofNumbers) {
        cout << n << " ";
    }
    cout << endl;
    
    // ====== 格式化输出 ======
    cout << "\n格式化输出示例:" << endl;
    double pi = 3.141592653589793;
    cout << "默认输出: " << pi << endl;
    cout.precision(4);
    cout << "保留4位: " << pi << endl;
    cout.precision(10);
    cout << "保留10位: " << pi << endl;
    
    // 恢复默认精度
    cout.precision(6);
    
    // ====== 实战示例 ======
    cout << "\n实战示例: 计算一系列数字的平均值" << endl;
    cout << "输入多个数字(用空格分隔): ";
    getline(cin, line);
    stringstream ss2(line);
    double sum = 0;
    int count = 0;
    while (ss2 >> num) {
        sum += num;
        count++;
    }
    if (count > 0) {
        cout << "平均值: " << (sum / count) << endl;
    } else {
        cout << "没有输入数字" << endl;
    }
    
    return 0;
}
```