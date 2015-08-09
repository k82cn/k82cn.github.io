---
layout: post
title: 475D – CGCDSSQ (Time limit exceeded on test 9)
---

##Problem Description:

Given a sequence of integers a1, …, an and q queries x1, …, xq on it. For each query xi you have to count the number of pairs (l, r) such that 1 ≤ l ≤ r ≤ n and gcd(al, al + 1, …, ar) = xi. is a greatest common divisor of v1, v2, …, vn, that is equal to a largest positive integer that divides all vi.

__Input__

The first line of the input contains integer n, (1 ≤ n ≤ 105), denoting the length of the sequence. The next line contains n space separated integers a1, …, an, (1 ≤ ai ≤ 109).

The third line of the input contains integer q, (1 ≤ q ≤ 3 × 105), denoting the number of queries. Then follows q lines, each contain an integer xi, (1 ≤ xi ≤ 109).

__Output__

For each query print the result in a separate line.

__Sample test(s)__

Input:

    3
    2 6 3
    5
    1
    2
    3
    4
    6

Output:

    1
    2
    2
    0
    1

## Solution:

The solution based on the following assumption:

    gcd(a1, a2, ..., an) = gcd(gcd(a1, a2, ..., aj), gcd(aj+1, aj+2, ..., an))

so all xi can be got by:

    |a1                     |a2             |a3          |a4|  ... |an-2               |an-1         |an|
    |gcd(a1, a2)            |gcd(a2, a3)    |gcd(a3, a4) |     ... |gcd(an-2, an-1)    |gcd(an-1, an)|
    |gcd(a1, a2, a3)        |gcd(a2, a3, a4)|                  ... |gcd(an-2, an-1, an)|
    ...
    |gcd(a1, a2, a3, ... an)|

Souce code of the solution:

    #include <map>
    #include <vector>
    #include <iostream>

    using namespace std;

    static int gcd(int a, int b)
    {
        if (a == 1 || b == 1) {
            return 1;
        }

        while (b != 0) {
            int r = b;
            b = a % b;
            a = r;
        }
        return a;
    }

    int main(int argc, char const *argv[])
    {
        int n, t;

        while (cin>>n) {
            vector v(n);
            map<int, int> db;

            for (int i = 0; i < n; ++i) {
                cin >> v[i];
                db[v[i]]++;
            }

            for (int i = n-1; i >=0; --i) {
                for (int j = 0; j < i; ++j) {
                    v[j] = gcd (v[j], v[j+1]);
                    db[v[j]]++;
                }
            }
            cin>> n;
            for (int i = 0; i < n; ++i) {
                cin >> t;
                cout << db[t] << endl;
            }
       }
       return 0;
    }
