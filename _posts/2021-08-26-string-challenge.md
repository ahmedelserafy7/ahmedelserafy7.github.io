---
title: String Challenge Explained
date: 2021-08-26
permalink: /posts/2021/08/string-challenge
excerpt: "Let's solve this challenge."
tags:
  - Coding
  - Programming
  - Problem Solving
---

Difficulty: **Medium**

{% capture notice-text %}
Have the function `StringChallenge(str)` read the `str` parameter being passed which will a string of HTML [DOM elements](https://www.w3schools.com/jsref/dom_obj_all.asp) and plain text. The elements that will be used are: b, i, em, div, p. For example if `str` is "<div>\<b><p>hello world</p></b></div>" then this string of DOM elements is nested correctly so your program should return the string **true**.

If a string is not nested correctly, return the first element encountered where, if changed into a different element, would result in a properly formatted string. If the string is not formatted properly, then it will be one element that needs to be changed. For example: if `str` is "<div>\<i>hello</i>world</b>" then  your program should return the string <b>div</b> because if the first <div> element were changed into a \<b>, the string would be properly formatted.

### Examples

```swift
    Input: "<div><div><b></b></div></p>"
    Output: div
```
``` swift
    Input: "<div>abc</div><p><em><i>test test test</b></em></p>"
    Output: i
```

{% endcapture %}

<div class="notice--danger">

{{ notice-text | markdownify }}

</div>

## What should we know before solving this challenge

This Challenge requires a basic understanding of stack:  a linear data structure in which elements can be inserted and deleted only from on side of the list. Stack follows the LIFO (Last In First Out) principle, i.e., the element inserted at the last is the first element to come out. So as the string of opening tag comes in, we can push it onto the stack, so that when it comes to reading back elements the most recently opened tag is the one we read first.

## Things you should know about Swift

Swift doesn’t have a native stack data type, but we can get the same behavior just by using an
array and using its `append()` and `popLast()` methods.

## First steps

First things first, we need to filter/strip out text between brackets like "test test test", and also remove angle brackets because it's not important here. If you want to keep it you can, but you'll sustain angle brackets non-removal pain.

So we need to check if there's any text between brackets, we'll remove it. First we'll convert `input` string to array to get the benefit of index.

```swift
    var inputArr = Array(input)
    let n = inputArr.count
    
    for i in 0..<(n - 1) {
        if inputArr[i] == ">" {
            if (i + 1) == n {
                break
            } else if inputArr[i + 1] == "<" {
                continue
            } else {
                while inputArr[i + 1] != "<" {
                    inputArr.remove(at: i+1)
                }
            }
        } else{
            continue
        }
    }
```

If you look at this code first you may think it's hard to understand, but once you get it, you'll find it's straightforward. First, we need to get the length of the input array to loop through all the items in the input string. Why (n - 1)? because we'll check the next element if it has opening bracket, so if this array just go up to the last element in the array, then we're going to get an out of bounds error on the very last element because there's no element after the very last element in the array. So we need to make sure that this `inputArr.count` - 1.

Now that we need to check if the character is closing angle bracket. If so, we'll check first if the next character equals the count of the `inputArr` that means there's no character after this closing angle bracket in the array and that also means it's the last element in our array, so we'll terminate the execution of a `for...in` loop immediately by `break`.

<blockquote>
Swift-speak aside: Although `break` isn’t required in Swift, you can use a `break` statement to match and ignore a particular case or to break out of a matched case before that case has completed its execution.
</blockquote>

Once we've done that, the next step is to go checking the opposite angle bracket which is opening one if it exsits we're okay means there's no text between angle brackets, so we need to head over to the next element. But if it's not the very last element in the array or opening angle we get to know we have text between angle brackets that we need to remove.

To **Remove** this text: We can do that by iterating over the text from the element/character that is next to closing bracket until we find opening bracket. All this text we'll remove it from the `inputArr`.

After we get to that point we need to go through other elements by `continue`, because we know it's opening bracket for sure, and has tag element inside it, so we don't need to touch them.

![](https://i0.wp.com/media1.giphy.com/media/LPn77YyDIqfhm/giphy.gif){: .align-center }

Now you can wind back to string, and convert `inputArr` to `newInput` string.

```swift
    Input: "<div><i>hello</i>world</b>"
    
    let newInput = String(inputArr)
    print(newInput)
    
    Output: <div><i></i></b>
```
    
This will, unsurprisingly, take you to get rid of the angle brackets, because we'll no longer need them. You can remove the closing brackets by replacing them with comma, filter out the opening ones, and remove the last closing bracket in the text string by `dropLast()`

```swift
    let tagElementsString = newInput.replacingOccurrences(of: ">", with: ",").filter{$0 != "<"}.dropLast()
    print(tagElementsString)
    
    Output: div,i,/i,/b
```

You'll need to use `components` to get each tag element individually, and you can do that by separating these elements by comma:

```swift
    let tagElements = tagElementsString.components(separatedBy: ",")
    print(tagElements)
    
    Output: ["div", "i", "/i", "/b"]
```

Once that's done, we need to declare the relationships between opening and closing tags, so Swift knows which closing tag goes with each opening tag. The easiest way to do that with dictionary, where the opening tag is the dictionary key and the closing tag is the value:

```swift
    let tagDict: [String: String] = ["div": "/div", "p": "/p", "i": "/i", "em": "/em", "b": "/b"]
```

Using this approach, when we meet a string like "/i" we can read the last used opening tag element from our stack, and check that `tagDict[key]`(which is the value of that key in the `tagDict`) matches "/i". If not, it'll return the key tag element the one that needs to be changed and that's it.

```swift
    var keysStack = [String]()
    
    for tag in tagElements {
    
        // Check if the element is Opening tag
        if tagDict.keys.contains(tag) {
            keysStack.append(tag)
        } else {
            // Check if it's Closing tag
            if let key = keysStack.popLast() {
                if tagDict[key] != tag {
                    return key
                }
                
            }
        }
        
    }
    
    return "true"
```

First, we created `keysStack` a stack or rather array to contain the opening tags. Then loop through `tagElements` array and check if the tag element is opening or closing one. If it's opening tag, we'll put it into `keysStack`. If it's closing tag, we'll pull off our last opening tag from `keysStack`, and check that `tagDict[key]`, which is the value of the last opening tag/key in the stack, matches our current closing tag. If not, we'll get that key. At this point if you get out of the `for...loop` the stack should be empty then you can  return **true**, but if you want to be pedantic you can check if the stack becomes empty by:

```swift
    return "\(keysStack.count == 0)"
```

![](https://i.imgur.com/jkfhYOS.gif){: .align-center }

Congrats, you've done it!
