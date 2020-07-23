---
layout: post
title: Merge Sort
subtitle: Merge Sort
comments: true
---
# Merge Sort In Array
## Pseudo Code

```text
function merge(A, left , middle, right) {
    buffer1 = [];
    buffer2 = [];
    //Enqueue buffer1 from left to middle
    //Enqueue buffer2 from middle + 1 to right
    
    

    //Compare 2 buffer
    i = 0;
    j = 0;
    k = left;
    while(i < buffer1.length || j < buffer2.length) {
        //if value of buffer 1 greather than buffer 2, append value of buffer 2 to A at k index
        //value of buffer 2 greather than buffer 1, append value of buffer 1 to A at k index
        //increase k 
    }
    //Append remaining values in buffer1 to A
    //Append remaining values in buffer2  to A
    
}
function mergeSort(array A, left ,right) {
    if (left < right) {
        middle = (left + right) / 2;
        mergeSort(A,left, middle);
        mergeSort(A,middle + 1, right);
        merge(A, left, middle, right);
    }
}
```

```javascript
function merge(a, low, middle, high) {
    var i;
    var buffer1 = [];
    var buffer2 = [];
    for (var i = low; i <= middle; i++) {
        buffer1.push(a[i]);
    }
    for (var j = middle + 1; j <= high; j++) {
        buffer2.push(a[j]);
    }
    var k = low;
    i = 0;
    j = 0;
    while (i < buffer1.length && j < buffer2.length) {
        if (buffer1[i] <= buffer2[j]) {
            a[k] = buffer1[i];
            i++
        } else {
            a[k] = buffer2[j];
            j++;
        }
        k++;
    }
    //append remaining value of buffer 1
    if (i < buffer1.length) {
        while (i <= buffer1.length - 1) {
            a[k] = buffer1[i];
            i++;
            k++;
        }
    }
    //append remaining value of buffer 2
    if (j < buffer2.length) {
        while (j <= buffer2.length - 1) {
            a[k] = buffer2[j];
            j++;
            k++;
        }
    }
}
function mergeSort(a, low, high) {
    var i;
    var middle;
    if (low < high) {
        middle = parseInt((low + high) / 2);
        mergeSort(a, low, middle);
        mergeSort(a, middle + 1, high);
        merge(a, low, middle, high);
    }
}
```

# Merge Sort In Linked List
## Pseudo Code

```text
function sortedMerge(left, right) {
    //if left or right is null , return each other
    //if data of left < data right, result = left, call sortedMerge with left.next and right
    //if data of left > data right, result = right, call sortedMerge with right.next and left
    //return result
}
function middle(node) {
    //iterate linked list begin from 'node'
    //slow node iterate next element
    //fast node iterate 2 next elements
    //return slow node
}
function mergeSort(head) {
    //base case of this recursion is head = null
    //find the middle begin from head
    middle =  middle(head)
    nextToMiddle = middle.next;
    middle.next = null;
    //get a ordered linked list begin from head
    leftLinkedList =  mergeSort(head);
    //get a order linked list begin from next to middle
    rightLinkedList = mergeSort(nextToMiddle);
    result =  sortedMerge(leftLinkedList, rightLinkedList);
    return result;

}
```

## Javascript Code

```javascript

function sortedMerge(a,b) {
    var result = null;
    if (a == null) {
        return b;
    }
    if (b == null) {
        return a;
    }
    if (a.data <= b.data) {
        result = a;
        result.next = sortedMerge(a.next,b);
    } else {
        result = b;
        result.next = sortedMerge(a, b.next);
    }
    return result;


}
function mergeSort(head) {
    
    // Base case : if head is null 
    if (head == null || head.next == null) { 
        return head; 
    } 

    // get the middle of the list 
    var middle = getMiddle(head);
    var nextofmiddle = middle.next; 

    // set the next of middle node to null 
    middle.next = null; 

    // Apply mergeSort on left list 
    var left = mergeSort(head);
    // Apply mergeSort on right list 
    var right = mergeSort(nextofmiddle);
    // Merge the left and right lists 
    var sortedlist = sortedMerge(left, right); 
    return sortedlist; 
}
function getMiddle(head) {
    if (head == null) {
        return head;
    }
    slow = head, fast = head; 
  
    while (fast.next != null && fast.next.next != null) { 
        slow = slow.next; 
        fast = fast.next.next; 
    } 
    return slow;
}
```