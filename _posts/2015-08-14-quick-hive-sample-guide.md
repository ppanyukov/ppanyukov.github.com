---
layout: post
title: "Hadoop: Quick Hive sample guide"
excerpt: "A simple guide for getting started with Hive."
---

Assuming that Hive is installed and is running in local standalone mode.

## Create sample data

Create a list of files with these fields, delimited by tabs


    depth, filetype, perms, size, user, group, date, time, dir, filename, fullname, linkto

Can use this command

    find /usr -fprintf files.txt "%d\t%y\t%M\t%s\t%u\t%g\t%AY-%Am-%Ad\t%AH:%AM\t%h\t%f\t%p\t%l\n"

The contents of the file should be something like this:

    0   d   drwxr-xr-x  4096    root    root    2015-08-14  17:03       usr /usr
    1   d   drwxr-xr-x  4096    root    root    2015-08-14  17:55   /usr    src /usr/src
    2   d   drwxr-xr-x  4096    root    root    2015-08-14  17:55   /usr/src    linux-headers-3.2.0-88-virtual  /usr/src/linux-headers-3.2.0-88-virtual
    3   l   lrwxrwxrwx  30  root    root    2015-08-14  17:55   /usr/src/linux-headers-3.2.0-88-virtual virt    /usr/src/linux-headers-3.2.0-88-virtual/virt    ../linux-headers-3.2.0-88/virt


**Make the file accessible to Hive**

Put the file somewhere everyone can get to it:

    # create the directory
    mkdir -p /user/philip/samples/files

    # copy the file there
    cp files.txt /user/philip/samples/files


## Create hive schema 

Run `hive`.

    hive


Create a structured view on top of the file.
Create `EXTERNAL` table because we want to leave the file where it is and
run queries directly against it.


    use demo;

    create external table if not exists files
    (
      depth int comment 'depth of file in the hierarchy',
      ftype string comment 'type of file: f, l, d',
      perms string comment 'permissions like in ls -l',
      size int comment 'file size in bytes',
      owner_user string comment 'the owner user name',
      owner_group string comment 'the owner group name',
      create_date string comment 'date created',
      create_time string comment 'time crated',
      dir string comment 'the parent directory of the file',
      file string comment 'the name of the file without directory',
      fullname string comment 'the full name of the file',
      linkto string comment 'the full name of file this link links to'
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    STORED AS TEXTFILE LOCATION '/user/philip/samples/files'
    ;


## Examine the schemas

**List the databases.**

    hive> show databases;
    OK
    default
    demo
    philip
    Time taken: 0.024 seconds, Fetched: 3 row(s)

**Show tables.**

    hive> show tables in demo;
    OK
    files
    Time taken: 0.048 seconds, Fetched: 1 row(s)


**Show the table structure**

    hive> describe files;
    OK
    depth                   int                     depth of file
    ftype                   string                  type of file
    perms                   string                  permissions like in ls -l
    size                    int                     file size in bytes
    owner_user              string                  the owner user name
    owner_group             string                  the owner group name
    create_date             string                  date created
    create_time             string                  time crated
    dir                     string                  the parent directory of the file
    file                    string                  the name of the file without directory
    fullname                string                  the full name of the file
    linkto                  string                  the full name of file this link links to
    Time taken: 0.127 seconds, Fetched: 12 row(s)

**See SQL which created the table**

    hive> show create table files;
    OK
    CREATE EXTERNAL TABLE `files`(
      `depth` int COMMENT 'depth of file',
      `ftype` string COMMENT 'type of file',
      `perms` string COMMENT 'permissions like in ls -l',
      `size` int COMMENT 'file size in bytes',
      `owner_user` string COMMENT 'the owner user name',
      `owner_group` string COMMENT 'the owner group name',
      `create_date` string COMMENT 'date created',
      `create_time` string COMMENT 'time crated',
      `dir` string COMMENT 'the parent directory of the file',
      `file` string COMMENT 'the name of the file without directory',
      `fullname` string COMMENT 'the full name of the file',
      `linkto` string COMMENT 'the full name of file this link links to')
    ROW FORMAT DELIMITED
      FIELDS TERMINATED BY '\t'
    STORED AS INPUTFORMAT
      'org.apache.hadoop.mapred.TextInputFormat'
    OUTPUTFORMAT
      'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
    LOCATION
      'file:/user/philip/samples/files'
    TBLPROPERTIES (
      'COLUMN_STATS_ACCURATE'='false',
      'numFiles'='0',
      'numRows'='-1',
      'rawDataSize'='-1',
      'totalSize'='0',
      'transient_lastDdlTime'='1439579630')
    Time taken: 0.503 seconds, Fetched: 28 row(s)



## Run queries

**First 20 rows.**

    hive> select * from files limit 3;

    0   d   drwxr-xr-x  4096    root    root    2015-08-14  17:03       usr /usr
    1   d   drwxr-xr-x  4096    root    root    2015-08-14  17:55   /usr    src /usr/src
    2   d   drwxr-xr-x  4096    root    root    2015-08-14  17:55   /usr/src    linux-headers-3.2.0-88-virtual  /usr/src/linux-headers-3.2.0-88-virtual


**Count**

    hive> select count(*) from files;

    60110


**Top 5 files by size**

    hive> select size, fullname from files order by size desc limit 5;

    32348929    /usr/lib/jvm/java-7-openjdk-amd64/jre/lib/rt.jar
    25387008    /usr/lib/jvm/java-7-openjdk-amd64/jre/lib/amd64/server/classes.jsa
    22446368    /usr/lib/x86_64-linux-gnu/libLLVM-3.0.so.1
    20599030    /usr/lib/hive/apache-hive-1.2.1-bin/lib/hive-exec-1.2.1.jar
    17360142    /usr/lib/hive/apache-hive-1.2.1-bin/lib/hive-jdbc-1.2.1-standalone.jar


**Count by file type**

    hive> select ftype, count(*) as count from files group by ftype order by count;

    l   6156
    d   7947
    f   46007

