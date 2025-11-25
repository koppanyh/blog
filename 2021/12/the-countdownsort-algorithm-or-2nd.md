# The CountdownSort Algorithm (or the 2nd worst sorting algorithm)

Monday, December 20, 2021

---

I was thinking about the [SleepSort algorithm](https://www.geeksforgeeks.org/sleep-sort-king-laziness-sorting-sleeping/) and how you might make it less unreliable. The problem with SleepSort is that if you try to make it faster by scaling down the sleep delays to milli/microsecond levels, then you run into some rather undeterministic behavior by the OS's scheduler in that it can't do exact millisecond-scale timing of when the threads should run. This is a normal aspect of non-realtime operating systems, but it makes SleepSort unreliable along with just plain slow. The solution is to operate on the second-scale (process schedulers do have the resolution for this), but then sorting a list with huge numbers could literally take more time than humans have had on Earth.

What if we could speed things up *and* have them be reliable and deterministic? We can; enter CountdownSort.

The idea is that instead of making processes with delays, you make nodes in a linked list with a counter. That counter is initialized with the same value as the delay in SleepSort, but we iterate through the linked list and decrement the counter for every node in each iteration. Once a node reaches 0, then it is pulled off the linked list and added to the end of a sorted linked list. The process repeats until there are no more nodes in the unsorted linked list.

Here I've written the basic algorithm in C++ (and man it's been a loooong looong time since I've written something like this in C++).
```c++
#include <iostream>
using namespace std;

class Node {
    void print2() {
        cout << value << " ";
        if(next != NULL) next->print2();
    }
    public:
    Node* next = NULL;
    int counter = 0;
    int value;
    Node(int val) {
        value = val;
        counter = val; //counter value defined here
    }
    ~Node() { if(next != NULL) delete next; }
    Node* append(int val) { return next = new Node(val); }
    void print() {
        cout << value << " ";
        if(next != NULL) next->print2();
        cout << endl;
    }
};

Node* countdownSort(Node* root) {
    Node* sortedRoot = NULL;
    Node* sortedCur = NULL;
    while(root != NULL) { //repeat until nothing left to sort
        Node* prev = NULL;
        Node* cur = root;
        while(cur != NULL) { //iterate through the list
            if(--cur->counter <= 0) { //add to sorted list
                //pop from old
                if(prev == NULL) root = cur->next;
                else prev->next = cur->next;
                //add to new
                if(sortedCur == NULL) sortedRoot = sortedCur = cur;
                else { sortedCur->next = cur; sortedCur = cur; }
                cur = cur->next;
                sortedCur->next = NULL;
            } else {
                prev = cur;
                cur = cur->next;
            }
        }
    }
    return sortedRoot;
}

int main(){
    Node* root = new Node(3); // initialize our list
    Node* cur = root;
    cur = cur->append(7);
    cur = cur->append(5);
    cur = cur->append(2);
    root->print(); //print: 3 7 5 2
    root = countdownSort(root); //sort
    root->print(); //print: 2 3 5 7
}
```
You can see that in the main function we add the numbers in this order: 3, 7, 5, 2. Then we print it out, giving us: 3 7 5 2. Then we use CountdownSort to sort it and end up with: 2 3 5 7. Success!

The space complexity is O(n) since we only use the nodes that exist, so no further  memory is used. But even though it works, it's still a terrible algorithm! There are 2 iterations needed to get to the point where we move 2 to the sorted list in the beginning. Then after 3 there are an additional 2 iterations to move the next value, 5, to the sorted list. Same for the value 7. So we eventually get to the point where each value is added to the sorted list, but we skip any steps in the middle! So if our list of values looked like [100(100), 2(2)] (where the value within the parenthesis is the counter for the real value outside of the parenthesis), then there would be 98 useless iterations in the process of sorting just 2 numbers, or almost the same cost as sorting 100 values (minus the part where we iterate through multiple values). As such, the worst-case time complexity is O(n*m) where n is the number of nodes and m is the largest counter value. This example takes 17 iterations total.

Welp, we made the 2nd worst sorting algorithm (it runs way faster than SleepSort and only wastes a bunch of CPU power in the process), so our job here is done. Or maybe not, it turns out you can actually optimize this into a half decent sorting algorithm.

We said that you need 2 iterations in order for the counter to reach the first value, but what if you could offset all the counters in the entire list by the right amount to make it so the smallest item would be added to the sorted list on the first iteration without ruining the relative offsets of each item. Instead of the counters starting at [100(100), 2(2)] (as per our previous example), we start them at [100(99), 2(1)] so the 2nd item (our smallest) would be added to the sorted list on the first iteration.

Sounds good, but if there's a gap then the next smallest item, 100(99), would still need more than one more iteration to be added to the sorted list. So what if on each iteration we just keep track of the current smallest value in the list? We can do that easily since we have to iterate through the remaining items anyways. So each iteration will always result in the addition of the next smallest item, and no wasted iterations would exist. Take our new list, [98(98), 100(100), 2(2)], find the smallest, and offset the whole list so it's [98(97), 100(99), 2(1)]. We pop item 3 since that counter reaches 0 and are left with [98(97), 100(99)], our last iteration tells us the smallest counter is 97 so we offset the list so it's now [98(1), 100(3)] and pop the first item when the counter reaches 0 and send that off to the sorted list. Our list is now [100(3)] and since we know 3 is the smallest number, we offset the list to become [100(1)] and then send that to the sorted list since the counter reaches 0.

If we add that modification to the function, then we end up with the following:
```c++
#include <iostream>
#include <climits>
using namespace std;

class Node {
    void print2() {
        cout << value << " ";
        if(next != NULL) next->print2();
    }
    public:
    Node* next = NULL;
    int counter = 0;
    int value;
    Node(int val) {
        value = val;
        counter = val; //counter value defined here
    }
    ~Node() { if(next != NULL) delete next; }
    Node* append(int val) { return next = new Node(val); }
    void print() {
        cout << value << " ";
        if(next != NULL) next->print2();
        cout << endl;
    }
};

//find the smallest counter in the list
int smallestCounter(Node* cur) {
    int smallest = cur->counter;
    while(cur != NULL) {
        if(cur->counter < smallest) smallest = cur->counter;
        cur = cur->next;
    }
    return smallest;
}

Node* countdownSort(Node* root) {
    Node* sortedRoot = NULL;
    Node* sortedCur = NULL;
    int offset = smallestCounter(root);
    while(root != NULL) { //repeat until nothing left to sort
        Node* prev = NULL;
        Node* cur = root;
        int newSmallest = INT_MAX;
        while(cur != NULL) { //iterate through the list
            cur->counter -= offset;
            if(cur->counter <= 0) { //add to sorted list
                //pop from old
                if(prev == NULL) root = cur->next;
                else prev->next = cur->next;
                //add to new
                if(sortedCur == NULL) sortedRoot = sortedCur = cur;
                else { sortedCur->next = cur; sortedCur = cur; }
                cur = cur->next;
                sortedCur->next = NULL;
            } else {
                if(cur->counter < newSmallest)
                    newSmallest = cur->counter;
                prev = cur;
                cur = cur->next;
            }
        }
        offset = newSmallest;
    }
    return sortedRoot;
}

int main(){
    Node* root = new Node(3); // initialize our list
    Node* cur = root;
    cur = cur->append(7);
    cur = cur->append(5);
    cur = cur->append(2);
    root->print(); //print: 3 7 5 2
    root = countdownSort(root); //sort
    root->print(); //print: 2 3 5 7
}
```
The only thing that was changed was adding the function to find the smallest counter before the first iteration, keeping track of the new smallest counter (but only if it's not being removed from the list), and then counting down by the entire offset instead of just by 1. That last one is actually better than trying to offset everything to make the smallest counter 1 and then using the decrement to reach 0 since you already know how much you have to subtract to reach 0 anyways. This slight modification to the algorithm results in the same sorting taking only 10 iterations total, with a new worst-case (and average case if all numbers are unique) time complexity of O(n<sup>2</sup>) (actually more like O(n<sup>2</sup>-n), which given an input of 4 actually returns 10, but we don't count the little n).

And now we've got an almost good sorting algorithm!

At this point it turns out to be basically [SelectionSort](https://en.wikipedia.org/wiki/Selection_sort) (which is a real-world sorting algorithm) but using linked lists and wasting memory on a counter and finding the minimum value by subtraction rather than comparisons. Since we're wasting memory, it would've been worth making it a doubly linked list so we could find the previous node easier and just write an extra method that would automatically bridge the previous and next nodes together and release this node from it's linking responsibilities, but I wrote the example code as I wrote this post, so that didn't happen (and it's late at night, so still not gonna happen).

The limitations of this algorithm is that the counter requires positive numbers, so if you want to use 0 or negative numbers then you have to somehow make sure the counter is always positive and not 0. One way to do this is to make the counter an unsigned int and add `INT_MAX` to it so the lowest negative numbers start at 0 (probably an off-by-one error waiting to happen here) and the lowest positive numbers start at around 2 billion (halfway up `UINT_MAX`). Also this probably wouldn't work on strings or complex objects since you'd have to find a way to make counter values that maintain relative ordering with each other, and that probably requires sorting to generate new values, which defeats the entire purpose.

This post started as a joke, but ended up being an actually serious analysis of an algorithm. The original Discord chat that prompted this post and explored the time complexities and optimizations is much more entertaining. Oh well, at least we had fun 😂

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2021/12/20 at 11:03 PM PDT</sub>

<sub>[Permalink](https://blog.kh-labs.org/2021/12/the-countdownsort-algorithm-or-2nd)</sub>
