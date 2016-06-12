---
layout: post
title:  "如何检查敏感词（DFA算法）"
date:   2016-06-12 18:35:39 +0800
categories: java
---
DFA：Deterministic Finite Automaton，确定有穷自动机。什么是确定有穷自动机？请自行搜索。

假设我们我们需要检查的敏感词List为：

```
[中国, 日本, 日本人, 日本女人]
```

首先我们需要把他转换成下面形式的Map：

```
{
  日={
    isEnd=false,
    本={
      女={
        isEnd=false,
        人={
          isEnd=true
        }
      },
      isEnd=true,
      人={
        isEnd=true
      }
    }
  },
  中={
    isEnd=false,
    国={
      isEnd=true
    }
  }
}
```

其中“isEnd”标记该字符是否为最后一个字符。

接下来就是检索这个Map集合，来检查是否包含敏感词。

{% highlight java %}
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 敏感词过滤器
 */
public class SensitiveWordFilter {

    private static Map sensitiveWordMap;

    public static void main(String[] args) {
        List<String> sensitiveWords = new ArrayList<>();
        sensitiveWords.add("中国");
        sensitiveWords.add("日本");
        sensitiveWords.add("日本人");
        sensitiveWords.add("日本女人");
        SensitiveWordFilter filter = new SensitiveWordFilter();
        sensitiveWordMap = filter.convertSensitiveWordToMap(sensitiveWords);

        String text1 = "日本女人来中国了";
        String text2 = "如何成为一个码农";
        System.out.println("“"+text1+"”"+(filter.check(text1)?"":"不")+"包含敏感词");
        System.out.println("“"+text2+"”"+(filter.check(text2)?"":"不")+"包含敏感词");
    }

    /**
     * 循环text中的字符，依次调用检查敏感词方法
     * @param text
     * @return
     */
    public boolean check(String text) {

        for (int i = 0, length = text.length(); i < length; i++) {
            if (containsSensitiveWord(text, i)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 检查是否包含敏感词
     *
     * @param text
     * @param index
     * @return
     */
    private boolean containsSensitiveWord(String text, int index) {
        char word;
        Map<Object, Object> currMap = sensitiveWordMap;
        for (int i = index, length = text.length(); i < length; i++) {
            word = text.charAt(i);
            currMap = (Map<Object, Object>) currMap.get(word);
            if (currMap == null) {
                break;
            } else {
                if ((boolean) currMap.get("isEnd")) {
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 将敏感词库转换成Map格式
     *
     * @param words
     * @return
     */
    private Map<Object, Object> convertSensitiveWordToMap(List<String> words) {
        if (words == null || words.size() == 0) {
            return new HashMap<>();
        }
        sensitiveWordMap = new HashMap<>(words.size()); //初始化敏感词容器，减少扩容操作
        Map<Object, Object> _currMap;
        Map<Object, Object> _newMap;
        for (String word : words) {
            if (word == null || "".equals(word)) {
                continue;
            }
            _currMap = sensitiveWordMap;
            for (int i = 0, length = word.length(); i < length; i++) {
                char _char = word.charAt(i); //获取当前字符
                Object _temp = _currMap.get(_char);
                if (_temp == null) {
                    _newMap = new HashMap<>();
                    _newMap.put("isEnd", false);
                    _currMap.put(_char, _newMap);
                    _currMap = _newMap;
                } else {
                    _currMap = (Map<Object, Object>) _temp;
                }
                if ((length - 1) == i) {//如是最后一个字符，设置 isEnd=true
                    _currMap.put("isEnd", true);
                }
            }
        }
        return sensitiveWordMap;
    }
}
{% endhighlight %}

输出结果：

```
“日本女人来中国了”包含敏感词
“如何成为一个码农”不包含敏感词
```
