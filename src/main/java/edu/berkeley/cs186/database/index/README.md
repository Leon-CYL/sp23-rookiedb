# Project 2: B+ Trees(UC Berkeley CS 186 spring 2023 version)

## Description:

> B+ Tree is a data structure that operate on top of the data files to speed up reads on specific key. Also, many application use B+ Trees in a Database Management System(DBMS) to efficiently look up the data that are store inside the data files.

## B+ Tree Properties:

1. The number `d` is the order of the B+ Tree. Each node(except the root node) must have `d` ≤ x ≤ `2d` entries. The entries of each node must be sorted.
2. In between each entry of an inner node, there is a pointer to a child node. Since there are at most `2d` entries in a node, inner nodes may have at most `2d+1` child pointers.
3. The keys in the children to the left of an entry must be less than the entry while the keys in the children to the right must be greater than or equal to the entry.
4. All leafNodes are at the same depth.


## Implementation:

### Overview:
> In this project, I implemented the `get()`, `getLeftmostLeaf()` (`LeafNode` and `InnerNode` only), `put()`, `remove()` and `bulkload()` method in `LeafNode.java`, `InnerNode.java` and `BPlusTree.java`. 

### [LeafNode.java](LeafNode.java):

> The `get` function returns the LeafNode itself with a given `key`.
```java
public LeafNode get(DataBox key) {
    // TODO(proj2): implement
    return this;
}
```

> The `getLeftMostLeaf` function returns the LeafNode itself same as the `get` function.

```java
public LeafNode getLeftmostLeaf() {
    // TODO(proj2): implement
    return this;
}
```

> The `put` function takes two arguments: `key` the value that used to find the corresponding LeafNode in the B+ tree and `rid` is the record ID that we put inside the data file after we find the correct LeafNode using the giving `key`. This function returns `Optional.empty()` if the size of the keys in the corresponding LeafNode is less than or equal to `order * 2`, else the current LeafNode splits into two LeafNode and left LeafNode will have the first `order` keys and record IDs of the original LeafNode and the right LeafNode will have the remaining keys and record ID of the original LeafNode, and finally return the first key of the right LeafNode.  

```java
public Optional<Pair<DataBox, Long>> put(DataBox key, RecordId rid) {
    // TODO(proj2): implement
        
    if (keys.contains(key)) {
        throw new BPlusTreeException("Duplicate keys.");
    }
    
    int order = metadata.getOrder();
    int count = InnerNode.numLessThanEqual(key, keys);
    keys.add(count, key);
    rids.add(count, rid);


    if (order * 2 >= keys.size()) {
        sync();
        return Optional.empty();
    }

    List<DataBox> rightKeys = keys.subList(order, 2 * order + 1);
    List<RecordId> rightRids = rids.subList(order, 2 * order + 1);

    LeafNode rightLeaf = new LeafNode(metadata, bufferManager, rightKeys, rightRids, rightSibling, treeContext);
    long pageNum = rightLeaf.getPage().getPageNum();
    this.keys = keys.subList(0, order);
    this.rids = rids.subList(0, order);
    this.rightSibling = Optional.of(pageNum);
    sync();
    return Optional.of(new Pair<>(rightKeys.get(0), pageNum));
}
```

> The `remove` function takes one argument: `key` the value that used to find the corresponding LeafNode in the B+ tree. This function will remove the `key` and `rid` in the LeafNode if it exists else do nothing.

```java
public void remove(DataBox key) {
    // TODO(proj2): implement
        
    int index = keys.indexOf(key);
    if (index != -1) {
        keys.remove(index);
        rids.remove(index);
        sync();
    }
}
```

> The `bulkload` function takes two arguments: `data`, an iterator of data that will be added to the B+ Tress, and `fillFactor`, a floating point number that 0.0 < `fillFactor` < 1.0. This function fills up to be 1 record more than `fillFactor * 2 * order` full, then "splits" by creating a right sibling that contains just one record.

```java
public Optional<Pair<DataBox, Long>> bulkLoad(Iterator<Pair<DataBox, RecordId>> data, float fillFactor) {
    // TODO(proj2): implement
        
    if (fillFactor > 1.0) {
        throw new BPlusTreeException("fill Factor can not be greater than 1.0");
    }

    int order = metadata.getOrder();

    while (data.hasNext() && keys.size() < Math.ceil(fillFactor * 2 * order)) {
        helper(data, keys, rids);
    }

    if (data.hasNext()) {

        helper(data, keys, rids);

        List<DataBox> rightKeys = new ArrayList<>();
        rightKeys.add(keys.remove(keys.size() - 1));
        List<RecordId> rightRids = new ArrayList<>();
        rightRids.add(rids.remove(rids.size() - 1));

        LeafNode rightLeaf = new LeafNode(metadata, bufferManager, rightKeys, rightRids,rightSibling, treeContext);
        long pageNum = rightLeaf.getPage().getPageNum();
        this.rightSibling = Optional.of(pageNum);
        sync();
        return Optional.of(new Pair<>(rightKeys.get(0), pageNum));
    }

    sync();
    return Optional.empty();
}
    
private void helper(Iterator<Pair<DataBox, RecordId>> data, List<DataBox> keys, List<RecordId> rids) {     
    Pair<DataBox, RecordId> pair = data.next();
    DataBox key = pair.getFirst();
    RecordId rid = pair.getSecond();
    if (keys.contains(key)) {
        throw new BPlusTreeException("Duplicate keys.");
    }
    int count = InnerNode.numLessThan(key, keys);
    keys.add(count, key);
    rids.add(count, rid);
}
```

### [InnerNode.java](InnerNode.java):

> The `get` function returns a LeafNode from the `children` list in the InnerNode with a given `key`.

```java
public LeafNode get(DataBox key) {
    // TODO(proj2): implement        
    int index = numLessThanEqual(key, keys);
    return getChild(index).get(key);

}
```

> The `getLeftMostLeaf` function returns the first LeafNode that is store in the InnerNode's `children` list .

```java
public LeafNode getLeftmostLeaf() {
    assert(children.size() > 0);
    // TODO(proj2): implement
    return getChild(0).getLeftmostLeaf();
}
```

> The `put` function takes two arguments: `key` the value that used to find the corresponding LeafNode in the B+ tree and `rid` is the record ID that we put inside the data file after we find the correct LeafNode using the giving key. This function returns `Optional.empty()` if the size of the keys in the InnerNode is less than to `order * 2`, else the current InnerNode splits into two InnerNode which the left InnerNode will have the first `order` keys and children of the original InnerNode and the right InnerNode will have the last `order` keys and children of the original InnerNode, and finally return the middle `key` of the original InnerNode.

```java
public Optional<Pair<DataBox, Long>> put(DataBox key, RecordId rid) {
    // TODO(proj2): implement
    int order = metadata.getOrder();
    int index = numLessThanEqual(key, keys);
    BPlusNode leaf = getChild(index);
    Optional<Pair<DataBox, Long>> overflow = leaf.put(key, rid);

    // Add overflow key and child to keys and children from the leaf node.
    if (overflow.isPresent()) {
        DataBox newKey = overflow.get().getFirst();
        long child = overflow.get().getSecond();
        keys.add(index, newKey);
        children.add(index + 1, child);
    }

    if (keys.size() > 2 * order) {
        List<DataBox> innerKeys = new ArrayList<>();
        List<Long> innerChildren = new ArrayList<>();
        for (int i = 0; i < order + 1; i++) {
            innerKeys.add(this.keys.remove(order));
            innerChildren.add(this.children.remove(order + 1));
        }

        DataBox updateKey = innerKeys.remove(0);
        InnerNode putInner = new InnerNode(metadata, bufferManager, innerKeys, innerChildren, treeContext);
        long page = putInner.getPage().getPageNum();
        putInner.sync();
        sync();
        return Optional.of(new Pair<>(updateKey, page));
    }

    sync();
    return Optional.empty();
}
```

> The `remove` function takes one argument: `key` the value that used to find the corresponding LeafNode in the B+ tree. This function will remove the `key` and `rid` in the LeafNode if it exists else do nothing.

```java
public void remove(DataBox key) {
    // TODO(proj2): implement        
    LeafNode leaf = get(key);
    leaf.remove(key);
    sync();
}
```

> The `bulkload` function takes two arguments: `data`, an iterator of data that will be added to the B+ Tress, and `fillFactor`, a floating point number that 0.0 < `fillFactor` < 1.0. This function fills up to be 1 record more than `fillFactor * 2 * order` full, then "splits" by creating a right sibling that contains just one key.

```java
public Optional<Pair<DataBox, Long>> bulkLoad(Iterator<Pair<DataBox, RecordId>> data, float fillFactor) {
    // TODO(proj2): implement

    int order = metadata.getOrder();
    while (data.hasNext() && keys.size() < 2 * order) {
        helper(data, fillFactor);
    }
    if (data.hasNext()){
        helper(data, fillFactor);
        DataBox pushUp = keys.get(order);

        List<DataBox> rightKey = keys.subList(order + 1, 2 * order + 1);
        List<Long> rightChildren = children.subList(order + 1, children.size());
        
        children = children.subList(0, order + 1);
        keys = keys.subList(0, order);

        InnerNode rightInner= new InnerNode(metadata, bufferManager, rightKey, rightChildren, treeContext);
        sync();
        Long curPageNum = rightInner.getPage().getPageNum();
        return Optional.of(new Pair<>(pushUp, curPageNum));
    }
    sync();
    return Optional.empty();
}

private void helper(Iterator<Pair<DataBox, RecordId>> data, float fillFactor) {     
    BPlusNode child = getChild(children.size() - 1);
    Optional<Pair<DataBox, Long>> middle = child.bulkLoad(data, fillFactor);
    
    if (middle.isPresent()) {
        DataBox key = middle.get().getFirst();
        Long newChild = middle.get().getSecond();

        int i = InnerNode.numLessThanEqual(key, keys);
        keys.add(i, key);
        children.add(i + 1, newChild);
    }
}
```

### [BPlusTree.java](BPlusTree.java):

> The `get` function returns a `Optional<RecordId>` if a LeafNode exist with given `key` else returns `Optional.empty()`.

```java
public Optional<RecordId> get(DataBox key) {
    typecheck(key);
    // TODO(proj4_integration): Update the following line
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): implement
    LeafNode leaf = root.get(key);
    if (leaf != null) {
        return leaf.getKey(key);
    } else {
        return Optional.empty();
    }
}
```

> The `put` function takes two arguments: `key` the value that used to find the corresponding LeafNode in the B+ tree and `rid` is the record ID that we put inside the data file after we find the correct LeafNode using the giving key. This function create a new InnerNode when not `Optional.empty()` is returned from calling the `put` function recursively to the LeafNode and update the `root` using the helper method `updateRoot`. 

```java
public void put(DataBox key, RecordId rid) {
    typecheck(key);
    // TODO(proj4_integration): Update the following line
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): implement    
    // Note: You should NOT update the root variable directly.    
    // Use the provided updateRoot() helper method to change
    // the tree's root if the old root splits.
    Optional<Pair<DataBox, Long>> temp = root.put(key, rid);
    if (temp.isPresent()) {
        List<DataBox> keys = new ArrayList<>();
        List<Long> children = new ArrayList<>();
        keys.add(temp.get().getFirst());
        children.add(root.getPage().getPageNum());
        children.add(temp.get().getSecond());
        BPlusNode newNode = new InnerNode(metadata, bufferManager, keys, children, lockContext);
        updateRoot(newNode);
    }
}
```

> The `remove` function takes one argument: `key` the value that used to find the corresponding LeafNode in the B+ tree. This function will remove the `key` and `rid` in the LeafNode if it exists else do nothing.

```java
public void remove(DataBox key) {
    typecheck(key);
    // TODO(proj4_integration): Update the following line
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): implement    
    root.remove(key);
}
```

> The `bulkload` function takes two arguments: `data`, an iterator of data that will be added to the B+ Tress, and `fillFactor`, a floating point number that 0.0 < `fillFactor` < 1.0. This function is similar to the `put` function, it creates a new InnerNode when not `Optional.empty()` is returned from calling the `bulkload` function and update the `root` using the helper method `updateRoot`. 

```java
public void bulkLoad(Iterator<Pair<DataBox, RecordId>> data, float fillFactor) {
    // TODO(proj4_integration): Update the following line    
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): implement    
    // Note: You should NOT update the root variable directly.    
    // Use the provided updateRoot() helper method to change    
    // the tree's root if the old root splits.
    assert (this.root == this.root.getLeftmostLeaf());
    while (data.hasNext()) {
        Optional<Pair<DataBox, Long>> temp = root.bulkLoad(data, fillFactor);
        if (temp.isPresent()) {
            List<DataBox> keys = new ArrayList<>();
            List<Long> children = new ArrayList<>();
            keys.add(temp.get().getFirst());
            children.add(root.getPage().getPageNum());
            children.add(temp.get().getSecond());
            BPlusNode newNode = new InnerNode(metadata, bufferManager, keys, children, lockContext);
            updateRoot(newNode);
        }
    }
}
```

> The `scanAll` function go to the left most LeafNode in the B+ Tree and returns a B+ Tree iterator that contain all the data store in the B+ Tree starting from left to right.

```java
public Iterator<RecordId> scanAll() {
    // TODO(proj4_integration): Update the following line    
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): Return a BPlusTreeIterator.
    LeafNode left = root.getLeftmostLeaf();
    return new BPlusTreeIterator(left, left.scanAll());
}
```

> The `scanGreaterEqual` function go to the corresponding LeafNode with the given `key` and returns a B+ Tree iterator that contain all the data store in the B+ Tree starting from left to right.

```java
public Iterator<RecordId> scanGreaterEqual(DataBox key) {
    typecheck(key);
    // TODO(proj4_integration): Update the following line    
    LockUtil.ensureSufficientLockHeld(lockContext, LockType.NL);

    // TODO(proj2): Return a BPlusTreeIterator.
    if (root != null) {
        LeafNode leftNode = root.get(key);
        return new BPlusTreeIterator(leftNode, leftNode.scanGreaterEqual(key));
    }

    return Collections.emptyIterator();
}
```

> The B+ Tree iterator.

```java
private class BPlusTreeIterator implements Iterator<RecordId> {     
    // TODO(proj2): Add whatever fields and constructors you want here.

    private Iterator<RecordId> rids;
    private LeafNode leaf;

    public BPlusTreeIterator(LeafNode leaf, Iterator<RecordId> rids) {
        this.rids = rids;
        this.leaf = leaf;
    }

    @Override
    public boolean hasNext() {
        // TODO(proj2): implement
        return (rids.hasNext() || leaf.getRightSibling().isPresent());
    }

    @Override
    public RecordId next() {
        // TODO(proj2): implement
        if (!hasNext()) {
            throw new NoSuchElementException();
        }

        if (rids.hasNext()) {
            return rids.next();
        } else {
            leaf = leaf.getRightSibling().get();
            rids = leaf.scanAll();
            return rids.next();
        }
    }
}
```
