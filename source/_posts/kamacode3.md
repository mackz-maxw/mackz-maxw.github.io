---
title: 代码随想录 | 刷题-链表
categories: leetcode
lang: zh-CN
---

### 203.移除链表元素
这回刷题的思路总算对了，但是一开始直接返回head,没有考虑到万一整个链表全部需要删除，原有的链表相当于没动。设置了dummy head就解决了。
```cpp
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode* n = new ListNode(0, head);
        ListNode* dum = n;
        while(n && n->next){
            if(n->next->val == val){
                // cout<<"next val eq. cur val: "<<n->val<<endl;
                ListNode* link = n->next->next;
                n->next = link;
            }else{
                n = n->next;
            }
            
        }
        return dum->next;
    }
};
```
### 707.设计链表
设计了一个单链表  
首先是结构体命名总是写错，应该注意检查这种问题。  
dummy node 也需要在构造函数外声明才可以在其它函数中调用；  
然后是尾部插入节点不需要再声明一个尾节点，直接插入即可，定义尾dummy code需要考虑的边界条件太多。
```cpp
class MyLinkedList {
private:
    struct ListNode{
        int val;
        ListNode* next;
        ListNode(int v): val(v),next(nullptr){}
        ListNode(int v, ListNode* n): val(v), next(n){}
    };
    ListNode* dummy;
public:
    MyLinkedList() {
        dummy = new ListNode(-1);
    }
    
    int get(int index) {
        if(!dummy->next) return -1;
        ListNode* d = dummy;
        cout<<"get ind:"<<index<<endl;
        for(;index>=0;index--){
            d = d->next;
            if(!d)return -1;
            cout<<"index:"<<index<<"getfor+1";
        }
        cout<<endl;
        // if(d->val == -1)return -1;
        return d->val;
    }
    
    void addAtHead(int val) {
        addAtIndex(0, val);
    }
    
    void addAtTail(int val) {
        // if(dummy->next)
        ListNode* d = dummy;
        while(d->next){
            d = d->next;
        }
        d->next = new ListNode(val);
    }
    
    void addAtIndex(int index, int val) {
        ListNode* d = dummy;
        for(int i = index - 1;i>=0;i--){
            d = d->next;
            if(!d)return;
        }
        // if(!d || !d->next)return;
        ListNode* n = new ListNode(val);
        ListNode* link = d->next;
        d->next = n;
        n->next = link;
    }
    
    void deleteAtIndex(int index) {
        ListNode* d = dummy;
        for(int i = index - 1;i>=0;i--){
            d = d->next;
            if(!d)return;
        }
        if(!d->next)return;
        d->next = d->next->next;
    }
};
```
### 206.反转链表
稀里糊涂地写对了...看了题解，发现这种做法相当于从后往前反转列表，结合以前的debug经验推导了一下这段代码会如何运行，有些理解了
```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(!head || !head->next)return head;
        ListNode* n = head->next;
        ListNode* h = reverseList(n);
        head->next = nullptr;
        n->next = head;
        return h;
    }
};
```