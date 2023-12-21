---
title: HashMap
date: 2023/11/29 10:12
tags: 
  - 知识点
categories: 知识点
top_img: ./img/41.jpg
cover: ./img/29.jpg

---



## HashMap

关于hashmap的几点思考

### 1.hash函数是对key做处理，利用int 类型的hashCode()函数，获取32位hash 值，然后前32位与后32位做亦或获得。

```java
static final int hash(Object key){
     int h;
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//扰动，这样前16位和后16位都会对hash值造成影响 
}
```

### 2.获取数组坐标是用length-1做掩码，取hash 后几位

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>,Cloneable,Serializable{
    
    static class Node<K,V> implements Map.Entry<K,V>{
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        
        Node(int hash,K key,V value,Node<K,V> next){
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        

    
    transient Node<K,V>[] table;
    transient Set<Map.Entry<K,V>> entrySet;
    
    final V putVal(int hash,K key,V value, boolean onlyIfAbsent,boolean evict){
        Node<K,V>[] tab; Node<K,V> p; int n,i;
        if((tab = table) == null || ( n = tab.length) == 0)//非空，如果为空resize
            n = (tab = resize()).length;
        if((p = tab[i = (n-1) & hash]) == null)//数组坐标为length-1 与 hash的与（相当于坐标减一做一个掩码），没有被占就创建新节点
            tab[i] = newNode(hash,key,value,null);
        else{
            //该数组节点已经有值
            Node<K,V> e; K k;
            if(p.hash == hash && //节点相同则e=该节点
              ((key = p.key) == key || (key != null && key.equals(k))))
                e= p;
            else if(p instanceof TreeNode) //不同，如果该节点是树节点，新建树节点
                e = ((TreeNode<K,V>)p).putTreeVal(this,tab,hash,key,value); 
            else{//既不是相同节点，也不是树节点
                for(int binCount = 0; ;  ++binCount){
                    if((e = p.next) == null){//遍历到最后一个节点，新建node
                        p.next = newNode(hash,key,value,null);
                        if(binCount >= TREEIFY_THRESHOLD - 1) //如果节点数大于域值，转红黑树
                            treeifyBin(tab,hash);
                        break;
                    }
                    if(e.hash == hash &&(k = e.key) == key || (key!= null && key.equals(k))))
                        break;//遍历到相同节点，跳出循环
                    p = e;
                }
            }
            if(e!= null){
                V oldValue= e.value;
                if(! onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++·;
        if(++size > shreshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
}
```



### 3.节点数超过8转红黑树，treeifyBin（linked node转tree）

```java
final void treeifyBin(Node<K,V>[] tab ,int hash){
    int n, index; Node<K,V> e;
    if(tab == null || (n = tab.length)< MIN_TREEIFY_CAPACITY)
        resize();
    else if((e = tab[index = (n-1)& hash]) != null){
        TreeNode<K,V> hd = null,tl = null;
        do{
            TreeNode<K,V> p = replacementTreeNode(e,null);
            if(tl == null)
                hd = p;
            else{
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        }while ((e = e.next) != null);
        if((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```



### 4.新增树节点，putTreeVal()

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V>{
      TreeNode<K,V> parent;
      TreeNode<K,V> left;
      TreeNode<K,V> right;
      TreeNode<K,V> prev;
      boolean red;
      TreeNode(int hash,K key,V val,Node<K,V> next){
          super(hash,key,val,next);
      }
            
      final TreeNode<K,V> root(){
            for(TreeNode<K,V> r = this,p;;){
                if((p = r.parent) == null)
                   return r;
                r = p;
            }
        }
     }
        
     public final int hashCode(){
         return Objects.hashCode(key) ^ Objects.hashCode(value);
     }
}
    

/**
* 新增红黑树节点
*/

final TreeNode<K,V> putTreeVal(HashMap<K,V> map,Node<K,V>[] tab
                              ,int h, K k,V v){
    Class<?> kc = null;
    boolean searched = false;
    TreeNode<K,V> root =  (parent != null) ? root() : this;
    for(TreeNode<K,V> p = root;;){//获取根节点p
        int dir,ph; K pk;
        if((ph = p.hash) > h) //根节点hash大于当前节点hash dir=-1
            dir = -1;
        else if (ph < h)   //根节点hash小于当前节点hash dir=1
            dir = 1;
        else if((pk = p.key) == || (k != null && k.equals(pk))) //根节点和添加节点相同
            return p;
        else if((kc == null && (kc = comparableClassFor(k)) == null) || 
               (dir = compareComparables(kc,k,pk)) == 0){
            if(!searched){
                TreeNode<K,V> q,ch;
                searched = true;
                if(((ch = p.left) != null &&
                   (q = ch.find(h,k,kc)) != null) ||
                  ((ch = p.right) != null && 
                  (q = ch.find(h,k,kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k,pk);
        }
        
        TreeNode<K,V> xp = p;
        if((p = (dir <= 0) ? p.left : p.right) == null){
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h,k,v,xpn);
            if(dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if(xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            moveRootToFront(tab,balanceInsertion(root,x));
            return null;     
        }
    }
}
```



### 5.扩容resize

```java
final Node<K,V>[] resize(){
    Node<K,V>[] oldTab = table; //取oldTab
    int oldCap = (oldTab == null) ?0 : oldTab.length; //取旧oldTab的length
    int oldThr = threshold;  //域值
    int newCap, newThr = 0;
    if(oldCap > 0){
        if(oldCap >= MAXIMUM_CAPACITY){
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
               oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; //域值翻倍
    }
    
    else if(oldThr > 0)
        newCap = oldThr;
    else{
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if(newThr == 0){
        float ft = (float) newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                 (int) ft:Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if(oldTab != null){
        for(int j = 0;j<oldCap;++j){
            Node<K,V> e;
            if((e = oldTab[j]) != null){
                oldTab[j] = null;
                if(e.next == null)
                    newTab[e.hash & (newCap -1)] = e;
                else if(e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this,newTab,j,oldCap);
                else{
                    Node<K,V> loHead = null,loTail = null;
                    Node<K,V> hiHead = null,hiTail = null;
                    Node<K,V> next;
                    do{
                        next = e.next;
                        if((e.hash & oldCap) == 0){
                            if(loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else{
                            if(hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    }while((e = next) != null);
                    if(loTail != null){
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if(hiTail != null){
                        hiTail.next = null;
                        newTab[j + oldCap]=hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

