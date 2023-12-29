---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'Codeforces Round #843 (Div. 2)'
pubDate: 2023-12-30
description: '最近看的一部电影和一些算法训练'
author: 'liyishui'
image:
    url: 'https://p1.ssl.qhimg.com/t011c48ab4cc1b56a8c.jpg'
    alt: '最近看的一部电影《末路狂花》.'
tags: ["Codeforces"]
---

 <p>安利一个叫codeforces better的插件&nbsp;&nbsp;<a title="点击即送" href="https://greasyfork.org/zh-CN/scripts/465777-codeforces-better" target="点击即送">https://greasyfork.org/zh-CN/scripts/465777-codeforces-better</a>
今天装了后使用cf体验非常舒适。</p>


<p><a href="https://codeforces.com/contest/1775/problem/A1">Gardener and the Capybaras (easy version)</a></p>
<p>问字符串s能否切分成3个字符串a、b、c，且满足a&lt;=b&amp;&amp;c&lt;=b或者b&gt;=a&amp;&amp;b&gt;=c</p>
<p>s长度&lt;=100，暴力枚举即可</p>


<p><a href="https://codeforces.com/contest/1775/problem/A2">A2 - Gardener and the Capybaras (hard version)</a></p>
<p>题意同上，但是s的长度变成2e5</p>
<p>那肯定有一些特殊的解法</p>
<p>考虑什么情况下字符串最小，s仅由a、b构成，显然如果存在a，那么a肯定最小</p>
<p>但是a可能出现在两端或者中间，讨论一下</p>
<ul>
<li>如果出现在中间，记出现的位置为pos，直接令字符串b="a"即可，字符串a=s[0:pos-1]，字符串c=s[pos+1,|s|-1]</li>
<li>如果出现在两端，则中间串必然为bbb...bbb，令字符串a=s[0]，字符串c=s[|s|-1]，字符串b=s[1,|s|-2]，这样做一定是对的，证明：　　
<ul>
<li>不失一般性地令s[0]="a"，则s[|s|-1]若="a"，中间的串必然大于两边的串</li>
<li>若s[|s|-1]="b"，则中间的串必然大于两边的串&nbsp; &nbsp;　</li>
</ul>
</li>
</ul>

<p><a href="https://codeforces.com/contest/1775/problem/C">C - Interesting Sequence</a></p>
<p>给定n和x(n&lt;=1e18)，要求最小的m使得 n&amp;(n+1)&amp;(n+2)..&amp;m=x</p>
<p>一开始直接做差，令del=n-x，则del在二进制表示下有1的位置必是x有1的位置的子集（因为&amp;只会使n的1减少，&amp;运算是单调不增的）</p>
<p>那么考虑del的最高位，显然要把这位的1消掉必须要加到这位会被进位</p>
<p>又因为这个过程是不断累加的，所以最高位之前的1也都会被抹去</p>
<p>所以只要在n中把del出现过的1都抹去，再在最高位+1的地方补1即可</p>
<p>不过这样写要特判一些东西：</p>
<p>显然</p>
<ol>
<li>x&lt;=n</li>
<li>del是n的子集</li>
<li>要保证del是x的某个后缀（对应&ldquo;最高位之前的1也都会被抹去&rdquo;）</li>
<li>补位时不能进位。这点很重要，有点难考虑到。如果补位会进位的话说明n的这位也会变成0，就无法得到x</li>
</ol>

<p><a href="https://codeforces.com/contest/1775/problem/D">D - Friendly Spiders</a></p>
<p>经典图论，两个数之间gcd!=1即可连边，代价为1。问从s到t的最小代价。</p>
<p>跟gcd有关的话很容易考虑到枚举因数，我最开始考虑的是按照每个质因数去分类，但是这样往下推会遇到无法解决的问题</p>
<p>正解是按照因数去分类（是的，直接枚举因子，略暴力了。。），对于每个点，和自己的因子连一条边权为1的边，这里的因子是新建的虚点</p>
<p>然后bfs，答案为最短路/2+1</p>

<p>&nbsp;</p>