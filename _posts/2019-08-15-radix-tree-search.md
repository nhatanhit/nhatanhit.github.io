---
layout: post
title: Radix Tree
subtitle: Radix Tree implementation
comments: true
tags: [alg]
---
1. [Implementation Radix Tree Node In Android ](https://android.googlesource.com/platform/external/smali/+/android-5.1.1_r8/util/src/main/java/ds/tree/RadixTreeNode.java)

# Golang Implementation
[Golang module radix tree source](https://gist.github.com/nhatanhit/1dee55aa9c09d60d37cc447764a9e4c8)
[main module source](https://gist.github.com/nhatanhit/1adacbca01d190661e7b8d92a40f6d16)

# Pseudo Code

```text
function searchPrefix(key, nodeSearchOn) {
    //get number of matching character between key and nodeSearchOn
    numberOfMatchingCharacter = findNumberOfMatching(nodeSearchOn, key);
    
    if (numberOfMatchingCharacter == length(key) && numberOfMatchingCharacter <= length(nodeSearchOn))
        return nodeSearchOn
    else if (nodeSearchOn == "") && (numberOfMatchingCharacter > length(nodeSearchOn) && numberOfMatchingCharacter < length(key)) 
        get remaining text number of matching chacter [numberOfMatchingCharacter, len of key]
        //iterate children nodes of nodeSearchOn, call the searchPrefix with remaining text on these children nodes 
        for(childNodes in nodeSearchOn.children) {
            searchPrefix(remainingText,childNode)
        } 
}
//append new node( key, value) to existed RadixTreeNode node 
function insert(String key, RadixTreeNode node, string value) {
    //check matching character
    int numberOfMatchingCharacters = node.getNumberOfMatchingCharacters(key);
    //check if we are begin at root node 
    if (node.key == "" || numberOfMatchingCharacters == 0 || (numberOfMatchingCharacters < key.length() && numberOfMatchingCharacters >= node.key.length())) {
        boolean flag = false;
      	//take remaining of new key and insert as a new node
        String newText = key.substring(numberOfMatchingCharacters, key.length());
        for (RadixTreeNode child : node.getChildern()) {
          	//append new node to existed child element
            if (child.key.startsWith(newText.charAt(0) + "")) {
                flag = true;
                insert(newText, child, value);
                break;
            }
        }
      //or create a new node then append to current node
      if (flag == false) {
        RadixTreeNode<T> n = new RadixTreeNode<T>();
        n.setKey(newText);
        n.setReal(true);
        n.setValue(value);
        node.getChildern().add(n);
      }
    }
    // there is a exact match just make the current node as data node
    else if (numberOfMatchingCharacters == key.length && numberOfMatchingCharacters == node.key.length) {
      if (node.real) 
      {
        //throw exception node is real, duplication key
      }
      node.real = true
      node.value = value
    }
    else if (numberOfMatchingCharacters > 0 && numberOfMatchingCharacters < node.key.length {
      RadixTreeNode<T> n1 = new RadixTreeNode<T>();
    	n1.setKey(node.getKey().substring(numberOfMatchingCharacters, node.getKey().length()));
    }
    
  }
```
