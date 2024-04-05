---
layout:     post
title:      RSA计算过程及RSA密钥生成代码
subtitle:   
date:       2019-01-15
author:     ZSJ
header-img: img/post-bg-debug.png
catalog: true
tags:
    - C#
    - RSA
---
# RSA计算过程
### 1.	取两个质数p, q
p = 23, q = 29
### 2.	计算n = p*q = 23 * 29 = 667
667用二进制表示为1010011011 共十位，这个位数就是RSA中的位数
### 3.	计算n的欧拉函数φ(n) = (p-1)*(q-1) = 22 * 28 = 616
### 4.	随机选择整数e，1 < e <φ(n)，并且e与φ(n)互质，选择13，实际应用中常选择65537
### 5.	计算e对于φ(n)的模反元素d，(模反元素：如果两个正整数a和n互质，那么一定可以找到整数b，使得 ab-1 被n整除，或者说ab被n除的余数是1，这时，b就叫做a的模反元素)
即e * d 除以φ(n)余数为1,  e*d Mod (φ(n)) = 1，13 * d Mod 616 = 1，可以转换为13*d - 1 = 616 * k ，13x + 616y = 1这个可以使用扩展欧几里德算法求解，可以求得多个解
得到d = 237
### 6.	计算完成，取（n, e）为公钥，公钥为(667, 13), (n, d)为私钥，私钥为(667, 237)
### 7.	加密过程，对m进行加密，要求m < n，加密时计算m的e次方除以n，得到的余数即为密文c，取m = 320
me Mod n = c  
32013 Mod 667 = 88  
c=88
### 8.	解密过程，解密时计算c的d次方除以n，得到的余数即为明文m
cd Mod n = m
88237 Mod 667 = 320

### 使用C#生成RSA密钥

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using System.Numerics;
    using System.Security.Cryptography;
    using System.IO;
    using System.Security.Cryptography.X509Certificates;

    namespace RSATest
    {
        public class RSABuildMgr
        {
            private static readonly int[] smallPrimeArray =
            new int[] { 2,3,5,7,11,13,17,19,23,29,
                31,37,41,43,47,53,59,61,67,71,
                73,79,83,89,97,101,103,107,109,113,
                127,131,137,139,149,151,157,163,167,173,
                179,181,191,193,197,199,211,223,227,229,
                233,239,241 };

            private static readonly int exponent = 65537;

            public void BuildRSAPwd()
            {
                int pwdLen = 2048; //要求2048位密钥长度
                //第一步，生成两个二进制长度为2048/8/2位的素数　
                //由于拉宾米勒测试有极其微小的概率把合数当作素数，所以再计算一遍最大公约数
                BigInteger p, q, n, gcd;
                bool flag = false;

                do
                {
                    do
                    {
                        p = GetPrime(pwdLen / 16);
                        q = GetPrime(pwdLen / 16);
                        gcd = Gcd(p, q);
                    } while (gcd != 1);
                    n = p * q;
                    byte[] pwdBytes = n.ToByteArray();
                    //最后一位是00的话，不计算长度, 00是用来补到字节数组中标识数据是正数的
                    if (pwdBytes[pwdBytes.Length - 1] == 0x0)
                    {
                        flag = pwdBytes.Length - 1 == pwdLen / 8;
                    }
                    else
                    {
                        flag = pwdBytes.Length == pwdLen / 8;
                    }
                } while (!flag);

                BigInteger faiN = (p - 1) * (q - 1);
                BigInteger x, y;

                //求解以下方程：
                //exponent *x + faiN * y = 1;
                BigInteger r = exGcd(exponent, faiN, out x, out y);

                Console.WriteLine("x is " + x.ToString());
                Console.WriteLine("y is " + y.ToString());

                //扩展欧几里德有可能会算出x为负值，此时可以将x设置为x + faiN
                //同样此时也会有对应的y值，但y值并不需要。
                if (x < 0)
                {
                    x += faiN;
                    y = (1 - x * exponent) / faiN;
                    Console.WriteLine("x is " + x.ToString());
                    Console.WriteLine("y is " + y.ToString());
                }

                File.AppendAllText("D:\\temp\\m.txt", n.ToString());
                File.AppendAllText("D:\\temp\\e.txt", exponent.ToString());
                File.AppendAllText("D:\\temp\\d.txt", x.ToString());
            }

            //最大公约数, BigInteger本身提供了BigInteger.GreatestCommonDivisor来计算最大公约数
            public BigInteger Gcd(BigInteger a, BigInteger b)
            {
                if (a == 0) return b;
                if (b == 0) return a;
                if (a % 2 == 0 && b % 2 == 0) return 2 * Gcd(a >> 1, b >> 1);
                else if (a % 2 == 0) return Gcd(a >> 1, b);
                else if (b % 2 == 0) return Gcd(a, b >> 1);
                else return Gcd(BigInteger.Abs(a - b), BigInteger.Min(a, b));
            }

            //扩展欧几里德算法
            private BigInteger exGcd(
            BigInteger a, BigInteger b, out BigInteger x, out BigInteger y)
            {
                if (b == 0)
                {
                    x = 1;
                    y = 0;
                    return a;
                }
                BigInteger r = exGcd(b, a % b, out x, out y);
                BigInteger t = x;
                x = y;
                y = t - a / b * y;

                return r;
            }

            public bool TestForPrime(BigInteger p)
            {
                bool result = false;

                if (p < int.MaxValue / 10)
                {
                    result = PrimeCheckedByDevid((long)p);
                }
                else
                {
                    if (PrimeFilter(p))
                    {
                        result = PrimeCheckedByMillerRabin(p);
                    }
                    else
                    {
                        result = false;
                    }
                }
                return result;
            }

            private BigInteger GetPrime(int len)
            {
                BigInteger p;
                bool flag = true;
                do
                {
                    p = GetBiginteger(1024 / 8);
                    if (PrimeFilter(p))
                    {
                        if (PrimeCheckedByMillerRabin(p))
                        {
                            flag = false;
                        }
                    }
                } while (flag);
                return p;
            }

            private BigInteger GetBiginteger(int len)
            {
                using (RNGCryptoServiceProvider csp = new RNGCryptoServiceProvider())
                {
                    byte[] randomBytes = new byte[len];
                    csp.GetBytes(randomBytes);
                    BigInteger d = new BigInteger(randomBytes.Concat(new byte[] { 0 }).ToArray());
                    return d;
                }
            }

            //试除法，对于小于int.MaxValue/10均可用
            private bool PrimeCheckedByDevid(long n)
            {
                bool result = true;
                if (n == 1)
                {
                    result = false;
                }
                else if (n <= 241)
                {
                    var b = smallPrimeArray.FirstOrDefault(c => c == n);
                    if (b == 0)
                    {
                        result = false;
                    }
                    else
                    {
                        result = true;
                    }
                }
                else
                {

                    long m = (long)(Math.Sqrt(n));
                    for (long i = 2; i <= m; i++)
                    {
                        if (n % i == 0)
                        {
                            result = false;
                            break;
                        }
                    }
                }
                return result;
            }

            //欧拉筛法统计素数个数，max不能太大，太大会out of memory,可以取到一亿
            private int PrimeCountEuler(int max)
            {
                int [] prime = new int[max /2 ];
                bool[] isPrime = new bool[max + 1];
                int totalPrimes = 1;

                isPrime[2] = true;
                for (int i = 3; i <= max; i+=2)
                {
                    isPrime[i] = true;
                }

                prime[0] = 2;
                for (int i = 3; i <= max; i += 2)
                {
                    if (isPrime[i])
                    {
                        prime[totalPrimes++] = i;
                    }
                    for (int j = 1; i * prime[j] <= max && j < totalPrimes; j++)
                    {
                        isPrime[i * prime[j]] = false;
                        if (i % prime[j] == 0) //保证每个数只被筛一次
                        {
                            break;
                        }
                    }
                }
                return totalPrimes;
            }

            
            //素数筛选，用带筛选数除以53个素数，均不能除尽，认为通过筛选
            private bool PrimeFilter(BigInteger b)
            {
                bool result = true;
                for (int i = 0; i < smallPrimeArray.Length; i++)
                {
                    if (b % smallPrimeArray[i] == 0)
                    {
                        result = false;
                        break;
                    }
                }
                return result;
            }

            //素数测试，拉宾米勒测试
            //将数字表示为 n - 1 = 2的s次方乘以d
            //然后测试
            /* 
            •该算法用于判断一个大于 3 的奇数 n 是否素数。参数 k 用于决定 n 是素数的概率。
            •该算法能够肯定地判断 n 是合数，但是只能说 n 可能是素数。
            •第 01 行，将 n – 1 分解为 2s·d 的形式，这里 d 是奇数。
            •第 02 行，将以下步骤(第 03 到 10 行)循环 k 次。
            •第 03 行，◇在 [2, n - 2] 的范围中独立和随机地选择一个正整数 a 。
            •第 04 行，◇计算该序列的第一个值：x ← a的d次方 mod n 。
            •第 05 行，◇如果该序列的第一个数是 1 或者 n - 1，符合上述条件，n 可能是素数，转到第 03 行进行一下次循环。
            •第 06 行，◇循环执行第 07 到 09 行，顺序遍历该序列剩下的 s – 1 个值。
            •第 07 行，◇◇计算该序列的下一个值：x ← x的2次方 mod n 。
            •第 08 行，◇◇如果这个值是 1 ，但是前边的值不是 n - 1，不符合上述条件，因此 n 肯定是合数，算法结束。
            •第 09 行，◇◇如果这个值是 n - 1，因此下一个值一定是 1，符合上述条件，n 可能是素数，转到第 03 行进行下一次循环。
            •第 10 行，◇发现该序列不是以 1 结束，不符合上述条件，因此 n 肯定是合数，算法结束。
            •第 11 行，已经对 k 个独立和随机地选择的 a 值进行了检验，因此判断 n 非常有可能是素数，算法结束。
            在一次检验中，该算法出错的可能顶多是四分之一，检验25次，则出错概率为10的15次方分之一
            参见《计算机程序设计艺术》 第二卷 P358
            */

            private bool PrimeCheckedByMillerRabin(BigInteger n)
            {
                bool result = true;

                //随机获取25个大于1并且小于b - 1的数进行测试
                int testCnt = 25;
                int len = n.ToByteArray().Length;
                BigInteger[] a = new BigInteger[testCnt];
                int index = 0;
                Random r = new Random();
                do
                {
                    int l = r.Next(len + 1);
                    BigInteger tempA = GetBiginteger(l);
                    if (tempA > 1 && tempA < n)
                    {
                        a[index++] = tempA;
                    }
                } while (index < testCnt);

                int s;
                BigInteger d;
                GetMRNum(n, out s, out d);
                for (int i = 0; i < testCnt; i++)
                {
                    BigInteger x = BigInteger.ModPow(a[i], d, n);
                    if (x == 1 || x == n - 1)
                    {
                        continue; //通过检测
                    }
                    else
                    {
                        int j = 1;
                        for (; j < s; j++)
                        {
                            BigInteger x1 = BigInteger.ModPow(x, 2, n);
                            if (x1 == 1)
                            {
                                result = false;
                                break;
                            }
                            else if (x1 == n - 1)
                            {
                                break;
                            }
                            else
                            {
                                x = x1;
                            }
                        }
                        if (j == s && x != 1)
                        {
                            result = false;
                            break;
                        }
                        if (!result)
                        {
                            break;
                        }
                    }
                }
                return result;
            }

            /// <summary>
            /// 将奇数n表示为n - 1 = 2的s次方乘以d (在是奇数)
            /// </summary>
            /// <param name="n"></param>
            /// <param name="s"></param>
            /// <param name="d"></param>
            private void GetMRNum(BigInteger n, out int s, out BigInteger d)
            {
                d = n - 1;
                s = 0;
                do
                {
                    s++;
                    d = d >> 1;
                } while (d % 2 == 0);
            }
        }
    }
