---
title: Algorithms, Part I - Week 2 - Deques and Randomized Queues
date: 2017-05-06 16:44:57
categories: MOOC
tags:
	- stack
	- linkedlist
---

来自 Coursera 上普林斯顿大学的 Algorithms, Part I 课程的第二周编程作业[Deques and Randomized Queues](http://coursera.cs.princeton.edu/algs4/assignments/queues.html)<!--more-->

---

# 分析

链表和堆栈的运用

# 答案

Deque.java

```java
import java.util.Iterator;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;

public class Deque<Item> implements Iterable<Item>
{
    private int n;
    private Node first, last;

    private class Node
    {
        private Item item;
        private Node prev;
        private Node next;
    }

    // construct an empty deque
    public Deque()
    {
        first = last = null;
        n = 0;
    }

    // is the deque empty?
    public boolean isEmpty()
    {
        return n == 0;
    }

    // return the number of items on the deque
    public int size()
    {
        return n;
    }

    // add the item to the front
    public void addFirst(Item item)
    {
        if (item == null) throw new java.lang.NullPointerException();
        Node oldfirst = first;
        first = new Node();
        first.item = item;
        first.next = oldfirst;
        if (isEmpty())     last = first;
        else             oldfirst.prev = first;
        n++;
    }

    // add the item to the end
    public void addLast(Item item)
    {
        if (item == null) throw new java.lang.NullPointerException();
        Node oldlast = last;
        last = new Node();
        last.item = item;
          last.prev = oldlast;
          if (isEmpty())     first = last;
        else             oldlast.next = last;
          n++;
    }

    // remove and return the item from the front
    public Item removeFirst()
    {
        if (isEmpty()) throw new java.util.NoSuchElementException();
        Node oldfirst = first;
        first = first.next;
        Item item = oldfirst.item;
        oldfirst = null;
        n--;
        return item;
    }

    // remove and return the item from the end
    public Item removeLast()
    {
        if (isEmpty()) throw new java.util.NoSuchElementException();
        Node oldlast = last;
        last = last.prev;
        Item item = oldlast.item;
        oldlast = null;
        n--;
        return item;
    }

    // return an iterator over items in order from front to end
    public Iterator<Item> iterator()
    {
        return new ListIterator();
    }

    private class ListIterator implements Iterator<Item>
    {
        private Node current;

        public Item next()
        {
            if (isEmpty()) throw new java.util.NoSuchElementException();

            current = first;
            Item item = current.item;
            current = current.next;
            return item;

        }

        public void remove()
        {
            throw new java.lang.UnsupportedOperationException();
        }

        public boolean hasNext() {

            return n > 0;
        }
    }

    // unit testing (optional)
    public static void main(String[] args)
    {
           Deque<Integer> deque = new Deque<Integer> ();
           deque.addLast(1);
           deque.addLast(2);
           StdOut.println(deque.removeLast());
    }
}
```

RandomizedQueue.java

```java
import java.util.Iterator;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;

public class RandomizedQueue<Item> implements Iterable<Item>
{
    private int n = 0;
    private Item[] items;

    // construct an empty randomized queue
    public RandomizedQueue()
    {
        items = (Item[]) new Object[2];
    }

    // is the queue empty?
    public boolean isEmpty()
    {
        return n == 0;
    }

    // return the number of items on the queue
    public int size()
    {
        return n;
    }

    // add the item
    public void enqueue(Item item)
    {
        if (item == null) throw new java.lang.NullPointerException();
        if (n == items.length) resize(2 * items.length);
        items[n++] = item;
    }

    // remove and return a random item
    public Item dequeue()
    {
        if (isEmpty()) throw new java.util.NoSuchElementException();

        int index = StdRandom.uniform(n);
        Item item = items[index];
        items[index] = items[n-1];
        items[n-1] = null;
        n--;
        if (n > 0 && n == items.length/4) resize(items.length/2);
        return item;
    }

    // return (but do not remove) a random item
    public Item sample()
    {
        if (isEmpty()) throw new java.util.NoSuchElementException();
        int index = StdRandom.uniform(n);
        return items[index];
    }

    private void resize(int capacity)
    {
        Item[] copy = (Item[]) new Object[capacity];
        for (int i = 0; i < n; i++)
        {
            copy[i] = items[i];
        }
        items = copy;
    }

    // return an independent iterator over items in random order
    public Iterator<Item> iterator()
    {
        return new ListIterator();
    }

    private class ListIterator implements Iterator<Item>
    {
        private int i = n;

        public boolean hasNext()
        {
            return i > 0;
        }

        public void remove()
        {
            throw new java.lang.UnsupportedOperationException();
        }

        public Item next()
        {
            if (isEmpty()) throw new java.util.NoSuchElementException();

            int index = StdRandom.uniform(n);
            return items[index];
        }
    }

    // unit testing (optional)
    public static void main(String[] args)
    {
        RandomizedQueue<Integer> rq = new RandomizedQueue<Integer> ();
        Iterator<Integer> it = rq.iterator();
        rq.enqueue(1);
        rq.enqueue(2);
        rq.enqueue(3);
        rq.enqueue(4);
        rq.enqueue(5);
        rq.enqueue(6);
        StdOut.println("Size: " + rq.size());
        StdOut.println(rq.dequeue());
        StdOut.println("Size: " + rq.size());
        StdOut.println(it.next());
//        StdOut.println(rq.sample());
//        StdOut.println(rq.sample());
    }
}
```

Permutation.java

```java
import edu.princeton.cs.algs4.StdIn;
import edu.princeton.cs.algs4.StdOut;

public class Permutation {
    public static void main(String[] args)
    {
        int k = Integer.parseInt(args[0]);
        RandomizedQueue<String> deque = new RandomizedQueue<String>();
        while (!StdIn.isEmpty()) {
            deque.enqueue(StdIn.readString());
        }
        for (int i = 0; i < k; i++) {
            StdOut.println(deque.dequeue());
        }
    }
}
```

---
