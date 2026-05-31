# Chapter 3 - Storage and retrieval

- Ultimate goal to use DBMS is to ensure efficient storage and retrieval of data, so anything that guarantees this two can be used as DBMS.
- Simply appending and searching in file works too, but as data volume increases it becomes harder to search for the data. To make search more efficient special data structures are maintained called "Index". Index are derived from the primary data. Index slows down writes and improves reads.

## Hash Indexes
- Index for key-value data.
- We are only appending to file, we maintain in-memory hash table that map key to byte offset in data file - location at which current value of that key is stored.
- If we keep appending to a file we will run out of disk-space, so file is written in segments, once the segment size exceeds set limit, new segment is created. In background periodic compaction is performed to merge old segments.
- So now each segment has two components, in-memory hash table and on-disc segment.
- To search for a particular key, segments are consulted as per their creation time, recent first - until the key is found.

- To delete a record special deletion record is appended sometimes called tombstone. Tombstone tells the merging process to discard any previous values for the deleted keys.
- If database is restarted in-memory hash tables have to be recreated, it can be derived from the segment files on disc, but it takes lot of time, so mostly on-disc snapshots of in-memory hash tables are maintained to make server restarts fast.
- Concurrency control and partially written record are easy to handle as files are append only and immutable.
- Compaction ensures no fragmentation.

- Hash table has to fit in-memory, all keys must be stored in memory.
- Range scans are hard to perform as we have to search for all keys one by one in in-memory hash table and then follow memory offset in on-disc file to fetch the value.
  
## SSTables and LSM-Trees
- Sorted string table -SSTable - we require that sequence of key-value pair is sorted by key.
- Merging can be done by mergesort alike algorithm, start reading input files side by side, copy the lowest key to the output file, if key is present in multiple files, take the value from the most recent segment. This makes output merged segment file also sorted by key.
- Now we don't have to keep index for each key, this indexes can be sparse - generally maintained for few KBs. We can jump to the offset for nearest lowest key and scan from there until the search key is found.
- When a write comes in, add it to in-memory balanced tree structure that maintains sorted order - called memtable.
- Once memtable's size increases threshold, it is flushed onto the on-disc segment file, meanwhile writes can be performed on newer memtable.
 To serve a read request we have to first check in the memtable and if not found on segment files.
- If database crashes memtable is lost, so we have to maintain separate log on disk to recover the memtable.
- If the key is not present in database, we will end up consulting all the segment files, resulting in huge load. Separate bloom filters can be maintained to figure out is key is not present.
- Size tiered compaction - newer and smaller SSTables are successively merged into older and bigger SSTables.
- Level tiered compaction - key range is split into smaller SSTables and older data is moved into separate levels.

## B-Trees
- Most widely used.
- No of references to child pages in one page is called branching factor.
- To update the value for an existing key in a B-tree, you search for the leaf page containing that key, change the value in that page, and write the page back to the disk. To add new key, find the page whose rage encompasses the new key and add it to that page, if there is not enough space to accommodate the new key page is split into two half filled pages, references have to updated to point to newly created pages.
- Updates are performed in-place so any pointer to page stays valid. This requires concurrency control generally implemented by latches.
- On disk write ahead log is maintained for any operation performed for recovery in case crash happens while performing B-Tree modification.
- Write amplification - one write to the database resulting in multiple writes on the disk over the course of the database's lifetime.
 Each key is present in just one place in B-tree, thus it provides some good transaction consistency.
- If compaction is not implemented properly in LSM, and if it fails to keep up with incoming writes, unmerged segments will keep increasing.

## Other indexing structures
- A primary key uniquely identifies one row in a relational table, or one document in a document database, or one vertex in a graph database.
- Several secondary indexes can be created that can improve queries performing joins.
- Unlike primary index, secondary indexes are not unique. Either we have to add row id and make it unique or we can make each value in index a list of matching row identifiers. Either way both B-tree and log-structured indexes can be used as secondary indexes.
 Key in the index is what query searches for, value can be 1. the actual row in question or 2. pointer to where this row is stored - in that case data is stored in heap file that maintains no order (are generally append only).
- In case of heap file, updates are performed in-place if new value can be accommodated, or a pointer pointing to new place is left behind. If we store value in index itself we are duplicating data and in case of update all indexes have to be updated.
- But the hope to data file can be too much of a performance penalty in some cases, so indexed row is directly stored in index itself - clustered index.
- Compromise between a clustered index and non-clustered index is known as a covering index or index with included columns, which stores some of a table's columns with the index.

- ### Multi-column indexes
  - concatenated index - several fields are appended are indexed as a single column

## Transaction processing or Analytics
- Transaction processing allowing clients to make low latency reads and writes. Batch processing jobs run periodically.
- Often the number of distinct values in a column is small compared to the number of rows. We can take a column with n distinct values and turn it into n separate bitmaps - one bitmap for each distinct value, with one bit for each row. The bit is 1 if the row has the value and 0 if not. If n is bigger, there will be lot of zeroes in most of the bitmaps in that case, bitmaps can be additionally be run-length encoded.
