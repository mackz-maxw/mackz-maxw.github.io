---
title: 代码随想录 | 刷题-链表2
categories: leetcode
lang: zh-CN
---

### 24. 两两交换链表中的节点
翻转链表加强版
```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(!head || !head->next)return head;
        ListNode* d = new ListNode(-1, head);
        ListNode* n = swapPairs(head->next->next);
        ListNode* r_pair = head->next;
        d->next = r_pair;
        r_pair->next = head;
        head->next = n;
        return d->next;
    }
};
```
### 19.删除链表的倒数第N个节点
还是那个当整个链表删除时不能直接返回head的问题，多复制几个dummy head就好了
```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummy = new ListNode(-1, head);
        ListNode* d = dummy;
        ListNode* d2 = dummy;
        int len = 0;
        while(d->next){
            d = d->next;
            len++;
        }
        cout<<"len: "<<len<<endl;
        int rm = len - n-1;
        while(rm>=0){
            cout<<"rm: "<<rm<<"d2 val"<<d2->val<<endl;
            d2 = d2->next;
            rm--;
        }
        d2->next = d2->next->next;
        return dummy->next;
    }
};
```
### 面试题 02.07. 链表相交 (leetcode 160)
本来想着可能需要挨个遍历节点是否相等找到相交节点，看了题解发现可以从共同长度同时往后搜索
```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* cur_a = headA;
        ListNode* cur_b = headB;
        int len_a = 0, len_b = 0, e = 0;
        while(cur_a){
            len_a++;
            cur_a = cur_a->next;
        }
        while(cur_b){
            len_b++;
            cur_b = cur_b->next;
        }
        cur_a = headA;
        cur_b = headB;
        cout<<"len_a:"<<len_a<<"len_b"<<len_b<<endl;
        if(len_a > len_b){
            e = len_a - len_b;
            cout<<"e:"<<e<<endl;
            while(e>0){
                e--;
                cur_a = cur_a->next;
            }
        }else{
            e = len_b - len_a;
            cout<<"e:"<<e<<endl;
            while(e>0){
                e--;
                cout<<"whiling"<<endl;
                cur_b = cur_b->next;
            }
        }
        cout<<"caval:"<<cur_a->val<<"cbval:"<<cur_b->val<<endl;
        while(cur_a || cur_b){
            if(cur_a == cur_b)return cur_a;
            cur_a = cur_a->next;
            cur_b = cur_b->next;
        }
        return NULL;
    }
};
```
### 142.环形链表II
模糊记得俩指针相遇的地方有一些特性，还是得确定地记下来：
从头结点出发一个指针，从相遇节点 也出发一个指针，这两个指针每次只走一个节点， 那么当这两个指针相遇的时候就是 环形入口的节点。
两个指针都从head开始如何避免判断一开始这个点？看题解发现可以循环内先走next再判断,错误的初始化容易错过相遇点
```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* slow = head;
        ListNode* h = head;
        if(!head || !head->next)return NULL;
        ListNode* fast = head;
        while(slow && fast && fast->next){
            slow = slow->next;
            fast = fast->next->next;
            if(slow == fast){
                ListNode* b = slow;
                while(b != h){
                    b = b->next;
                    h = h->next;
                }
                return h;
            }
            
        }
        return NULL;
    }
};
```