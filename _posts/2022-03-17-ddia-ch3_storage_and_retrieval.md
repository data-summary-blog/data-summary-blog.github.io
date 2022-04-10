---
layout: post
title: "DDIA Study -  Chapter 3 : Storage and Retrieval (ENG)"
categories: DDIA
author: "Yongchan Hong"
---


# Chapter 3 : Storage and Retrieval

> In this DDIA category, I will post a summary of Designing Data Intensive Applications (DDIA) from chapter 1 to 12.

In this chapter, we will learn about storage and retrieval.

## Data Structures That Power Your Databases
Simple database using db_set and db_get
```
db_set() {
    echo "$1, $2" >> database
}


db_get() {
    grep "^$1" dtabase | sed -e "s/^$1, //" | tail -n 1
    # Find where beginning of line이 is $1, and get reset of the part without $1 (which will be $2) and return the most recent value
```
- `db_set`: we simply append to log
    - `log`: ***append-only, immutable*** data file
    - merging and compaction (throwing away duplicate keys in the log, and keeping only the most recent update for each key) is required 
- `db_get`: since it scans the entire DB file it will have O(n) terrible performance 
    - So we need `index`, which is ***additional*** structure derived from the primary data. However while we can speed up read queries, this can incur overhead on writes.
    - `key-value index`(≈`primary key`): ***uniquely*** identifies one row/document/vertex
        1. Hash Index (log-structured)
        2. LSM-Trees (log-structured)
        3. B-Trees (page-oriented)
    - `secondary index`: the indexed values are not unique

## Hash Indexes
- Hash Indexes use hash map or hash table for indexing, and it keeps in-memory hash map where every key is mapped to a byte offset (amount that should be added from address to achieve second address) in the data file
- Stoarage engine like `Bitcask` use hash indexes. Since all hash maps are in-memory, it stores all keys and thus can read and write in high performance
- Hash Indexes are good for situation where the value for `key is updated frqeuently` and where there are `large number of writes but have limited number of keys`.
    - Ex) Key: URL of video, Value: How often video has been played
- Hash Indexes will have `Segment` file where each segment has its own in-memory hash map and writes until it approached certain size. When read, the most recent segment will be searched first.
- Segment file will go through `merging and compaction`. In background thread, it will read old segment and compact/merge with most recent/unique key values. When compaction/merging is done, it will create latest segement file and all read requests will be switched to the new merged segment. Old segments will be deleted. 
> Number of segment should be small since we want small amount of hash map to be checked
- Issues in implementation
    - File format: binary format (length in bytes + raw string)
    - Deleting records: `tombstone` (All values before tombstone will be ignored)
    - Crash recovery: if DB crashes, in-memory hash maps are lost
        - Bitcask: store snapshot of each hash map on disk
    - Partially written records: DB crash while appending record to the log
        - Bitcask: checksum to find out damaged log
    - Concurrency control: writes are appended to the log in a strictly sequential order
        - write: only one thread
        - read: multiple threads
- Pro: Since append-only log is sequential write operation, it is much faster than random writes since it does not require intermediate seek during write. Also concurrency and crash recovery is much simpler. Finally data fragmentation can be avoided by merging.
- Con: Since hash maps must fit in memory, there are limits on number of keys. Also resolving hash collisions are not easy. Most importantly, `it is very inefficient for range queries`.

## LSM-Trees(Log-Structured Merge-Tree)
- LSM Trees will have segment file format of `SSTable (Sorted String Table)`
    - key-value pairs are sorted by key
    - unique key in segment
    - each segment has its own sparse index
- LSM Trees are used in LevelDB, RocksDB, Cassandra, HBase, Lucene
    - `term dictionary`: mapping from term to postings list is kept in SSTable-like sorted file (merged in the background as needed)
- LSM Trees are better than Hash Indexes!
    - no longer need to keep index of all the keys in memory
        - can only have some keys and we can know it is between two sparse index since it is ordered.
    -  group records into a block and compress it before writing it to disk. Each entry of sparse index then points at the start of a compressed block. This save disk space and reduce I/O bandwitdth use.
    - merging segments is simple and efficient even if the files are bigger than the available memory (mergesort)
-  How can we construct and maintain SSTables: How to sort by key
    - Use structure like AVL tree or red-black tree
    - Write
        1. write → `memtable`: *in-memory* balanced tree data structure
        2. when memtable gets bigger → write it out *to disk* as an SSTable file
    - Read
        1. find the key in the memtable andread on-disk segments by latest
    - Merge and compact from time to time
- How do we have optimze performance?
    - Use `Bloom filters` which can tells if a key does not appear in the DB. This can prevent slowing down issue when looking up keys that do not exist in the DB.
    - How to determine the order & timing of merging & compaction
        - `Size-tiered`: HBase, Cassandra
            - newer and smaller SSTables successively merged into older and larger SSTables
        - `Level-tiered`: LevelDB, RocksDB, Cassandra
            - key range is split up into smaller SSTables and older data is moved into separate "levels"
            - allows compaction to proceed more incrementally and use less disk space 

## B-Trees
- The most common type of index
- log-structured vs. page-oriented indexes
    - log-structured
        - break the database down into *variable-size* `segments`
        - write a segment *sequentially*
    - page-oriented
        - break the database down into *fixed-size* `pages`(`blocks`): hardware-friendly
            - each page is identified by an address or location thereby allow reference
            - `branching factor`: number of references to child pages
            - `leaf page`: final individual key.
            - one page will be selected `root` and search will be started from this root.
        - read or write one page at a time
- update, add, delete: Keep it ***balanced*** (n keys always has depth of O(log n))
- Making B-Trees reliable
    - some require overwriting several different pages ➡ may crash before done
        - `write-ahead log`: append-only file on disk. written before applied
    - concurrency control: updating pages complicated
        - `latches` (lightweight locks)
- B-Tree optimizations
    - copy-on-write scheme: crash recovery & concurrency control 
        - a modified page is written to a different location, and a new version of the parent pages in the tree is created (which will point to modifeid page)
    - `B+ Tree`: data pointers are present only at the leaf nodes and additional pointers where each leaf page reference to its siblings exist. Commonly used.

## B-Trees vs. LSM-Trees
- `faster writes`: LSM-Trees
    - B-Trees will have to write twice (write-ahead log and tree page)
    - LSM Trees can have better compaction
    - LSM just have to write sequential SS table unlike B Trees which has to update multiple pages in tree
- `faster reads`: B-Trees
    - LSM-Trees: have to check several different structures and SSTables at different stages of compaction
    - B Tree will have exactly one index

## Other Indexing Structures
- `secondary index` vs. `key-value index`
    - the indexed values are not unique vs. unique
- Storing values within the index
    - `nonclustered index`: reference to the row
        - `heap file`: the place where rows are stored
        - avoids duplication when multiple secondary indexes are present
    - `clustered index`: actual row/document/vertex
        - hop from the index to the heap file is too much of a performance penalty for reads
        - store the indexed row directly within an index
    - `covering index`
        - stores some of a table's columns within the index
- Multi-column indexes: query multiple columns simultaneously
    - `concatenated index`
        - combine several fields *in defined order* into one key by appending one column to another
        - useless if you want to query subsets
    - `multi-dimensional index`
        - geospatial data example
        - general case
- `In-memory Database`
    - keep datasets entirely in memory (maybe distributed among several machines)
    - durability issue: when DB restarts
        - if caching use only then data lost OK (ex. Memcached)
        - many methods: special hardware, writing log of changes to disk, writing periodic snapshots to disk, replicating the in-memory state to other machines ...
    - Databases
        - VoltDB, MemSQL, Oracle TimesTen, RAMCloud
        - Redis, Couchbase: weak durability by writing to disk asynchronously
    - Performance advantage because...
        - faster reads (X)
            - disk-based storage engine may never read from disk: the OS caches recent disk blocks in memory
        - faster writes (O)
            - avoid the overheads of encoding in-memory data structures in a format that can be written to disk
    - `anti-caching`
        - evict the least recently used data from memory to disk when there is not enough memory, loading it back into memory when it is accessed again

### Thanks to Youkyoung Cha for first half part of summary. I paraphrased/reorganized this summary based on her summary.
            

