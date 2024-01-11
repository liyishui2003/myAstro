---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'Educational Codeforces Round 152 (Rated for Div. 2)'
pubDate: 2024-01-11
description: '一些训练'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/65a0052d871b83018a7ecf4c.jpg'
    alt: '在南京'
tags: ["Codeforces"]
---

### A Morning Sandwich
奶酪和火腿可以夹在面包里，已知奶酪、火腿、面包的数量，问你的汉堡包最多可以叠多少层（奶酪/火腿/面包都算一层）

解：

显然奶酪和火腿是等价的，可以直接加起来。
分类讨论一下谁多即可
```
void solve(){
    int b,c,h;
    cin>>b>>c>>h;
    if(c+h<=b-1) cout<<(c+h)*2+1<<endl;
    else cout<<(b-1)*2+1<<endl;
}
```

### B Monsters
有n只怪兽，每只的血量为ai。每次操作会选择当前血量最高的怪兽减少k滴血。
求怪兽死亡的先后顺序。

解：

最开始没注意到数据范围老老实实打了一个优先队列，然后tle了。
想一下，当第一次怪兽死之前，应该是什么样的局面？
显然每只怪兽的血量会<=k。
想到这里就非常显然了，对每个怪兽的血量%k，如果%k后==0的话设为k。
然后从大到小死亡。
```
const int N=1e6+5;
struct node{
	int id,val;
	bool operator < (const node a) const{
		if(val==a.val) return id>a.id;
		return val<a.val;
	}
};
void solve(){
	int n,k;
	cin>>n>>k;
	priority_queue<node,vector<node>,less<node>>q;
	for(int i=1;i<=n;i++) {
		int x;
		cin>>x;
		x%=k;
		if(!x) x=k;
		q.push((node){i,x});
	}	
	while (!q.empty())
	{
		node v=q.top();
		q.pop();
		cout<<v.id<<" ";
	}
	cout<<endl;
}
```

### C Binary String Copying
给定一个01串，有m种操作，每个操作给出区间 [l,r] ,并把区间[l,r]排序。
每个操作是独立的，求最后能得到多少本质不同的字符串？

解：

考虑对区间[l,r]进行操作时，到底做了什么？
首先如果已经有序不用考虑。
如果无序的话，一定是存在某个1在0的前面。
显然对于前缀的0和后缀的1都不需要考虑，真正需要改变的是[从l开始的右边第一个1,从r开始的左边第一个0]。

那么问题就好办了，O(n)的求出每个数右边第一个1（记作r1），左边第一个0(记作l0)，
区间[l,r]实质改变的区间就是[r1[l],l0[r]]，对每个区间用set去重即可。
```
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;
int sum[N],l0[N],r1[N];
void solve(){
    int n,k;
    cin>>n>>k;
    string s;cin>>s;
    s=" "+s;
    s[0]=0;
    for(int i=1;i<=n;i++){
        //left most zero
        if(s[i]=='0') l0[i]=i;
        else l0[i]=l0[i-1];
        sum[i]=sum[i-1]+(s[i]=='1');
    }
    for(int i=n;i>=1;i--){
        if(s[i]=='1')  r1[i]=i;
        else r1[i]=r1[i+1];
    }
    set<pair<int,int>>q;
    int flag=0;
    while(k--){
        int l,r;cin>>l>>r;
        int sum_1=sum[r]-sum[l-1];
        if(sum[r]-sum[r-sum_1]==sum_1) {
            flag=1;
            continue;
        }
        q.insert({r1[l],l0[r]});
    }
    cout<<flag+q.size()<<endl;
}
int main(){
    int t;
    cin>>t;
    while(t--){
        solve();
    }
}
```
这里自己吃了个教训，最开始求r0和l1直接写了个前缀和+二分，看不出什么毛病，一直改不出来。

后面发现写麻烦了，重构了一遍，用O（n）的写法一发就过了。

所以：
* 能写简单的代码就不要写复杂，增加debug开销
* 一直改改不出来时考虑重构

### D Array Painting
有一列数a，ai的值为0，1，2。最开始每个位置都是蓝色，要把它们涂成红色。
* 选择一个蓝色花费1的代价涂成红色
* 选择一个红色的位置，把它相邻的蓝色位置涂红，涂一次自己的ai值-1，为0不能涂。

解:

我先考虑了没有2时怎么办，显然连续的1要一块涂，然后一段连续1能带一个0，这个好处理。
然后考虑2，首先2是能当1用的，所以答案上界就是刚才没有2时的上界。
考虑2左右两边的分布。
* 2左右两边都是0 这个2被孤立了，没有人能来帮它，那么自己肯定要涂。既然自己涂了顺带把周围0也涂了呗。
* 2左右两边都是1 这个2起码能当1用，并且如果有一段连续的1，中间混入一个2，肯定是先用2最好，还能把两端的1留给 两端的1旁边的0
* 2左右两边一个1一个0 这个2能当作一连串可涂段的开头，还能顺便把右边的0给涂了，肯定比涂这段里的某个1好
* 2在边界 此时等价为1
综上所述，有2涂2，尽量扩展。
涂完2再来看1，一连串的1能带一个0，能带则带。

```
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;
int a[N],vis[N];
int main(){
    int n;
    cin>>n;
    for(int i=1;i<=n;i++) cin>>a[i];
    
    if(a[1]==2) a[1]=1;
    if(a[n]==2) a[n]=1;

    int ans=0;
    for(int i=1;i<=n;i++){
        if(a[i]==2&&vis[i]==0){
            ans+=1;
            vis[i]=1;
            int j=i-1;
            while(j>=1){
                vis[j]=1;
                if(!a[j]) break;
                j--;
            }
            j=i+1;
            while(j<=n){
                vis[j]=1;
                if(!a[j]) break;
                j++;
            }
        }
    }

   /*
   for(int i=1;i<=n;i++) cout<<vis[i]<<" ";
    cout<<endl;
    cout<<"ans: "<<ans<<endl;
   */ 

    for(int i=1;i<=n;i++){
        if(vis[i]) continue;
        if(a[i]==1){
            ans++;
            vis[i]=1;
            int flag=0;
            if(i>=2&&vis[i-1]==0&&a[i-1]==0) {
                vis[i-1]=1;
                flag=1;
            }
            int j=i+1;
            while(j<=n){
                if(!a[j]) break;
                vis[j]=1;
                j++;
            }
            if(!flag&&j<=n&&vis[j]==0&&a[j]==0) {
                vis[j]=1;
            }
        }
    }

/*
    for(int i=1;i<=n;i++) cout<<vis[i]<<" ";
    cout<<endl;
    cout<<"ans: "<<ans<<endl;
   // cout<<ans<<endl;
*/
    
    for(int i=1;i<=n;i++){
        ans+=1-vis[i];
    }

    cout<<ans;
}
```
