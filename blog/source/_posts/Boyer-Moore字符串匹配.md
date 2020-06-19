---
title: Boyer-Moore字符串匹配
date: 2020-06-18 10:28:41
tags: Algorithm
---
Boyer-Moore是一种快速的字符串匹配算法，它对目标字符串（模式串）进行倒序查找，并在字符串匹配失败时无需像暴力查找那样对整个模式串进行重新匹配，而是通过坏字符和好后缀计算滑动窗口，降低查询的时间复杂度。
如图：在 文本HERE IS A  SIMPLE EXAMPLE中查找模式串EXAMPLE。Boyer-Moore算法（下文简称B-M算法）从模式串的最后一个位置开始与文本进行比较，模式串中的E与文本中的S不匹配，则称字符S为坏字符。
![1.png](https://upload-images.jianshu.io/upload_images/19433926-193bf53fc6aad958.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<escape><!-- more --></escape>
由于在一开始比较的时候，模式串和文本头部是对齐的。所以，此时文本的前7位字符不可能与模式串完全匹配，我们可以直接将模式串移动7位到文本S的后一位。如下图：
![2.png](https://upload-images.jianshu.io/upload_images/19433926-398ad5341ce964be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时，文本中的P和模式串中的E不匹配，则P也是坏字符，但由于P字符同时出现在了模式串的第4位，于是将模式串右移2位，将两个P字符对齐。那么，通过坏字符进行移位的规则是什么呢，通过以上两次比较，总结出坏字符移位规则：

<center>坏字符后移位数=坏字符在模式串中失配的位置-坏字符在模式串中上一次出现的位置</center>

如果坏字符之前从未在模式串中出现，则位置默认为-1。比如：在第一次比较时，字符S出现在模式串的第6位，而在E字符左侧无字符S，即字符S从未在模式串中出现，则据计算规则得出模式串移动位数=6-（-1）=7。在第二次比较时，字符S的上一次出现是在模式串的第四位，则后移位数=6-4=2。
![3.png](https://upload-images.jianshu.io/upload_images/19433926-6d1bae2e1a736aec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在将模式串中的P后移2位对齐后得到上图，此时从最后一位E开始比较：
![4.png](https://upload-images.jianshu.io/upload_images/19433926-e233d71437cc3d74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 模式串字符E与文本字符E匹配，继续倒序比较
  ![5.png](https://upload-images.jianshu.io/upload_images/19433926-a783848cd04a2206.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 模式串字符L与文本字符L匹配，继续倒序比较
  ![6.png](https://upload-images.jianshu.io/upload_images/19433926-18b36fc46fac03d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 模式串字符P与文本字符P匹配，继续倒序比较
  ![8.png](https://upload-images.jianshu.io/upload_images/19433926-a2aecae2f1650c28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 模式串字符M与文本字符M匹配，继续倒序比较
  ![9.png](https://upload-images.jianshu.io/upload_images/19433926-d759ac7f50e5f39a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 模式串字符A与文本字符I不匹配，I为坏字符
  此时，得到I为坏字符，而前面匹配成功的MPLE，PLE,LE,E则被称作好后缀。通过坏字符移位规则计算得到此时后移位数为2-（-1）=3位。那在这种部分字符有4位匹配的情况下，如果仅移动3位，显然移动后很可能最后一位仍然不匹配，那还有更好的移位方法么？

在此次的匹配中，我们得到了4个好后缀，在坏字符移位不够多的情况下，尝试计算好后缀的移位结果，选取最优解。好后缀的计算规则如下：

<center>好后缀后移位数=好后缀的位置-好后缀在模式串中上一次出现时的位置 </center>

- 和坏字符类似，如果好后缀只出现一次，则其上一次出现位置为-1
- 好后缀的位置都指其最后一个字符在模式串中的位置
- 如有多个好后缀，则在计算时，除了最长的那个好后缀，其他好后缀必须出现在模式串头部中

以上文的好后缀MPLE,PLE,LE,E为例，接下来进行其好后缀移位计算：

- MPLE,未出现，上一次出现位置为-1
- PLE,未出现在头部，上一次出现位置为-1
- LE,未出现在头部，上一次出现位置为-1
- E,出现在头部，上一次出现位置为0

此处只能选取E进行好后缀计算，得到后移位数=6-0= 6位。而此时坏字符后移位数位3位，显然选用好后缀移位更多，此时选择好后缀后移，移动后，如下图：

![12.png](https://upload-images.jianshu.io/upload_images/19433926-7ca9c4f4d04c2326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时，P字符失配，根据坏字符规则6-4=2位，后移2位，移位后效果如下图：
![13.png](https://upload-images.jianshu.io/upload_images/19433926-6177199269dd2b0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```
using System;
using System.Collections.Generic;
 
namespace Boyer_Moore
{
    class Program
    {
        static void Main(string[] args)
        {
            string text = "This is a simple example";
            string pattern = "example";
            int index = Arithmetic_BM(text, pattern );
            Console.WriteLine(index);
            Console.ReadKey();
        }
 
        /// <summary>
        /// 单个匹配
        /// </summary>
        /// <param name="operateStr"></param>
        /// <param name="findStr"></param>
        /// <returns></returns>
        public static int Arithmetic_BM(string operateStr, string findStr)
        {
            //i：匹配开始的索引，j：operateStr字符串的索引迭代，k：findStr字符串索引迭代
            int i = 0, j = findStr.Length - 1, k = j;
            int n, m = 0; //n:坏字符规则计算出的移动位数，m:好后缀计算出的移动位数
 
            while (k >= 0 && j < operateStr.Length)
            {
                if (k == 0) //全部匹配，return
                {
                    return i;
 
                }
                if (operateStr[j] == findStr[k]) //匹配，next
                {
                    j--;
                    k--;
                }
                else
                {
                    //当k<要匹配的字符串长度时，说明已经有匹配的字符了，即有“好后缀”
                    if (k < findStr.Length - 1)
                    {
                        //采用"好后缀规则"，先找出“全好后缀”有没有在前面存在
                        var goodSuffix = findStr.Substring(k + 1); //分割出全好后缀
                        var tempStr = findStr.Substring(0, k + 1); //去掉好缀后的字符串
                        //最全好后缀在剩下的字符串中出现
                        if (tempStr.Contains(goodSuffix))
                        {
                            var lastGoodSuffix = char.Parse(goodSuffix.Substring(goodSuffix.Length - 1)); //好后缀的最后一个字符
                            //找出 该字符的出现位置
                            IList<int> indexs = new List<int>();
                            for (int x = 0; x < tempStr.Length; x++)
                            {
                                if (lastGoodSuffix == tempStr[x])
                                {
                                    indexs.Add(x);
                                }
                            }
                            //找出 好后缀在搜索词中的上一次出现位置
                            var result = -1;
                            for (int x = indexs.Count - 1; x >= 0; x--)
                            {
                                if (indexs[x] >= goodSuffix.Length &&
                                    tempStr.Substring(indexs[x] - goodSuffix.Length + 1, goodSuffix.Length) == goodSuffix)
                                {
                                    result = indexs[x];
                                    break;
                                }
                            }
                            //好后缀规则结果
                            m = findStr.Length - 1 - result;
                        }
                        //最长好后缀没有没出现，但是好后缀最后一个字符，出现在头部
                        //后移位数 = 好后缀的位置 - (0)搜索词中的上一次出现位置
                        else if (findStr.Substring(0, 1) == findStr.Substring(findStr.Length - 1))
                        {
                            m = findStr.Length - 1;
                        }
                        else //好后缀只出现一次  (后移位数 = 好后缀的位置 - (-1)搜索词中的上一次出现位置)
                        {
                            m = findStr.Length;
                        }
                    }
                    //坏字符规则：后移位数 = 坏字符的位置 - 搜索词中的上一次出现位置
                    n = (j - i) - findStr.LastIndexOf(operateStr[j]);
                    //比较坏字符规则和好后缀规则移动的位数，得出最终移动位数
                    if (n > m)
                    {
                        i += n;
                        j = i + findStr.Length - 1;
                    }
                    else
                    {
                        i += m;
                        j = i + findStr.Length - 1;
                    }
                    k = findStr.Length - 1;
                    m = 0; //清零
                }
            }
            return -1;
        }
    }
}
```

