---
title: 单链表相关问题
layout: post
description: 单链表的常用操作实现:删除、增加、反转、环形...
tags:
  - 数据结构
  - tech
---

下面是对单链表的一些总结：

定义情况如下：

```c
#define TRUE 1
#define FALSE 0
typedef int bool;
typedef struct node {
        struct node * next;
        int value;
} Node;
```

# 1. 针对一个单链表的常用操作


##（1）给点一个值value，删除链表中等于此值的节点


        先判断头节点的值是否为value，是的话，改变头指针，指向下一个节点；
        否则，遍历链表，知道找到一个节点，他的next的值为value，
        将此节点指向下下个节点，最后删除中间的节点。
        程序如下：


```c  
void removeNode ( Node ** head, int value){
    if ( head == NULL || *head == NULL )
        return;
    Node * deleteNode = NULL;
    if ( (*head)->value == value){ /* if *head is the node to be deleted */
        deleteNode = *head;
        *head = (*head)->next;
    }else{
        while ( (*head)->next != NULL && (*head)->next->value != value ){
                (*head) = (*head)->next;
        }
 
        if ( (*head)->next != NULL && (*head)->next->value == value){ /* ensure we 've found the node to be deleted */
            deleteNode = (*head)->next;
            (*head)->next = (*head)->next->next;
        }
    }
    if ( deleteNode != NULL ){ /* free the node and set it to NULL in case of stray pointer */
        free(deleteNode);
        deleteNode = NULL;
    }
 
}
```

 
##（2）在链表尾部新增一个节点，其值为value：


    先分配空间，初始化新的节点；如果链表为空，则头部指向新节点；
    否则，在尾部直接添加新节点即可；
    程序如下：

```c   
void addNodeToTail ( Node ** head, int value){
    Node * newNode = (Node *) malloc(sizeof(Node));
    newNode->next = NULL;
    newNode->value = value;
 
    if (*head==NULL){
        *head = newNode;
    }else{
        while ( (*head)->next != NULL )
            *head = (*head)->next;
        (*head)->next = newNode;
    }
}
```

##（3）打印一个单链表，但是输出的顺序反向：


    可以利用递归，知道最后一个节点（next=NULL）时才打印；
    然后从栈中返回前面调用的函数，实现了反向打印。
    程序如下：

```c 
void printListReversely ( Node * head ){
    if ( head == NULL )
        return;
    if ( head->next != NULL )
        printListReversely (head->next);
    printf("%d\t",head->next);
}
```

##（4）翻转一个单链表，即把原来的链表反向：


   只需改变链表中每一个箭头的方向即可；
   需要3个变量，一个保存现在的节点，一个保存前一个节点，一个保存下一个节点。
   程序如下：

```c
Node *  reverseLinkList ( Node * pHead ){
    if (pHead==NULL)
        return NULL;
    Node * pPre =NULL;
    Node * pCur = pHead;
    Node * pNext = NULL;
    while ( pCur != NULL ){
        pNext = pCur->next;
        pCur->next = pPre;
        pPre = pCur;
        pCur = pNext;
    }
    return pCur; /* pCur is the last node b4; and it's the head of the new node now */
}
```

##(5) 判断一个链表是否有环


   如果链表有环，必定出现在尾部；就像去操场前，需要先走一段路，再进入圆形的跑道；
   可以设定两个指针，开始都指向head，然后一个指针每次走2步，另外一个每次一步；
   如果有环，经过圆形环后，快指针必定会追上慢指针；
   如果碰到指针指向NULL了，则没有环。

   程序如下：

```c
bool isLoopInList (Node * head){
    Node * slow = head;
    Node * fast = head;
    while ( fast != NULL && fast->next != NULL ){
        slow = slow->next;
        fast = fast->next->next;
        if ( slow == fast)
            return TRUE;
    }
    if ( fast==NULL || fast->next == NULL )
        return FALSE;
 
 
}
```

##（6）如果一个链表有环，请找出进入环的第一个节点


 用步骤（5）的方法判断有环时，快慢指针相遇，然后将慢指针重新置为head，
 快指针停留在相遇的节点；
 这时候，都同时走，每次一步，再次相遇时，必定在进环的第一个节点。
 （可以按照圆环的路径证明）
  程序如下：
 
```c
Node * fistNodeOfLoop(Node * head){
    Node * slow = head;
    Node * fast = head;
    while ( fast != NULL && fast->next != NULL ){
        slow = slow->next;
        fast = fast->next->next;
        if ( slow == fast)
            break;
    }
    slow = head;
    while ( slow != fast ){
        slow = slow->next;
        fast = fast->next;
    }
    return slow;
 
}
```

##（7） 打印链表的倒数第K个节点


设定2个指针，先让一个走k-1步，然后一起走，每次一步；
当前面的指针走到尾时，后的指针刚好走到倒数第K个节点。
程序如下：

```c
Node * FindLastKNode ( Node * head, unsigned int k ){
    if ( head == NULL || k ==0 )
        return NULL;
 
    Node * fast = head;
    Node * slow = head;
    int i=1;
    for (;i<k;i++){
        if (fast->next!=NULL)
            fast = fast->next;
        else /* k might be larger than the length of list */
            return NULL;
    }
 
 
    while (fast->next!=NULL){
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```
 
#2. 以上都是针对一个单链表的操作，下面讨论两个单链表，
现在讨论两个单链表的相交情况


##（1）判断给定的2个单链表是否相交


 由节点指向的单一性，相交后不会再分开；即相交后的形状如Y，不是X；
可以分别遍历2个链表，只需判断最后一个节点是否一致，一致则相交，不一致则不相交；
时间复杂度为 O(m+n)


```c
bool HasCommonNodeOf2Lists ( Node * p1,Node * p2 ){
        while ( p1->next != NULL )
            p1 = p1->next;
        while ( p2->next != NULL )
            p2 = p2->next;
 
        return (p1==p2)? TRUE : FALSE;
 
}
```

##(2) 如果两个链表相交，找出第一个相交的节点


两个链表可能有长有短，可以分别遍历2个链表，计算出长度差diff；
然后，在长链表上先走diff步，再同时走，直到节点相等时停止，
这个节点即为第一个相交的节点。
程序如下：


```c
Node *  firstCommonNodeOf2Lists ( Node * head1,Node * head2 ){
        if (head1==NULL || head2==NULL)
            return NULL;
        int len1 = 0;
        int len2 = 0;
        Node * p1 = head1;
        Node * p2 = head2;
        while ( p1->next != NULL ){
             p1 = p1->next;
             len1++;
        }
 
        while ( p2->next != NULL ){
             p2 = p2->next;
             len2++;
        }
        int diff = len1 - len2;
        Node * fast = head1;
        Node * slow = head2;
        if ( len1<len2 )
            diff = len2 - len1;
            fast = head2;
            slow = head1;
 
        int i = 1;
        for (;i<=diff;i++)
            fast = fast->next;
 
        while ( fast != slow ){
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
}
```
