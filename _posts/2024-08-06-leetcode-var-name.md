---
layout: post
title: 力扣中的变量名会影响运行时间吗
--- 

__599 两个列表的最小索引总和__

该解法耗时 8ms，代码 CV 自官解。

如果将循环中的变量 `int j` 重命名为 `ja` 或者 `a` 或者 `cur`，在力扣上提交后，耗时变成 9ms。

这太奇怪了，，，力扣你不会干了什么奇怪的优化吧

```java
public String[] findRestaurant(String[] list1, String[] list2) {
    Map<String, Integer> index = new HashMap<>();
    for (int i = 0; i < list1.length; i++) {
        index.put(list1[i], i);
    }
    List<String> ret = new ArrayList<>();
    int indexSum = Integer.MAX_VALUE;
    for (int i = 0; i < list2.length; i++) {
        if (index.containsKey(list2[i])) {
            int j = index.get(list2[i]);
            if (i + j < indexSum) {
                ret.clear();
                ret.add(list2[i]);
                indexSum = i + j;
            } else if (i + j == indexSum) {
                ret.add(list2[i]);
            }
        }
    }
    return ret.toArray(new String[0]);
}
```
