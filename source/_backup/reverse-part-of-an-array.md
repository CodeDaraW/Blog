title: JS反转数组中指定部分
date: 2016-6-26 21:14:00
categories: JavaScript
tags: [JavaScript]
---
其实这是我在网易面试时的一题面试题，翻看复习《JS高程》时想起来了这个问题，随手写下这题答案。
<!-- more -->
```JavaScript
function reversePart(toReverseArray, start = 0, end = toReverseArray.length) {
    let firstPart = toReverseArray.slice(0, start),
        toReversePart = toReverseArray.slice(start, end),
        lastPart = toReverseArray.slice(end, toReverseArray.length);

    return firstPart.concat(toReversePart.reverse(), lastPart);
}
```
函数接受三个函数，第一个是待处理的数组，第二个是要反转部分的起始位置，第三个是要反转部分的终止位置。若不传第三个参数，则返回从起始位置到末尾部分反转后的数组；若不传二三两个参数，则和`reverse()`方法等效。注意和数组的`slice()`方法类似，反转起始位置到终止位置的部分，但是不会包含结束位置的项。
