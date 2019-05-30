可持久化 Trie 的方式和可持久化线段树的方式是相似的，即每次只修改被添加或值被修改的节点，而保留没有被改动的节点，在上一个版本的基础上连边，使最后每个版本的 Trie 树的根遍历所能分离出的 Trie 树都是完整且包含全部信息的。

大部分的可持久化 Trie 题中，Trie 都是以 01-Trie 的形式出现的。

??? note " 例题[最大异或和](https://www.lydsy.com/JudgeOnline/problem.php?id=3261)"

对一个长度为 $n$ 的数组 $a$ 维护以下操作：

1.  在数组的末尾添加一个数 $x$ ，数组的长度 $n$ 自增 $1$ 。

2.  给出查询区间 $[l,r]$ 和一个值 $k$ ，求当 $l\le p\le r$ 时， $k \oplus \bigoplus^{n}_{i=p} a_i$ 。

这个求的值可能有些麻烦，利用常用的处理连续异或的方法，记 $s_x=\bigoplus_{i=1}^x a_i$ ，则原式等价于 $s_{p-1}\oplus s_n\oplus k$ ，观察到 $s_n \oplus k$ 在查询的过程中是固定的，题目的查询变化为查询在区间 $[l-1,r-1]$ 中异或定值（ $s_n\oplus k$ ）的最大值。

继续按类似于可持久化线段树的思路，考虑每次的查询都查询整个区间。我们只需把这个区间建一棵 Trie 树，将这个区间中的每个树都加入这棵 Trie 中，查询的时候，尽量往与当前位不相同的地方跳。

查询区间，只需要利用前缀和和差分的思想，用两棵前缀 Trie 树（也就是按顺序添加数的两个历史版本）相减即为该区间的线段树。再利用动态开点的思想，不添加没有计算过的点，以减少空间占用。

同时，因为取到的最大值可能是当 $l=1$ 时，此时 $l-1=0$ ，所以应事先加入这个节点。

```cpp
#include<bits/stdc++.h>
using namespace std;
inline int read()
{
    char ch=getchar();
    while(!isdigit(ch)) ch=getchar();
    int ans=0;
    while(isdigit(ch))
    {
        ans=ans*10+ch-48;
        ch=getchar();
    }
    return ans;
}
#define N 300005
struct Trie
{
    struct Node
    {
        int ch[2];
        int siz;
    };
    Node t[N*48];
    int ncnt;
    int rcnt,rot[N*2];
    void insert(int x,int rt)
    {
        rt=rot[rt];
        int cur=rot[++rcnt]=++ncnt;
        for(int i=24;i>=0;i--)
        {
            int tmp=(x>>i)&1;
            t[cur].ch[tmp]=++ncnt;
            t[cur].ch[tmp^1]=t[rt].ch[tmp^1];
            t[cur].siz=t[rt].siz+1;
            cur=t[cur].ch[tmp];
            rt=t[rt].ch[tmp];
        }
        t[cur].siz=t[rt].siz+1;
    }
    int query(int x,int l,int r)
    {
//		printf("%d %d %d\n",x,l,r);
        int ans=0;
        int rtl=rot[l-1];
        int rtr=rot[r];
        for(int i=24;i>=0;i--)
        {
            int tmp=(x>>i)&1;
            if(t[t[rtr].ch[tmp^1]].siz-t[t[rtl].ch[tmp^1]].siz>0)
            {
//				printf("** %d",i);
                ans^=(1<<i);
                rtr=t[rtr].ch[tmp^1];
                rtl=t[rtl].ch[tmp^1];
            }
            else
            {
                rtr=t[rtr].ch[tmp];
                rtl=t[rtl].ch[tmp];
            }
        }
        return ans;
    }
};
Trie tr;
int main()
{
    int n,m;
    cin>>n>>m;
    int x=0;
    tr.insert(0,0); //处理最大值当l=1时取到的情况
    for(int i=1;i<=n;i++)
    {
        x^=read();
        tr.insert(x,i);
    }
    char opt[3];
    for(int i=1;i<=m;i++)
    {
        scanf("%s",&opt);
        if(opt[0]=='A')
        {
            x^=read();
            tr.insert(x,tr.rcnt);
        }
        else
        {
            int l,r,xx;
            l=read(),r=read(),xx=read();
            printf("%d\n",tr.query(xx^x,l,r));
        }
    }
    return 0;
}
```
