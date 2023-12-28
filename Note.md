# <div style="text-align: center;" markdown="1"> CS 186 Introduction to Database Systems Note
### <div style="text-align: center;" markdown="1"> Author: ChaoYuan Lin

## Introduction:
> This is a personal note that I created for my learning in Database Management System(DBMS) and future review. My note is based on the notes from UC Berkeley's CS 186: Introduction to Database Systems and you can find the notes [here](https://cs186berkeley.net/notes/).

## Overview:

> There are five main components inside the Database Management Systems:

![Database](management.png)
1. Query Parsing & Optimization
2. Relational Operators
3. Files and index Management
4. Buffer Management
5. Disk Space management

> Numbered from the highest level to the lowest level of the Database Management System(DBMS).

## 1. Disks and Files

### Disk
> - Disks are used to cheaply store all of a database’s data, but they incur a large cost whenever data is accessed or new data is written.
> - Disks includes `READ` and `WRITE` which stands for transferring “pages” of data from disk to RAM and transferring “pages” of data from RAM to disk respectively.
> - `Storage Hierarchy`:

![Storage_Hierarchy](hiearchy.png)

### Disk Space Management
> - Disk Space Management is the lowest layer of DBMS and its main purposes include `mapping pages to locations on disk`, `loading pages from disk to memory`, and `saving pages back to disk` and `ensuring writes`.

### Files, Pages, Records

> - The basic unit of data for relational databases is a `record (row)`. These records are organized into `relations (tables)` and can be `modified`, `deleted`, `searched`, or `created` in memory.
> - Each relation is stored in its own file and its records are organized into pages in the file in relation database.
> - Based on the relation’s schema and access pattern, the database will determine:
> 1. Type of file used
> 2. How pages are organized in the file
> 3. How records are organized on each page
> 4. How each record is formatted

### Files Type

- Heap File

1. LinkedList Implementation:
> - In the linked list implementation, each data page contains `records`, a `free space tracker`, and `pointers` (byte offsets) to the `next` and `previous` page.
> - When space is needed, empty pages are allocated and appended to the free pages portion of the list.
> - When free data pages become full, they are moved from the free space portion to the front of the full pages portion of the linked list.
![LinkedListFile](LinkedList.png)

2. Page Directory Implementation:
> - 





