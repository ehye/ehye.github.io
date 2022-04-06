---
title: Algorithms, Part I - Week 5 - Kd-Trees
date: 2017-05-21 17:26:13
categories: MOOC
tags:
    - KdTree
---

来自 Coursera 上普林斯顿大学的 Algorithms, Part I 课程的第五周编程作业[Kd-Trees](http://coursera.cs.princeton.edu/algs4/assignments/kdtree.html)<!--more-->

---

## 分析

编写数据类型以表示单位正方形中的一组点 (所有点都具有0和1之间的 x 和 y 坐标)

使用2维树进行范围搜索 (查找查询矩形中包含的所有点) 和最近的邻居搜索 (查找与查询点最近的点)

2维树有许多应用，从将天文物体分类到计算机动画，再到加速神经网络，挖掘数据到图像检索。

{% asset_img kdtree-ops.png" %}

**Geometric primitives.** 定义了图元的坐标表示方法
{% asset_img RectHV.png %}

主要是完成 API 里的 draw(), range(), nearest() 方法

题目给了 Point2D, RectHV 这两种数据类型以使用，要提交的 PointSET.java 使用蛮力运算，KdTree.java 则使用 2d-Trees 来运算，题目就是完成这两个文件

### Node data type

根据[CheckList](http://coursera.cs.princeton.edu/algs4/checklists/kdtree.html)所给的结点参考模型进行一些修改

```java
private static class Node {
    private boolean isVertical;
    private Point2D p;
    // the left/bottom subtree
    private Node lb;
    // the right/top subtree
    private Node rt;

    public Node(Point2D p, boolean isVertical) {
        this.p = p;
        this.isVertical = isVertical;
        this.lb = null;
        this.rt = null;
    }
}
```

### Writing KdTree

可以从编写 isEmpty() 和 size() 开始，再写 insert()，再 写 contains() 并测试 insert()是否可用
要注意的是, insert() 和 contains() 的写法要使用 *private helper methods*（书399页），并增加一个 *boolean orientation* 作为这些帮助器方法的参数

**2d-tree implementation.** 编写一个可变数据类型 KdTree, 它使用2维树实现和蛮力运算相同的 api
{% asset_img kdtree-insert.png %}

注意的是，垂直分割线段是红色，垂直分割线段是蓝色，第一条一定是垂直分割，所以用一个布尔变量 isVertical 来识别线段颜色

**Range search.** 思想在课件里给出了，主要就是递归以，而且要分4种情况（垂直线左，垂直线右，水平线下，水平线上）进行回溯剪枝，这样可以排除不可能的子树

**Nearest neighbor search.** 从根结点开始递归搜索，更新候选点，同样使用剪枝

## 答案

PointSET.java

```java
public class PointSET {

    private SET<Point2D> set;
    private ArrayList<Point2D> result;

    // construct an empty set of points
    public PointSET() {
        set = new SET<Point2D>();
    }

    // is the set empty?
    public boolean isEmpty() {
        return set.size() == 0;
    }

    // number of points in the set
    public int size() {
        return set.size();
    }

    // add the point to the set (if it is not already in the set)
    public void insert(Point2D p) {
        if (p == null)             throw new java.lang.NullPointerException();
        if (!set.contains(p)) set.add(p);
    }

    // does the set contain point p?
    public boolean contains(Point2D p) {
        if (p == null)
            throw new java.lang.NullPointerException();

        return set.contains(p);
    }

    // draw all points to standard draw
    public void draw() {
        // drawing the points
        StdDraw.setPenColor(StdDraw.BLACK);
        StdDraw.setPenRadius(0.01);

        for (Point2D point : set) {
            StdDraw.point(point.x(), point.y());
        }
    }

    // all points that are inside the rectangle
    public Iterable<Point2D> range(RectHV rect) {
        if (rect == null) {
            throw new java.lang.NullPointerException();
        }

        result = new ArrayList<Point2D>();

        for (Point2D point : set) {
            if (rect.contains(point))
                result.add(point);
        }
        return result;
    }

    // a nearest neighbor in the set to point p; null if the set is empty
    public Point2D nearest(Point2D p) {
        if (p == null)         throw new java.lang.NullPointerException();
        if (set.isEmpty())     return null;

        double min = Double.MAX_VALUE;
        Point2D result = null;

        for (Point2D point : set) {
            double distence = Point.distanceSq(p.x(), p.y(), point.x(), point.y());
            if (distence < min) {
                min = distence;
                result = point;
            }
        }
        return result;
    }

    // unit testing of the methods (optional)
    public static void main(String[] args) {}
}
```

PointSET.java

```java
public class KdTree {

    private static class Node {
        private boolean isVertical;
        private Point2D p;
        // the left/bottom subtree
        private Node lb;
        // the right/top subtree
        private Node rt;

        public Node(Point2D p, boolean isVertical) {
            this.p = p;
            this.isVertical = isVertical;
            this.lb = null;
            this.rt = null;
        }
    }

    private int size;
    private Node root;
    private final RectHV RECT = new RectHV(0, 0, 1, 1);

    // construct an empty set of points
    public KdTree() {
        this.root = null;
        this.size = 0;
    }

    // is the set empty?
    public boolean isEmpty() {
        return this.size == 0;
    }

    // number of points in the set
    public int size() {
        return this.size;
    }

    // add the point to the set (if it is not already in the set)
    public void insert(Point2D p) {
        if (p == null) throw new java.lang.NullPointerException();
        this.root = insert(root, p, true);
    }

    private Node insert(Node node, Point2D p, boolean isVertical) {
        // if it is not in the set, create new node
        if (node == null) {
            size++;
            return new Node(p, isVertical);
        }

        // already in, return it
        if (node.p.equals(p))
            return node;

        // else insert it
        if (node.isVertical && p.x() < node.p.x() || !node.isVertical && p.y() < node.p.y()) {
            node.lb = insert(node.lb, p, !node.isVertical);
        } else {
            node.rt = insert(node.rt, p, !node.isVertical);
        }
        return node;
    }

    // does the set contain point p?
    public boolean contains(Point2D p) {
        if (p ==null) return false;
        return contains(root, p, false);
    }

    private boolean contains(Node node, Point2D p, boolean orientation) {
        int cmp = p.compareTo(node.p);
        if      (cmp < 0) return contains(node.lb, p, !orientation);
        else if (cmp > 0) return contains(node.rt, p, !orientation);
        else return true;
    }

    // draw all points to standard draw
    public void draw() {
        StdDraw.setScale(0, 1);
        StdDraw.setPenColor(StdDraw.BLACK);
        StdDraw.setPenRadius();

        RECT.draw();
        draw(root, RECT);
    }

    private void draw(Node node, RectHV rect) {
        if (node == null) return;

        // drawing the points
        StdDraw.setPenColor(StdDraw.BLACK);
        StdDraw.setPenRadius(0.01);
        StdDraw.point(node.p.x(), node.p.y());

        // drawing the splitting lines
        StdDraw.setPenRadius();
        if (node.isVertical) {
            // vertical
            StdDraw.setPenColor(StdDraw.RED);
            StdDraw.line(node.p.x(), rect.ymin(), node.p.x(), rect.ymax());
        }
        else {
            // horizontal
            StdDraw.setPenColor(StdDraw.BLUE);
            StdDraw.line(rect.xmin(), node.p.y(), rect.xmax(), node.p.y());
        }

        // recursively draw children
        draw(node.lb, leftRect(rect, node));
        draw(node.rt, rightRect(rect, node));
    }

    private RectHV leftRect(final RectHV rect, final Node node) {
        if (node.isVertical) {
            return new RectHV(rect.xmin(), rect.ymin(), node.p.x(), rect.ymax());
        } else {
            return new RectHV(rect.xmin(), rect.ymin(), rect.xmax(), node.p.y());
        }
    }

    private RectHV rightRect(final RectHV rect, final Node node) {
        if (node.isVertical) {
            return new RectHV(node.p.x(), rect.ymin(), rect.xmax(), rect.ymax());
        } else {
            return new RectHV(rect.xmin(), node.p.y(), rect.xmax(), rect.ymax());
        }
    }

    // all points that are inside the rectangle
    public Iterable<Point2D> range(RectHV rect) {
        final TreeSet<Point2D> rangeSet = new TreeSet<Point2D>();
        range(root, RECT, rect, rangeSet);
        return rangeSet;
    }

    private void range(final Node node, final RectHV qrect, final RectHV rect, final TreeSet<Point2D> rangeSet) {
        if (node == null) return;

        if (rect.intersects(qrect)) {                                    // if query rect is in rectangle
            final Point2D p = new Point2D(node.p.x(), node.p.y());
            if (rect.contains(p)) rangeSet.add(p);                // find the point

            // pruning rule
            if (node.isVertical) {
                if (qrect.xmax() < p.x())
                    range(node.lb, leftRect(qrect, node), rect, rangeSet);
                if (qrect.xmax() > p.x())
                    range(node.rt, rightRect(qrect, node), rect, rangeSet);
                if (qrect.contains(p)) {
                    range(node.lb, leftRect(qrect, node), rect, rangeSet);
                    range(node.rt, rightRect(qrect, node), rect, rangeSet);
                }
            }
            else {
                if (qrect.ymax() < p.y())
                    range(node.lb, leftRect(qrect, node), rect, rangeSet);
                if (qrect.ymax() > p.y())
                    range(node.rt, rightRect(qrect, node), rect, rangeSet);
                if (qrect.contains(p)) {
                    range(node.lb, leftRect(qrect, node), rect, rangeSet);
                    range(node.rt, rightRect(qrect, node), rect, rangeSet);
                }
            }
        }
    }

    // a nearest neighbor in the set to point p; null if the set is empty
    public Point2D nearest(Point2D p) {
        if (root == null) return null;
        Point2D retp = null;
        double mindis = Double.MAX_VALUE;
        Queue<Node> queue = new Queue<Node>();
        queue.enqueue(root);
        while (!queue.isEmpty()) {
            Node x = queue.dequeue();
            double dis = p.distanceSquaredTo(x.p);
            if (dis < mindis) {
                retp = x.p;
                mindis = dis;
            }
            if (x.lb != null && x.lb.p.distanceSquaredTo(p) < mindis)
                queue.enqueue(x.lb);
            if (x.rt != null && x.rt.p.distanceSquaredTo(p) < mindis)
                queue.enqueue(x.rt);
        }
        return retp;
    }

    private Point2D nearest(final Node node, final RectHV rect, final Point2D p, Point2D candidate) {
        if (node == null) return candidate;

        double dqn = 0.0;
        double drq = 0.0;
        RectHV leftRect = null;
        RectHV rigtRect = null;
        final Point2D query = new Point2D(p.x(), p.y());

        if (candidate != null) {
            dqn = query.distanceSquaredTo(candidate);
            drq = rect.distanceSquaredTo(query);
        }

        if (candidate == null || dqn > drq) {
            final Point2D point = new Point2D(node.p.x(), node.p.y());
            if (candidate == null || dqn > query.distanceSquaredTo(point))
                candidate = point;

            if (node.isVertical) {
                // only p.x() changes
                leftRect = new RectHV(rect.xmin(), rect.ymin(), node.p.x(), rect.ymax());
                rigtRect = new RectHV(node.p.x(), rect.ymin(), rect.xmax(), rect.ymax());

                if (p.x() < node.p.x()) {
                    candidate = nearest(node.lb, leftRect, p, candidate);
                    candidate = nearest(node.rt, rigtRect, p, candidate);
                }
                else {
                    candidate = nearest(node.rt, rigtRect, p, candidate);
                    candidate = nearest(node.lb, leftRect, p, candidate);
                }
            }
            else {
                // only p.y() changes
                leftRect = new RectHV(rect.xmin(), rect.ymin(), rect.xmax(), node.p.y());
                rigtRect = new RectHV(rect.xmin(), node.p.y(), rect.xmax(), rect.ymax());

                if (p.y() < node.p.y()) {
                    candidate = nearest(node.lb, leftRect, p, candidate);
                    candidate = nearest(node.rt, rigtRect, p, candidate);
                } else {
                    candidate = nearest(node.rt, rigtRect, p, candidate);
                    candidate = nearest(node.lb, leftRect, p, candidate);
                }
            }
        }
        return candidate;
    }
}
```
