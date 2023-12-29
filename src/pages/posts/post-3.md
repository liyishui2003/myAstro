---
layout: ../../layouts/MarkdownPostLayout.astro
title: Educational Codeforces Round 154 (Rated for Div. 2) A-D
author: liyishui
description: "一些训练"
image:
    url: "http://img02.sogoucdn.com/app/a/201103/fd0cea76be18023c-49d895bce49191af-ee0f26aed93b07d51235ab095caddda4"
    alt: "Thumbnail of Astro rays."
pubDate: 2023-12-30
tags: ["Codeforces"]
---

传送门：[edu154/div2](https://codeforces.com/contest/1861)

### A. Prime Deletion
- 题意：给定一个0-9的排列，要求一个长度>=2的子序列，使得该子序列是质数

- 做法：考虑31或者13即可。不过当时没有立刻想到，感觉1000以内的质数必有答案，打了暴力。用时就多了点。


<details>
<summary>Code</summary>

```
#include<bits/stdc++.h>
using namespace std;
int pri[1005],pos[11],cnt;
void solve(){
	string s;
	cin>>s;
	int len=s.length();
	memset(pos,0,sizeof pos);
	for(int i=0;i<len;i++){
		int x=s[i]-'0';
		pos[x]=i;
	}
	for(int i=1;i<=cnt;i++){
		int l=0,x=pri[i];
		if(x<=10) continue;
		int num[11];
		while(x){
			l++;
			num[l]=x%10;
			x/=10;
		}
		reverse(num+1,num+1+l);
		int ok=1;
		for(int i=2;i<=l;i++){
			if(pos[num[i]]>pos[num[i-1]]) continue;
			ok=0;
		}
		if(ok){
			cout<<pri[i]<<endl;
			return;
		}
	}
}
void init(){
	for(int i=2;i<1000;i++){
		int flag=1;
		for(int j=2;j<i;j++)
			if(i%j==0) flag=0;
		if(flag==1){
			cnt++;
			pri[cnt]=i;
		}
	}
}
int main(){
//	freopen("lys.in","r",stdin);
	ios_base::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
	init();
	int t;
	cin>>t;
	while(t--){
		solve();
	}
}

```
</details>

### B.Two Binary Strings
- 题意：给定01串，第一位必然是0，最后一位是1，可以做的操作是每次把两个字符相同(就是都1或者都是0)之间的所有字符都变成一样，问能否使ab串相等

- 做法：有一个观察：所有可以变成相同的串最后必然会变成00001111串的形式，所以考虑能否变成这样。符合情况当且仅当存在i，使得a[i]=b[i]='1',a[i-1]=b[i-1]='0'

<details>
<summary>Code</summary>

```
#include<bits/stdc++.h>
using namespace std;
const int maxn=1e6+5;
int a[maxn],l[maxn],r[maxn];
void solve(){
	int n;
	cin>>n;
	for(int i=1;i<=n;i++) cin>>a[i],l[i]=r[i]=0;
	
	l[0]=l[n+1]=r[0]=r[n+1]=0;
	for(int i=2;i<=n;i++){
		l[i]=l[i-1]+(a[i]>=a[i-1]);
	}
	
	for(int i=n-1;i>=1;i--){
		r[i]=r[i+1]+(a[i]>=a[i+1]);
	}
	
	int ans=n+1;
	for(int i=1;i<=n;i++){
		ans=min(ans,l[i]+1+r[i+1]);
	}
	ans=min(ans,l[n]+1);
	ans=min(ans,r[1]);
	cout<<ans<<endl;
}
int main(){
	//freopen("lys.in","r",stdin);
	int t;
	cin>>t;
	while(t--){
		solve();
	}
}
```
</details>

### C.Queries for the Array
- 题意：给定一个固定的操作序列，+表示在数列末尾加一个任意值，-表示删去当前末尾的数，1表示当前的序列是有序的，0表示无须，问是否可能。

- 做法：最开始的想法是维护一个最近的有序长度，然后详细讨论从状态1变成状态0，从状态0变成状态1可不可行等等，但是想法有些难落地。从维护有序长度出发，考虑干脆维护一个区间好了，那最终思路就出来了。具体的细节看代码

<details>
<summary>Code</summary>

```
#include<bits/stdc++.h>
using namespace std;
void solve(){
	string s;
	cin>>s;
	int len=s.length();
	int arr_l=0,l=0,r=0;
	for(int i=0;i<len;i++){
		if(s[i]=='+'){
			if(r==arr_l) r++;
			arr_l++;
		    if(!l) l++;
		}
		else if(s[i]=='-'){
			if(arr_l==r){
				r--;
			}
			if(arr_l==l){
				l--;
			}
			arr_l--;
		}
		else if(s[i]=='1'){
			if(arr_l!=r&&arr_l!=1){
			//	cout<<i<<" "<<l<<" "<<r<<" "<<arr_l<<endl;
				cout<<"NO"<<endl;
				return;
			}
			l=r;
		}
		else if(s[i]=='0'){
			if(l>=arr_l) {
			//	cout<<i<<" "<<l<<" "<<r<<" "<<arr_l<<endl;
				cout<<"NO"<<endl;
				return;
			}
			r=min(r,arr_l-1);
			l=min(l,r);
		}
	}
	cout<<"YES"<<endl;
}
int main(){
	//freopen("lys.in","r",stdin);
	ios_base::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
	
	int t;
	cin>>t;
	while(t--){
		solve();
	}
}
```
</details>

### D.Sorting By Multiplication
- 题意：每次可以选择一个区间对区间内每一个数乘上一个整数（可以为负），要求使得数组最后变成严格递增的最少次数

- 做法：首先考虑如果只用正数去乘的话就很显然。再考虑乘以一个负数。
这里有几个观察：
  - 1 是负数段一定是连续的，设想如果不连续，中间有一截正数，那最后还是要把中间的正数变成负数，或者把负数再变回去（显然这没有意义），基于这种思想我们就可以把负数段合并了。
  - 2 负数段一定是在前面的部分，也就是说，它是一个前缀。如果不是前缀的话，前面肯定有正数，那么同理，要么把正数变成负数，要么再把负数段变成正数(你干啥子咧)。
  
  - 3 所以最终的思路是枚举分段点，计算前面的一段变成严格递减的代价，后面一段，然后前后合并，记得+1（把递减的前缀乘以一个负数）

<details>
<summary>Code</summary>

```
#include<bits/stdc++.h>
using namespace std;
const int maxn=1e6+5;
int a[maxn],l[maxn],r[maxn];
void solve(){
	int n;
	cin>>n;
	for(int i=1;i<=n;i++) cin>>a[i],l[i]=r[i]=0;
	
	l[0]=l[n+1]=r[0]=r[n+1]=0;
	for(int i=2;i<=n;i++){
		l[i]=l[i-1]+(a[i]>=a[i-1]);
	}
	
	for(int i=n-1;i>=1;i--){
		r[i]=r[i+1]+(a[i]>=a[i+1]);
	}
	
	int ans=n+1;
	for(int i=1;i<=n;i++){
		ans=min(ans,l[i]+1+r[i+1]);
	}
	ans=min(ans,l[n]+1);
	ans=min(ans,r[1]);
	cout<<ans<<endl;
}
int main(){
	//freopen("lys.in","r",stdin);
	int t;
	cin>>t;
	while(t--){
		solve();
	}
}
```
</details>



###### 我想说，在很多时候，要走很多错误的路才能通往正确的那一条。所以现在还在茫然和没有答案，我跟自己说不要害怕。
# 