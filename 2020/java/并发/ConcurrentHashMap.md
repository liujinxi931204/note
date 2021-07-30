## ConcurrentHashMap简介  

ConcurrnetHashMap是在JDK1.5时，J.U.U引入的一个同步集合工具类，这是一个线程安全的HashMap。来看一下它的类继承关系  

![image-20210730154850453](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/image-20210730154850453.png)  

可以看到ConcurrentHashMap类继承了AbstractMap类，这是一个Java.util包下的抽象类，提供Map接口的骨干实现，以最大限度地减少实现Map这类数据结构时所需地工作量。  此外ConcurrentHashMap实现了ConcurrentMap接口，ConcurrentMap是在JDK1.5时随着J.U.C包引入地，这个接口其实就是实现了一些针对Map的原子操作  

## ConcurrentHashMap基本结构  

![ConcurentHashMap基本结构](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/ConcurentHashMap%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84.png)  

### 基本结构

CouncurrentHashMap内部维护了一个Node类型的数组，也就是table。数组的每一个位置`table[i]`代表了一个桶，当桶插入键值对时，会根据键的hash值映射到不同的桶位，table一共有四种不同类型的桶：Node、TreeBin、ForwardingNode、ReservationNode。其中TreeBin桶所连接的是一棵红黑树，红黑树的节点是TreeNode类型的，因此ConcurrentHashMap其实有物种类型的Node节点  

### 节点定义  

#### Node节点  

Node节点是其他四种类型节点的父类  

默认连接到`table[i]`桶上的就是Node节点。当出现hash冲突的时候，Node节点会首先以链表的形式连接到table上，当节点的数量超过了一定数目时，链表会转化为红黑树。因为链表的查询的时间复杂度是`O(n)`，红黑树的插叙的时间复杂度是`O(logn)`  

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;//链表的指针

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }

    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    public final String toString(){ return key + "=" + val; }
    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    public final boolean equals(Object o) {
        Object k, v, u; Map.Entry<?,?> e;
        return ((o instanceof Map.Entry) &&
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                (v = e.getValue()) != null &&
                (k == key || k.equals(key)) &&
                (v == (u = val) || v.equals(u)));
    }

    /**
    * Virtualized support for map.get(); overridden in subclasses.
    * 以链表的方式查询元素
    */
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }
}
```

#### ForwardingNode节点  

ForwardingNode节点仅仅在扩容的时候才会使用  

```java
/**
* ForwardingNode是一种临时节点，在扩容进行中才会出现，hash值固定为-1，且不存储实际数据
* 如果旧table数组中的一个hash桶中全部节点都迁移到了新table中，则在旧table中这个桶位放置一个ForwardingNode节点
* 读操作碰到ForwardingNode时，将操作转到扩容后的新table中去执行
* 写操作盆碰到ForwardingNode时，则会尝试帮助扩容
*/

static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }

    //在新的table中进行查询操作
    Node<K,V> find(int h, Object k) {
        // loop to avoid arbitrarily deep recursion on forwarding nodes
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;
            if (k == null || tab == null || (n = tab.length) == 0 ||
                (e = tabAt(tab, (n - 1) & h)) == null)
                return null;
            for (;;) {
                int eh; K ek;
                if ((eh = e.hash) == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                if (eh < 0) {
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    else
                        return e.find(h, k);
                }
                if ((e = e.next) == null)
                    return null;
            }
        }
    }
}
```

#### TreeBin节点  

TreeBin节点相当于TreeNode的代理节点。TreeBin会直接连接到`table[i]`中，该节点提供了一系列红黑树相关的操作，以及加锁、解锁操作  

```java
/**
* TreeNode的代理节点（相当于封装了TreeNode的容器，提供了针对红黑树的转换操作和锁控制）
* hash值固定为-2
*/
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;              //红黑树的根节点
    volatile TreeNode<K,V> first;    //链表结构的头节点
    volatile Thread waiter;          //最近的一个设置WAITER标识的线程
    volatile int lockState;          //整体的锁状态的标识
    // values for lockState
    static final int WRITER = 1;     // 二进制001，红黑树的写锁状态
    static final int WAITER = 2;     // 二进制010，红黑树的等待获取写锁的状态
    static final int READER = 4;     // 二进制100，红黑树的读锁状态，读可以并发，每多一个读线程，lockState都加上一个READER

    /**
    * Tie-breaking utility for ordering insertions when equal
    * hashCodes and non-comparable. We don't require a total
    * order, just a consistent insertion rule to maintain
    * equivalence across rebalancings. Tie-breaking further than
    * necessary simplifies testing a bit.
    */
    static int tieBreakOrder(Object a, Object b) {
        int d;
        if (a == null || b == null ||
            (d = a.getClass().getName().
             compareTo(b.getClass().getName())) == 0)
            d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                 -1 : 1);
        return d;
    }

    /**
    * 将b为头节点的链表转换为红黑树
    */
    TreeBin(TreeNode<K,V> b) {
        super(TREEBIN, null, null, null);
        this.first = b;
        TreeNode<K,V> r = null;
        for (TreeNode<K,V> x = b, next; x != null; x = next) {
            next = (TreeNode<K,V>)x.next;
            x.left = x.right = null;
            if (r == null) {
                x.parent = null;
                x.red = false;
                r = x;
            }
            else {
                K k = x.key;
                int h = x.hash;
                Class<?> kc = null;
                for (TreeNode<K,V> p = r;;) {
                    int dir, ph;
                    K pk = p.key;
                    if ((ph = p.hash) > h)
                        dir = -1;
                    else if (ph < h)
                        dir = 1;
                    else if ((kc == null &&
                              (kc = comparableClassFor(k)) == null) ||
                             (dir = compareComparables(kc, k, pk)) == 0)
                        dir = tieBreakOrder(k, pk);
                    TreeNode<K,V> xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        r = balanceInsertion(r, x);
                        break;
                    }
                }
            }
        }
        this.root = r;
        assert checkInvariants(root);
    }

    /**
    * 对红黑树的头节点加写锁
    */
    private final void lockRoot() {
        if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
            contendedLock(); // offload to separate method
    }

    /**
    * 释放红黑树头节点的写锁
    */
    private final void unlockRoot() {
        lockState = 0;
    }

    /**
    * 
    */
    private final void contendedLock() {
        boolean waiting = false;
        for (int s;;) {
            if (((s = lockState) & ~WAITER) == 0) {
                if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                    if (waiting)
                        waiter = null;
                    return;
                }
            }
            else if ((s & WAITER) == 0) {
                if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) {
                    waiting = true;
                    waiter = Thread.currentThread();
                }
            }
            else if (waiting)
                LockSupport.park(this);
        }
    }

    /**
    * 从根节点开始遍历查找，找到需要的节点就返回，没有找到就返回null
    * 当存在写锁时，以链表的方式查找
    */
    final Node<K,V> find(int h, Object k) {
        if (k != null) {
            for (Node<K,V> e = first; e != null; ) {
                int s; K ek;
                /**
                * 以下两种情况以链表的方式进行查找
                * 1. 有线程正持有写锁，这样做能够不阻塞读线程
                * 2. 有线程等待获取写锁，不再继续加读锁，相当于"写优先"模式
                */
                if (((s = lockState) & (WAITER|WRITER)) != 0) {
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                    e = e.next;
                }
                else if (U.compareAndSwapInt(this, LOCKSTATE, s,
                                             s + READER)) {
                    TreeNode<K,V> r, p;
                    try {
                        p = ((r = root) == null ? null :
                             r.findTreeNode(h, k, null));
                    } finally {
                        Thread w;
                        if (U.getAndAddInt(this, LOCKSTATE, -READER) ==
                            (READER|WAITER) && (w = waiter) != null)
                            LockSupport.unpark(w);
                    }
                    return p;
                }
            }
        }
        return null;
    }

    /**
    * 查找指定key对应的节点，如果未找到，则插入
    * @return，插入成功后返回null，否则返回找到的节点
    */
    final TreeNode<K,V> putTreeVal(int h, K k, V v) {
        Class<?> kc = null;
        boolean searched = false;
        for (TreeNode<K,V> p = root;;) {
            int dir, ph; K pk;
            if (p == null) {
                first = root = new TreeNode<K,V>(h, k, v, null, null);
                break;
            }
            else if ((ph = p.hash) > h)
                dir = -1;
            else if (ph < h)
                dir = 1;
            else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                return p;
            else if ((kc == null &&
                      (kc = comparableClassFor(k)) == null) ||
                     (dir = compareComparables(kc, k, pk)) == 0) {
                if (!searched) {
                    TreeNode<K,V> q, ch;
                    searched = true;
                    if (((ch = p.left) != null &&
                         (q = ch.findTreeNode(h, k, kc)) != null) ||
                        ((ch = p.right) != null &&
                         (q = ch.findTreeNode(h, k, kc)) != null))
                        return q;
                }
                dir = tieBreakOrder(k, pk);
            }

            TreeNode<K,V> xp = p;
            if ((p = (dir <= 0) ? p.left : p.right) == null) {
                TreeNode<K,V> x, f = first;
                first = x = new TreeNode<K,V>(h, k, v, f, xp);
                if (f != null)
                    f.prev = x;
                if (dir <= 0)
                    xp.left = x;
                else
                    xp.right = x;
                if (!xp.red)
                    x.red = true;
                else {
                    lockRoot();
                    try {
                        root = balanceInsertion(root, x);
                    } finally {
                        unlockRoot();
                    }
                }
                break;
            }
        }
        assert checkInvariants(root);
        return null;
    }

    /**
    * 删除红黑树的节点
    * 1. 红黑树规模太小时，返回true，然后进行树--》链表的转换
    * 2. 红黑树规模足够时，不用变换成链表，但删除节点时需要加写锁
    * Removes the given node, that must be present before this
    * call.  This is messier than typical red-black deletion code
    * because we cannot swap the contents of an interior node
    * with a leaf successor that is pinned by "next" pointers
    * that are accessible independently of lock. So instead we
    * swap the tree linkages.
    *
    * @return true if now too small, so should be untreeified
    */
    final boolean removeTreeNode(TreeNode<K,V> p) {
        TreeNode<K,V> next = (TreeNode<K,V>)p.next;
        TreeNode<K,V> pred = p.prev;  // unlink traversal pointers
        TreeNode<K,V> r, rl;
        if (pred == null)
            first = next;
        else
            pred.next = next;
        if (next != null)
            next.prev = pred;
        if (first == null) {
            root = null;
            return true;
        }
        if ((r = root) == null || r.right == null || // too small
            (rl = r.left) == null || rl.left == null)
            return true;
        lockRoot();
        try {
            TreeNode<K,V> replacement;
            TreeNode<K,V> pl = p.left;
            TreeNode<K,V> pr = p.right;
            if (pl != null && pr != null) {
                TreeNode<K,V> s = pr, sl;
                while ((sl = s.left) != null) // find successor
                    s = sl;
                boolean c = s.red; s.red = p.red; p.red = c; // swap colors
                TreeNode<K,V> sr = s.right;
                TreeNode<K,V> pp = p.parent;
                if (s == pr) { // p was s's direct parent
                    p.parent = s;
                    s.right = p;
                }
                else {
                    TreeNode<K,V> sp = s.parent;
                    if ((p.parent = sp) != null) {
                        if (s == sp.left)
                            sp.left = p;
                        else
                            sp.right = p;
                    }
                    if ((s.right = pr) != null)
                        pr.parent = s;
                }
                p.left = null;
                if ((p.right = sr) != null)
                    sr.parent = p;
                if ((s.left = pl) != null)
                    pl.parent = s;
                if ((s.parent = pp) == null)
                    r = s;
                else if (p == pp.left)
                    pp.left = s;
                else
                    pp.right = s;
                if (sr != null)
                    replacement = sr;
                else
                    replacement = p;
            }
            else if (pl != null)
                replacement = pl;
            else if (pr != null)
                replacement = pr;
            else
                replacement = p;
            if (replacement != p) {
                TreeNode<K,V> pp = replacement.parent = p.parent;
                if (pp == null)
                    r = replacement;
                else if (p == pp.left)
                    pp.left = replacement;
                else
                    pp.right = replacement;
                p.left = p.right = p.parent = null;
            }

            root = (p.red) ? r : balanceDeletion(r, replacement);

            if (p == replacement) {  // detach pointers
                TreeNode<K,V> pp;
                if ((pp = p.parent) != null) {
                    if (p == pp.left)
                        pp.left = null;
                    else if (p == pp.right)
                        pp.right = null;
                    p.parent = null;
                }
            }
        } finally {
            unlockRoot();
        }
        assert checkInvariants(root);
        return false;
    }

    /* ------------------------------------------------------------ */
    // Red-black tree methods, all adapted from CLR

    static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                          TreeNode<K,V> p) {
        TreeNode<K,V> r, pp, rl;
        if (p != null && (r = p.right) != null) {
            if ((rl = p.right = r.left) != null)
                rl.parent = p;
            if ((pp = r.parent = p.parent) == null)
                (root = r).red = false;
            else if (pp.left == p)
                pp.left = r;
            else
                pp.right = r;
            r.left = p;
            p.parent = r;
        }
        return root;
    }

    static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                           TreeNode<K,V> p) {
        TreeNode<K,V> l, pp, lr;
        if (p != null && (l = p.left) != null) {
            if ((lr = p.left = l.right) != null)
                lr.parent = p;
            if ((pp = l.parent = p.parent) == null)
                (root = l).red = false;
            else if (pp.right == p)
                pp.right = l;
            else
                pp.left = l;
            l.right = p;
            p.parent = l;
        }
        return root;
    }

    static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                TreeNode<K,V> x) {
        x.red = true;
        for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
            if ((xp = x.parent) == null) {
                x.red = false;
                return x;
            }
            else if (!xp.red || (xpp = xp.parent) == null)
                return root;
            if (xp == (xppl = xpp.left)) {
                if ((xppr = xpp.right) != null && xppr.red) {
                    xppr.red = false;
                    xp.red = false;
                    xpp.red = true;
                    x = xpp;
                }
                else {
                    if (x == xp.right) {
                        root = rotateLeft(root, x = xp);
                        xpp = (xp = x.parent) == null ? null : xp.parent;
                    }
                    if (xp != null) {
                        xp.red = false;
                        if (xpp != null) {
                            xpp.red = true;
                            root = rotateRight(root, xpp);
                        }
                    }
                }
            }
            else {
                if (xppl != null && xppl.red) {
                    xppl.red = false;
                    xp.red = false;
                    xpp.red = true;
                    x = xpp;
                }
                else {
                    if (x == xp.left) {
                        root = rotateRight(root, x = xp);
                        xpp = (xp = x.parent) == null ? null : xp.parent;
                    }
                    if (xp != null) {
                        xp.red = false;
                        if (xpp != null) {
                            xpp.red = true;
                            root = rotateLeft(root, xpp);
                        }
                    }
                }
            }
        }
    }

    static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                               TreeNode<K,V> x) {
        for (TreeNode<K,V> xp, xpl, xpr;;)  {
            if (x == null || x == root)
                return root;
            else if ((xp = x.parent) == null) {
                x.red = false;
                return x;
            }
            else if (x.red) {
                x.red = false;
                return root;
            }
            else if ((xpl = xp.left) == x) {
                if ((xpr = xp.right) != null && xpr.red) {
                    xpr.red = false;
                    xp.red = true;
                    root = rotateLeft(root, xp);
                    xpr = (xp = x.parent) == null ? null : xp.right;
                }
                if (xpr == null)
                    x = xp;
                else {
                    TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                    if ((sr == null || !sr.red) &&
                        (sl == null || !sl.red)) {
                        xpr.red = true;
                        x = xp;
                    }
                    else {
                        if (sr == null || !sr.red) {
                            if (sl != null)
                                sl.red = false;
                            xpr.red = true;
                            root = rotateRight(root, xpr);
                            xpr = (xp = x.parent) == null ?
                                null : xp.right;
                        }
                        if (xpr != null) {
                            xpr.red = (xp == null) ? false : xp.red;
                            if ((sr = xpr.right) != null)
                                sr.red = false;
                        }
                        if (xp != null) {
                            xp.red = false;
                            root = rotateLeft(root, xp);
                        }
                        x = root;
                    }
                }
            }
            else { // symmetric
                if (xpl != null && xpl.red) {
                    xpl.red = false;
                    xp.red = true;
                    root = rotateRight(root, xp);
                    xpl = (xp = x.parent) == null ? null : xp.left;
                }
                if (xpl == null)
                    x = xp;
                else {
                    TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                    if ((sl == null || !sl.red) &&
                        (sr == null || !sr.red)) {
                        xpl.red = true;
                        x = xp;
                    }
                    else {
                        if (sl == null || !sl.red) {
                            if (sr != null)
                                sr.red = false;
                            xpl.red = true;
                            root = rotateLeft(root, xpl);
                            xpl = (xp = x.parent) == null ?
                                null : xp.left;
                        }
                        if (xpl != null) {
                            xpl.red = (xp == null) ? false : xp.red;
                            if ((sl = xpl.left) != null)
                                sl.red = false;
                        }
                        if (xp != null) {
                            xp.red = false;
                            root = rotateRight(root, xp);
                        }
                        x = root;
                    }
                }
            }
        }
    }

    /**
         * Recursive invariant check
         */
    static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
        TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
        tb = t.prev, tn = (TreeNode<K,V>)t.next;
        if (tb != null && tb.next != t)
            return false;
        if (tn != null && tn.prev != t)
            return false;
        if (tp != null && t != tp.left && t != tp.right)
            return false;
        if (tl != null && (tl.parent != t || tl.hash > t.hash))
            return false;
        if (tr != null && (tr.parent != t || tr.hash < t.hash))
            return false;
        if (t.red && tl != null && tl.red && tr != null && tr.red)
            return false;
        if (tl != null && !checkInvariants(tl))
            return false;
        if (tr != null && !checkInvariants(tr))
            return false;
        return true;
    }

    private static final sun.misc.Unsafe U;
    private static final long LOCKSTATE;
    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = TreeBin.class;
            LOCKSTATE = U.objectFieldOffset
                (k.getDeclaredField("lockState"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

#### TreeNode节点  

TreeNode节点就是红黑树的节点，TreeNode不会直接连接到`table[i]`桶上，而是由TreeBin节点连接，TreeBin会指向红黑树的根节点  

```java
//红黑树的节点，存储实际的数据
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    //prev指针是为了删除方便
    //删除链表的非头节点时，需要知道它的前驱节点才能删除，所以直接提供一个prev指针
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
    * Returns the TreeNode (or null if not found) for the given key
    * starting at given root.
    * 以当前节点（this）为根节点，开始遍历指定的key
    */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}

```

#### ReservationNode节点  

保留节点，ConcurrentHashMap中的一些特殊方法会专门用到该类节点  

```java
/**
* hash值固定为-3，不保存实际数据
* 只在computeIfAbsent和compute这两个函数式API中充当占位符加锁使用
*/
static final class ReservationNode<K,V> extends Node<K,V> {
    ReservationNode() {
        super(RESERVED, null, null, null);
    }

    Node<K,V> find(int h, Object k) {
        return null;
    }
}
```

#### 总结  

由上面可以知道，Node节点是其他节点的父类。ForwardingNode节点在扩容的时候才会用到，它的hash值固定为-1。TreeBin节点相当于是TreeNode节点的代理节点，不存储真实的数据，在它的内部定义了关于红黑树的操作，它的hash值固定为-2。TreeNode是存储真实数据的红黑树节点。ReservationNode节点只有在一些特殊方法中才会使用，它的hash值固定为-3。

