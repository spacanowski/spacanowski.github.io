---
layout: post
title:  "Interview questions about dictionary"
date:   2016-10-24 16:22:28
categories: java interview dictionary trie
---

When you go to interviews sometimes you'll get some strange question about data structures. Most commonly it's something about dictionary. Sometimes it's 'how you would implement dictionary', sometimes it's 'find two words in dictionary' or even 'check if word is concatenation of words from dictionary'. If you think dictionary should be array/list then you're screwed. So let's fix some knowledge gaps.

## What to use for dictionary ?

When it comes to data structures you should use one of two recommended for implementing dictionary.

First is [hash table] or simply associative array or even better hash map. That's just fancy data structures names and some will use it interchangeably just to mess with you (those bastards). In Java it's simply [HashMap] or [Hashtable]. If you ever look into implementation of Hashtable you'll see that it even extends abstract class [Dictionary]. That's says a lot about if solution is correct. Implementing your own simplified version of HashMap is simple. It's just array of linked lists.

Second approach is for those that want to have instant respect on the hood. It's called [Trie]. It's not typo. It's a tree called trie. Tree trie. Someone had a little to much of fire water with green fairy to come up with this shit. On interview you'll need to listen carefully if they said tree or trie. I'm not native speaker so both of those sounds exactly the same for me. It can be confusing at times. Getting back to the point. Computational complexity for search in hash table is on average O(1). Worst case is O(n). But for trie it's length of string you'll searching for.

### Implementing your own trie

For ease of adding and searching we'll add children length field. Which will be useful during interview if you'll forget some details. This way it'll be easier to figure out what's what.

Basic structure is simple enough.

{% highlight java %}
public class Trie {
    private int childValueLength = 1;
    private Map<String, Node> children = new HashMap<>();

    private static class Node extends Trie {
        private final String value;

        private Node(String value) {
            this.value = value;
            super.childValueLength = value.length() + 1;
        }
    }
}
{% endhighlight %}

Now let's add ```add``` for adding new nodes.

{% highlight java %}
private void add(Trie node, String value) {
    if (node.childValueLength == value.length()) {
        if (!node.children.containsKey(value)) {
            node.children.put(value, new Node(value));
        } else {
            throw new IllegalStateException("Value already exists");
        }
    } else {
        String valuePart = value.substring(0, node.childValueLength);

        if (node.children.containsKey(valuePart)) {
            add(node.children.get(valuePart), value);
        } else {
            Node partNode = new Node(valuePart);

            node.children.put(valuePart, partNode);

            add(partNode, value);
        }
    }
}
{% endhighlight %}

As always you'll need to search some for values.

{% highlight java %}
private Node find(Trie node, String value) {
    if (node.childValueLength > value.length()) {
        return null;
    }

    String valuePart = value.substring(0, node.childValueLength);

    if (node.children.containsKey(valuePart)) {
        Node child = node.children.get(valuePart);

        if (valuePart.length() == value.length()) {
            return child;
        } else if (valuePart.length() < value.length()) {
            return find(child, value);
        }
    }

    return null;
}
{% endhighlight %}

For distinguishing between dictionary values and intermediate nodes you'll need to add definition or other similar field and check if it's empty on not during search. You get the idea.

For additional exercise try and implement delete method and add missing field. It'll do you good just as much as it'd helped me.

Full code example is [here].

## Summary

It's only scary when you don't know the answer. Sometime ago I've learned that optimization of application is hard if you only do language optimization and not data structure or algorithm. Now go play with other data structures.

[hash table]: https://en.wikipedia.org/wiki/Hash_table
[Hashtable]: https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html
[HashMap]: https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html
[Dictionary]: https://docs.oracle.com/javase/8/docs/api/java/util/Dictionary.html
[Trie]: https://en.wikipedia.org/wiki/Trie
[here]: https://gist.github.com/spacanowski/a5bf74c1a2df60bfd33da74c62e4bb29
