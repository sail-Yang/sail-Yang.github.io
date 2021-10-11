---
title: 冒泡排序及C语言实现
date: 2021-10-11 10:10:38
tags:
- 排序算法
categories:
- 数据结构和算法
mathjax: true
---

# 前言

&emsp;&emsp;在写链表的作业时用到了这种排序方法，虽然是一种很简单的排序算法，并且效果不是很好，时间复杂度是O(n^2^)，但这是我第一个掌握的排序算法，所以还是值得记录一下的。

![Bubble_sort_animation](https://gitee.com/sail-Yang/blogiamge/raw/master/img/20211011101536.gif)

<!--more-->

# 算法描述

&emsp;&emsp;算法描述：

- 比较相邻的元素，如果第一个元素 >第二个元素，就交换两个元素的位置。

- 从头到尾对每一对相邻的元素做同样的工作。

- 对所有元素重复上述操作，除了最后一个元素。

- 持续每次对越来越少的的元素重复上述操作，直到没有任何数字需要比较。

  ---

&emsp;&emsp;可能用文字描述还不够清晰，我们来看一个实例的运行：

&emsp;&emsp;假设现在我们有下面这样的一排数：

  |  1   |  9   |  8   |  4   |  2   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;比较1和9，1< 9，不交换，再比较9和8，9>8交换：

  |  1   |  8   |  9   |  4   |  2   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;同理9>4,9>2，交换：

  |  1   |  8   |  4   |  2   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;从头开始，**对1、8、4、2进行冒泡排序**（这里强调是因为后面可以针对此优化算法），我们开始比较1和8，1<8,不交换，比较8和4，交换：

  |  1   |  4   |  8   |  2   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;同理，8和2交换：

  |  1   |  4   |  2   |  8   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;重复上述操作：

  |  1   |  4   |  2   |  8   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

  |  1   |  2   |  4   |  8   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

  |  1   |  2   |  4   |  8   |  9   |
  | :--: | :--: | :--: | :--: | :--: |

&emsp;&emsp;以上就是一个冒泡排序的完整操作，下面我们来用C语言实现这一算法。

---

  # C语言实现

  &emsp;&emsp;我们针对一个带头结点的链表使用冒泡排序，使其从头结点到尾结点按照升序排列。链表结点声明如下：

```c
typedef int ElemType;
typedef struct node
{
    ElemType data;
    struct node *next;
}Node;
typedef Node *LinkList;
```

&emsp;&emsp;假设现在:

- `Create_LinkList()`函数返回一个随机生成的链表。
- `Get_Len(LinkList L)`返回int型的不算头结点的长度。
- `Get_Pre(LinkList L,Node p)`返回输入结点的上一个结点。

&emsp;&emsp;下面我们来实现冒泡排序：

```c
void Bubble_Sort(LinkList L)
{
    int i,j,len;
    len = Get_Len(L)		//获得链表长度（不算头结点）
    Node p,p_next,pre_p;
    p = L->next;
    for(i=0;i<len-1;i++)	//设链表长度为6，最后一个不用操作，所以咱们5次就够了，
    {
        j = len - i -1;		//每循环一次，我们就可以少比较一个元素，因为每次大的都会被拍到后面，没必要在比较
        while(p->next != NULL && j!=0)
        {
            j--;
            p_next = p->next;
            if(p->data > p_next->data)
            {
                pre_p = GetPreNode(L,p);
                pre_p->next = p_next;
                p->next = p_next->next;
                p_next->next = p;
            }else
            {
                p = p_next;
            }
        }
        p = L->next;
    }
    return;
}
```

