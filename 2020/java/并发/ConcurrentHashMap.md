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

## ConcurrentHashMap的构造  

#### 构造器定义  

ConcurrentHashMap提供了五个构造器，这五个构造器内部最多只是计算了下table数组的初始长度容量最大值，并没有完成实际的创建table数组的动作。ConcurrentHashMap采用了一种懒加载的方式，只有首次插入键值对的时候，才会真正的去创建table数组  

##### 空构造器  

```java
public ConcurrentHashMap() {
}
```

 ##### 指定初始容量的构造器  

```java
/**
* 这个构造器指定了table数组的初始容量
* 最后tableSizeFor方法会返回(initialCapacity + (initialCapacity >>> 1) + 1)最小的2次幂
* 例如initialCapacity=3，那么initialCapacity >>> 1=1，tableSizeFor方法就返回8
*/
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```

##### 根据已有的Map构造  

```java
/**
* 根据已有的map构造ConcurrentHashMap
*/
public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}
```

##### 指定table初始容量和负载因子的构造器  

```java
/**
* 指定table的初始容量和负载因子
*/
public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}
```

##### 指定table的初始容量、负载因子和并发级别的构造器  

```java
/**
* 指定table的初始容量、负载因子和并发级别
* 1.8中的concurrencyLevel并发级别只是为了兼容之前的版本，并不是实际的并发级别
* 负载因子loadFactory也不是真实的负载因子，仅仅对初始容量有一定的控制作用
*/
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

这里来看一下tableSizeFor方法  

```java
private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

这个方法就是返回输入数字最相近的2次幂。使用这个方法为的是让ConcurrentHashMap的长度始终都是2的幂次，在计算键值对在桶中的位置时可以借助于位运算以提高速度

### 常量字段定义  

#### 常量  

```java
/**
 * 最大容量.
 */
private static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认初始容量
 */
private static final int DEFAULT_CAPACITY = 16;

/**
 * The largest possible (non-power of two) array size.
 * Needed by toArray and related methods.
 */
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * 负载因子，为了兼容JDK1.8以前的版本而保留。
 * JDK1.8中的ConcurrentHashMap的负载因子恒定为0.75
 */
private static final float LOAD_FACTOR = 0.75f;

/**
 * 链表转树的阈值，即链接结点数大于8时， 链表转换为树.
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 树转链表的阈值，即树结点树小于6时，树转换为链表.
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 在链表转变成树之前，还会有一次判断：
 * 即只有键值对数量大于MIN_TREEIFY_CAPACITY，才会发生转换。
 * 这是为了避免在Table建立初期，多个键值对恰好被放入了同一个链表中而导致不必要的转化。
 */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * 在树转变成链表之前，还会有一次判断：
 * 即只有键值对数量小于MIN_TRANSFER_STRIDE，才会发生转换.
 */
private static final int MIN_TRANSFER_STRIDE = 16;

/**
 * 用于在扩容时生成唯一的随机数.
 */
private static int RESIZE_STAMP_BITS = 16;

/**
 * 可同时进行扩容操作的最大线程数.
 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

/**
 * The bit shift for recording size stamp in sizeCtl.
 */
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

static final int MOVED = -1;                // 标识ForwardingNode结点（在扩容时才会出现，不存储实际数据）
static final int TREEBIN = -2;              // 标识红黑树的根结点
static final int RESERVED = -3;             // 标识ReservationNode结点（）
static final int HASH_BITS = 0x7fffffff;    // usable bits of normal node hash

/**
 * CPU核心数，扩容时使用
 */
static final int NCPU = Runtime.getRuntime().availableProcessors();
```

#### 字段 

```java
/**
 * Node数组，标识整个Map，首次插入元素时创建，大小总是2的幂次.
 */
transient volatile Node<K, V>[] table;

/**
 * 扩容后的新Node数组，只有在扩容时才非空.
 */
private transient volatile Node<K, V>[] nextTable;

/**
 * 控制table的初始化和扩容.
 * 0  : 初始默认值
 * -1 : 有线程正在进行table的初始化
 * >0 : table初始化时使用的容量，或初始化/扩容完成后的threshold
 * -(1 + nThreads) : 记录正在执行扩容任务的线程数
 */
private transient volatile int sizeCtl;

/**
 * 扩容时需要用到的一个下标变量.
 */
private transient volatile int transferIndex;

/**
 * 计数基值,当没有线程竞争时，计数将加到该变量上。类似于LongAdder的base变量
 */
private transient volatile long baseCount;

/**
 * 计数数组，出现并发冲突时使用。类似于LongAdder的cells数组
 */
private transient volatile CounterCell[] counterCells;

/**
 * 自旋标识位，用于CounterCell[]扩容时使用。类似于LongAdder的cellsBusy变量
 */
private transient volatile int cellsBusy;


// 视图相关字段
private transient KeySetView<K, V> keySet;
private transient ValuesView<K, V> values;
private transient EntrySetView<K, V> entrySet;
```

## ConcurrentHashMap的put操作  

put操作是插入一个键值对到ConcurrentHashMap中  

```java
public V put(K key, V value) {
    //只是在内部调用了putVal(key, value, false)方法
    return putVal(key, value, false);
}
```

可以看到，这个方法只是在内部调用了一下`putVal(key, value, false)`方法  

```java
//实际上插入操作的完成者
final V putVal(K key, V value, boolean onlyIfAbsent) {
    //concurrentHashMap中不能插入key为null或者value为null的键值对
    if (key == null || value == null) throw new NullPointerException();
    //再次计算hash值，让hashCode的高低16位都参与运算，降低hash碰撞的概率
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {//自旋，直到插入成功
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            //首次插入元素的时候，ConcurrentHashMap为空，所以在这里真正的创建ConcurrentHashMap，懒加载的体现
            tab = initTable();
        /**
        * n是ConcurrentHashMap的长度，因为n始终是2的幂次
        * 所以元素在桶中的位置i=hash%n=(n-1)&hash，这就是为什么之前使用tableSizeFor方法的原因 
        * 如果table[i]的位置为空，使用cas操作将键值对插入到table[i]，如果插入失败，就会自旋，进行
        * 下一次插入操作，直到插入成功
        */
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //发现ForwardingNode节点，说明此时table正在迁移，则协助完成table中数据的迁移
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            //到这里说明，table[i]位置已经有元素了
            V oldVal = null;
            //锁住table[i]，然后再检查一下table[i]是不是第一个节点，防止有别的线程进行了修改
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //说明table[i]是链表节点，按照链表的方式插入键值对
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //找到相等的元素，判断是否需要更新val值
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            //没有找到相等的元素，采用尾插法，插入新节点
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //table[i]是TreeBin节点，按照红黑树的方式插入新节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    //链表转换为红黑树
                    treeifyBin(tab, i);
                if (oldVal != null)
                    //表明这次put操作只是替换了旧值，不用更改计数值
                    return oldVal;
                break;
            }
        }
    }
    //新插入键值对以后将计数值加1
    addCount(1L, binCount);
    return null;
}

```

putVal方法一共处理四种情况  

### 首次初始化table  

ConcurrentHashMap在构造的时候不会初始化table数组，而是在首次插入元素时进行初始化，初始化就是在initTable中完成的  

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    //自旋直到初始化成功
    while ((tab = table) == null || tab.length == 0) {
        //在构造器中已经给sizeCtl赋值了，如果sizeCtl小于0，说明正在初始化或者扩容，所以放弃CPU一段时间
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        //将sizeCtl通过cas设置为-1，表示正在初始化
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //sc=0.75*n
                    sc = n - (n >>> 2);
                }
            } finally {
                //设置threshold=0.75*length
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

可以看到初始化table数组的时候就是初始化一个长度为sizeCtl的数组，然后将sizeCtl设置为0.75，表示设置了threshold为0.75  

### table[i]对应的桶为空  

最简单的情况，直接通过cas的方式占用桶`table[i]`即可  

### 遇到ForwardingNode节点

  ForwardingNode节点是ConcurrentHashMap中五类节点之一，相当于一个占位节点，表示当前table正在进行扩容，当前线程可以尝试协助数据迁移  

### 出现hash冲突   

当两个不同key的hash值映射到同一个`table[i]`桶中时，就会出现这种情况  

+ 当`table[i]`的节点类型为Node（链表节点）就会出现以"尾插法"的形式插入链表的结尾  

+ 当`table[i]`的节点类型为TreeBin（红黑树代理节点）就会将新节点通过红黑树的插入方式插入  

putVal方法的最后，涉及将链表转化为红黑树——treeifBin，但实际情况并非立即就会转换，当table的容量小于64时，处于性能的考虑，只是对table数组进行扩容一倍——tryPresize  

putVal方法可以用下面的流程图来表示

![ConcurrentHashMap插入操作](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/ConcurrentHashMap%E6%8F%92%E5%85%A5%E6%93%8D%E4%BD%9C.png)

## ConcurrentHashMap的get操作  

```java
/**
* 根据指定的key值查找对应的key
*/
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    //重新计算hash值
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        //table[i]就是要查找的值，直接返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //特殊节点，调用节点的find方法
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //按照链表的方式查找
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

get方法的逻辑很简单，首先根据key的hash值计算出映射到table的哪个桶——`table[i]`

1. 如果`table[i]`的key和带查找的key相同，那就直接返回

2. 如果`table[i]`对应的节点时特殊节点（hahs值小于0），则通过find方法查找

3. 如果`table[i]`对应的节点时普通链表节点，则按照链表的方式查找  

### Node节点的查找  

当`table[i]`被普通节点Node占用，说明是链表形式，直接从链表头开始查找  

```java
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
```

### TreeBin节点的查找  

```java
/**
* 从根节点开始遍历查找，找到"相等"的节点就返回，没有找到就返回null
* 当存在写锁时，以链表的形式查找
*/
final Node<K,V> find(int h, Object k) {
    if (k != null) {
        for (Node<K,V> e = first; e != null; ) {
            int s; K ek;
            /**
            * 当有以下情况时以链表的方式查找
            * 1. 有线程正持有写锁，这样做能够不阻塞读线程
            * 2. 有线程等待获取写锁，不再继续加读锁，相当于"写优先模式"
            */
            if (((s = lockState) & (WAITER|WRITER)) != 0) {
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                e = e.next;
            }
            //读线程数量加一，读状态累加
            else if (U.compareAndSwapInt(this, LOCKSTATE, s,
                                         s + READER)) {
                TreeNode<K,V> r, p;
                try {
                    p = ((r = root) == null ? null :
                         r.findTreeNode(h, k, null));
                } finally {
                    Thread w;
                    //如果当前线程是最后一个读线程，且有写线程因为读线程而阻塞，则告诉它可以获取写锁了
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

```

TreeBin的查找比较特殊，当槽`table[i]`被TreeBin节点占用时，说明连接的是一棵红黑树。由于红黑树的插入、删除操作会涉及到整个结构的调整，所以通常会涉及到读写并发操作的时候，是需要加锁的。ConcurrentHashMap采用了类似读写锁的方式：当线程持有写锁（修改红黑树）时，如果读线程需要查找，不会像传统读写锁那样阻塞，而是转而采用链表的形式进行查找（TreeBin节点本身就是Node节点，拥有Node节点的所有属性）

### ForwardingNode节点的查找  

```java
/**
* 在新的扩容的table——nextTable上进行查找
*/
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

```

ForwardingNode节点是一种临时节点，只有在扩容的时候才会出现，所以查找也在扩容的table上进行  

### ReservationNode节点的查找  

```java
Node<K, V> find(int h, Object k) {
    return null;
}
```

ReservationNode节点是保留节点，不保存实际的数据，所以直接返回null

## ConcurrentHashMap的remove操作  

remove操作是删除ConcurrentHashMap中的一个键值对  

```java
public V remove(Object key) {
    return replaceNode(key, null, null);
}
```

看到，remove方法什么也没有做，只是在内部调用了replaceNode这个方法  

```java
final V replaceNode(Object key, V value, Object cv) {
    //重新进行一次hash
    int hash = spread(key.hashCode());
    for (Node<K,V>[] tab = table;;) {//自旋
        Node<K,V> f; int n, i, fh;
        //如果table[i]上没有元素，说明key不存在，返回null
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;
        //说明正在扩容，则协助进行扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            //如果存在，可能是链表也可能是红黑树
            V oldVal = null;
            boolean validated = false;
            //锁定table[i]
            synchronized (f) {
                //再检查一下table[i]是不是第一个节点，防止有别的线程进行了修改
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {//说明是链表，要删除的节点在链表中，采用链表的方式删除一个节点
                        validated = true;
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                //找到要删除的节点
                                V ev = e.val;
                                if (cv == null || cv == ev ||
                                    (ev != null && cv.equals(ev))) {
                                    oldVal = ev;//记录旧值
                                    if (value != null)
                                        //对旧值进行覆盖，因为在replace方法的内部也调用了该方法，提供通用性
                                        e.val = value;
                                    else if (pred != null)
                                        //要删除的节点有前驱节点，则让前驱节点的next指向当前节点的后继阶段，即pred.next=e.next
                                        pred.next = e.next;
                                    else
                                        //说明是链表头，即table[i]，则cas重新设置table[i]=e.next
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            //不是当前节点，则向链表的后面遍历
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    //说明是红黑树，要删除红黑树中的节点，采用红黑树的方式删除节点
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        if ((r = t.root) != null &&
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    //如果删除了节点，更新size
                    if (value == null)
                        addCount(-1L, -1);
                    //返回旧值
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```

可以看到remove方法什么也没有做，而是简单的在内部调用了replaceNode方法。replaceNode方法的逻辑也很清晰  

1. 如果对应的`table[i]`为空，说明key不存在，那么返回null  
2. 如果对应的`table[i].hash==MOVED`，说明正在扩容，则尝试取协助扩容
3. 如果`table[i]`存在，且hash值大于等于0，说明是链表或者只有`table[i]`这一个元素，统一按照链表的方式删除
4. 如果`table[i]`存在，而且是TreeBin节点，说明是红黑树，则按照红黑树的方式删除链表

## ConcurrentHashMap的扩容操作  

### 扩容思路  

HashMap的扩容，一般包括两个步骤  

#### 数组扩容

table数组的扩容，一般就是新建一个2被大小的新数组，这个过程由一个线程完成，且不允许出现并发

#### 数组迁移  

所谓数组迁移，就是把旧table中的各个槽中的数组重新分配到新的table中。

这一过程通常涉及到槽中key的rehash，因为key映射到桶的位置与桶的大小有关，新table的大小发生了变化，key映射的位置一般也会发生变化

ConcurrentHashMap在处理rehash的时候，并不会重新计算每个key的hash值，而是利用一种很巧妙的方法。之前说过ConcurrentHashMap的数组长度必须为2的幂次，原因是让key分布更均匀，减少冲突，这只是其中一个原因，另一个原因就是：

当数组的长度为2的幂次时，通过`(table.legth-1)&key.hash`这种方式计算的索引为`i`，那么扩容以后（2倍），新的索引要么是`i`，要么是`i+n`（这里n是旧的table的长度）

![ConcurrentHashMap数组扩容](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/ConcurrentHashMap%E6%95%B0%E7%BB%84%E6%89%A9%E5%AE%B9.png)  

### 扩容时机  

在之前putVala方法中，当往Map中插入节点时，如果链表的节点数目超过了一定的阈值，就会触发链表转红黑树的操作  

```java
if (binCount >= TREEIFY_THRESHOLD)
    treeifyBin(tab, i);             // 链表 -> 红黑树 转换
```

来看一下treefyBin方法  

```java
/**
* 进行链表转红黑树的操作
* 1. 如果数组的长度小于MIN_TREEIFY_CAPACITY，默认是64，不会转换为红黑树，而是会进行扩容
* 2. 如果数组的长度大于了MIN_TREEIFY_CAPACITY，才会进行链表到红黑树的转换
*/
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        //如果数组的长度小于MIN_TREEIFY_CAPACITY，默认是64，不会转换为红黑树，而是会进行扩容
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        //如果数组的长度大于了MIN_TREEIFY_CAPACITY，才会进行链表到红黑树的转换
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {//依然是锁住table[i]
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    //遍历整个链表，建立红黑树
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    //以TreeBin类型包装，并连接到table[index]中
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

因此在进行链表转换为红黑树的过程中，还会再对table数组进行一次判断  

1. 如果数组的长度小于MIN_TREEIFY_CAPACITY，默认是64，不会转换为红黑树，而是会进行扩容

2. 如果数组的长度大于等于了MIN_TREEIFY_CAPACITY，才会进行从链表到红黑树的转换

来看一下tryPresize方法是如何扩容的  

```java
/**
* 尝试对table数组进行扩容
* 1. table数组还没有初始化，则先进行初始化
* 2. 如果数组已经扩容过了或者超过了最大长度的限制，直接返回
* 3.1 已经有线程在扩容了，如果当前线程可以帮助迁移数据，则协助数据的迁移；如果不能协助数据的迁移，则退出
* 3.2 没有线程在扩容，则设置当前线程为第一个进行扩容的线程，将sc置为负数
*/
private final void tryPresize(int size) {
    //调整c为2的整数幂
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
    tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        //table还没有初始化，则先进行初始化
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        //如果已经进行了扩容或者数组的长度超过了最大的限制，就返回了
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        //进行table数组扩容
        else if (tab == table) {
            //根据容量n生成一个随机数，唯一标识本次扩容操作
            int rs = resizeStamp(n);
            //sc小于0，说明有别的线程正在扩容
            if (sc < 0) {
                Node<K,V>[] nt;
                //如果当前线程不能协助数据转移，则退出
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                //协助数据转移，把正在进行transfer任务的线程数加一
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            //没有线程在扩容，通过cas设置当前线程为第一个扩容操作的线程，将sc置为负数
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
```

从上面的分析可以看到，扩容的时候一共有三个分支  

1. table数组还没有初始化，则先进行初始化

2. 如果数组已经扩容过了或者超过了最大长度的限制，直接返回

   3.1 已经有线程在扩容了，如果当前线程可以帮助迁移数据，则协助数据的迁移；如果不能协助数据的迁移，则退

   3.2 没有线程在扩容，则设置当前线程为第一个进行扩容的线程，将sc置为负数

一般扩容的时候会走到3这个分支中来，但是无论是协助迁移数据还是扩容都会调用transfer方法

来看一下transfer方法  

```java
/**
* 入参说明
* 如果nextTab为null，表示首次发起扩容
* 如果nextTab不为null，表示协助数据迁移
* 每个调用tranfer的线程会对旧table中[transferIndex-stride,transferIndex]位置的节点进行迁移
*/
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //stride理解为"步长"，即数据迁移时，每个线程要负责旧table中多少个桶
    //如果桶较少话，默认一个cpu（一个线程）处理16个桶的数据迁移
    //如果只有一个cpu，处理所有的桶的数据迁移
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    //首次扩容
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            //扩容为原来的两倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //ForwardingNode节点，当旧table的某个桶中的数据全部迁移完成后，用该节点占据这个桶
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    //表示一个桶的迁移工作是否完成，advance==true，表示可以进行下一个位置的迁移
    boolean advance = true;
    //最后一个数据迁移的线程将该值置为true，并进行本轮扩容的收尾工作
    boolean finishing = false; // to ensure sweep before committing nextTab
    //i标识桶索引，bound标识边界
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        //每次自旋前的预处理，主要是定位本轮处理的桶区间
        //正常情况下，预处理完成后：i=transferIndex-1,bound=transferIndex-stride
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        //所有的桶已经迁移完成或者当前线程的任务已经完成
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            //当前所有桶的迁移已经完成
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            //表示当前线程的自己的任务已经完成，扩容线程数减一
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                //判断当前线程是不是最后一个线程，如果不是，直接退出
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                /**
                * 最后一个数据迁移线程要重新检查一遍旧table中的所有桶，看是否都被正确迁移到了新table中
                * 1. 正常情况下，重新检查时，旧table的所有桶都应该时ForwardingNode节点
                * 2. 特殊情况下，比如扩容冲突（多个线程申请到同一个transfer任务）此时当前线程领取的任务会作废，那么最后检查的时候，还要处理因为作废而没有被迁移的桶，把它们正确的迁移到新table中
                */
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        //旧桶为null，不用迁移，直接设置桶为ForwardingNode
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        //旧桶已经完成迁移，直接跳过
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        //旧桶未迁移完成，进行数据迁移
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    /**
                    * 桶的hash>0，说明是链表
                    * 将链表分为两部分：ln链表和lh链表
                    * ln链表会插入到新table的槽i中，lh链表会插入到新table的槽i+n中
                    */
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    /**
                    * 对于红黑树，会先以链表的方式遍历，复制所有节点，像链表一样组装成两个链表hi和lo
                    * 然后看一下这两个链表是否是要进行红黑树转换，最后放到新table中槽为i和i+n的位置
                    */
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

transfer方法的开头会计算一个stride的变量值，这个stride其实就是一个线程处理的桶的取键，也就是补偿  

```java
// stride可理解成“步长”，即数据迁移时，每个线程要负责旧table中的多少个桶
if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
    stride = MIN_TRANSFER_STRIDE;
```

stride有以下几种情况  

1. 如果NCPU==1，说明只有一个CPU，那么`stride==n，也就是这个CPU负责所有的桶的数据的迁移

2. 如果NCPU>1，有以下两种情况：

   2.1 如果`((n>>>3)/NCPU)<MIN_TRANSFER_STRIDE`，那么`stride==MIN_TRANSFER_STRIDE`，此时一个CPU负责16个桶的数据迁移

   2.2 如果`((n>>>3)/NCPU)>=MIN_TRANSFER_STRIDE`，那么`stride==(n>>>3)/NCPU`，此时一个CPU负责`(n>>>3)/NCPU`个桶的数据迁移

首次扩容时，会将table数组变成原来的2倍  

```java
if (nextTab == null) {            // initiating
    try {
        @SuppressWarnings("unchecked")
        //扩容为原来的两倍
        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
        nextTab = nt;
    } catch (Throwable ex) {      // try to cope with OOME
        sizeCtl = Integer.MAX_VALUE;
        return;
    }
    nextTable = nextTab;
    transferIndex = n;
}
```

注意上面的`transferIndex`，这是一个字段，`table[transferIndex-stride,transferIndex-1]`就是当前线程要进行数据迁移的桶区间

```java
/**
 * 扩容时需要用到的一个下标变量.
 */
private transient volatile int transferIndex;
```

整个transfer方法几乎都在一个自旋中完成，从右往左进行数据迁移，transfer的推出点是当某个线程处理完最后的table区段——`table[0,stride-1]`

transfer方法主要有4个分支，即对4种情况进行处理，从易到难分析一下这四种情况  

#### 桶为空  

当旧的桶为空`table[i]==null`，说明这个桶中没有元素，那么就不要进行迁移，仅仅尝试放置一个ForawrdingNode节点，表示该桶已经处理过  

```java
//旧桶为null，不用迁移，直接设置桶为ForwardingNode
else if ((f = tabAt(tab, i)) == null)
    advance = casTabAt(tab, i, null, fwd);
```

#### 桶已完成迁移  

当桶中的数据已经迁移完成，会用ForwardingNode节点占用桶位，表示迁移完成

```java
else if ((fh = f.hash) == MOVED)            // CASE3：该旧桶已经迁移完成，直接跳过
    advance = true;
```

#### 桶未完成迁移  

如果旧的桶的数据没有完成迁移，就要进行迁移，根据节点的类型，分为：链表迁移、红黑树迁移

##### 链表迁移

链表的迁移过程如下：

首先会遍历一遍链表，找到最后一个相邻`runBit`不同的节点。`runBit`是根据`key.hash&table.length`计算出来的，由于table的长度为2的幂次，所以`runBit`只可能为0或者最高位为1

然后，第二次遍历链表，按照第一次遍历的节点为界，将原链表分为两个子链表，再将两个新的子链表，连接到新的table中。子链表在新的table中的索引要么是`i`，要么是`i+n`  

```java
if (fh >= 0) {                  // CASE4.1：桶的hash>0，说明是链表迁移
    /**
     * 下面的过程会将旧桶中的链表分成两部分：ln链和hn链
     * ln链会插入到新table的槽i中，hn链会插入到新table的槽i+n中
     */
    int runBit = fh & n;    // 由于n是2的幂次，所以runBit要么是0，要么高位是1
    Node<K, V> lastRun = f; // lastRun指向最后一个相邻runBit不同的结点
    for (Node<K, V> p = f.next; p != null; p = p.next) {
        int b = p.hash & n;
        if (b != runBit) {
            runBit = b;
            lastRun = p;
        }
    }
    if (runBit == 0) {
        ln = lastRun;
        hn = null;
    } else {
        hn = lastRun;
        ln = null;
    }

    // 以lastRun所指向的结点为分界，将链表拆成2个子链表ln、hn
    for (Node<K, V> p = f; p != lastRun; p = p.next) {
        int ph = p.hash;
        K pk = p.key;
        V pv = p.val;
        if ((ph & n) == 0)
            ln = new Node<K, V>(ph, pk, pv, ln);
        else
            hn = new Node<K, V>(ph, pk, pv, hn);
    }
    setTabAt(nextTab, i, ln);               // ln链表存入新桶的索引i位置
    setTabAt(nextTab, i + n, hn);        // hn链表存入新桶的索引i+n位置
    setTabAt(tab, i, fwd);                  // 设置ForwardingNode占位
    advance = true;                         // 表示当前旧桶的结点已迁移完毕
}
```

![ConcurrentHashMap的链表扩容](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/ConcurrentHashMap%E7%9A%84%E9%93%BE%E8%A1%A8%E6%89%A9%E5%AE%B9.png)  

##### 红黑树迁移

红黑树的迁移迁移过程如下

首先以链表的形式遍历，复制所有节点；然后像链表一样根据高位为0还是1组成两个链表；再对这两个链表进行判断是否需要进行红黑树的转换；最后再放到新table对应的桶中

```java
else if (f instanceof TreeBin) {    // CASE4.2：红黑树迁移
    /**
     * 下面的过程会先以链表方式遍历，复制所有结点，然后根据高低位组装成两个链表；
     * 然后看下是否需要进行红黑树转换，最后放到新table对应的桶中
     */
    TreeBin<K, V> t = (TreeBin<K, V>) f;
    TreeNode<K, V> lo = null, loTail = null;
    TreeNode<K, V> hi = null, hiTail = null;
    int lc = 0, hc = 0;
    for (Node<K, V> e = t.first; e != null; e = e.next) {
        int h = e.hash;
        TreeNode<K, V> p = new TreeNode<K, V>
            (h, e.key, e.val, null, null);
        if ((h & n) == 0) {
            if ((p.prev = loTail) == null)
                lo = p;
            else
                loTail.next = p;
            loTail = p;
            ++lc;
        } else {
            if ((p.prev = hiTail) == null)
                hi = p;
            else
                hiTail.next = p;
            hiTail = p;
            ++hc;
        }
    }

    // 判断是否需要进行 红黑树 <-> 链表 的转换
    ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
        (hc != 0) ? new TreeBin<K, V>(lo) : t;
    hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
        (lc != 0) ? new TreeBin<K, V>(hi) : t;
    setTabAt(nextTab, i, ln);
    setTabAt(nextTab, i + n, hn);
    setTabAt(tab, i, fwd);  // 设置ForwardingNode占位
    advance = true;         // 表示当前旧桶的结点已迁移完毕
}
```

#### 当前是最后一个迁移任务或者出现扩容冲突  

调用transfer方法会自动领用某个区段的桶，进行数据迁移操作，当区段的初始索引`i`变成负数的时候，说明当前线程处理的其实就是最后剩下的桶，并且处理完了

所以首先会跟新sizeCtl变量，将扩容线程数减一，然后做一些收尾工作：设置table指向扩容后的新数组，遍历一遍旧数组，确保每个桶的数据都迁移完成——被ForwardingNode占用

另外，可能在扩容过程中，出现扩容冲突的情况，比如多个线程领用了同一区段的桶，这时任何一个线程都不能进行数据迁移  

```java
if (i < 0 || i >= n || i + n >= nextn) {    // CASE1：当前是处理最后一个tranfer任务的线程或出现扩容冲突
    int sc;
    if (finishing) {    // 所有桶迁移均已完成
        nextTable = null;
        table = nextTab;
        sizeCtl = (n << 1) - (n >>> 1);
        return;
    }

    // 扩容线程数减1,表示当前线程已完成自己的transfer任务
    if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
        // 判断当前线程是否是本轮扩容中的最后一个线程，如果不是，则直接退出
        if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
            return;
        finishing = advance = true;

        /**
         * 最后一个数据迁移线程要重新检查一次旧table中的所有桶，看是否都被正确迁移到新table了：
         * ①正常情况下，重新检查时，旧table的所有桶都应该是ForwardingNode;
         * ②特殊情况下，比如扩容冲突(多个线程申请到了同一个transfer任务)，此时当前线程领取的任务会作废，那么最后检查时，
         * 还要处理因为作废而没有被迁移的桶，把它们正确迁移到新table中
         */
        i = n; // recheck before commit
    }
}
```

