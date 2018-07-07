---
layout: post
title:  "SQLite - in-memory cache"
categories: [Cache, DB, Development]
tags: [Design, cache, DB, Development]
date:   2018-07-03 08:44:21 -0700
---


SQLite is a self-contained, high-reliability, embedded, full-featured, public-domain, SQL database engine. It is super easy to install and use it in the code. Here I am going to cover how to use SQLite as an in-memory cache. Often times, we might have some metadata like zipcodes/citycodes mapping which are couple MBs in size. It might be an overkill to create a database and maintain them separately if they are not going to change. For those kind of scenarios my goto solution is SQLite which can be used to easily load CSV files as tables. It is super fast to access those data since they are in memory. If you are using docker, then you don't even need to worry about cleaning/recreating DB files used by SQLite.

Below are some generic methods I would like to go through.
1. Creating table from CSV
2. Get values for a column, filter value.


{% highlight python %}

import gzip
import os
import csv
import re
import sqlite3
from sqlite3 import Error

# allows you to access columns using names 
# whereas a plain tuple would make you use numbered indices
def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

db_file = 'database.db'

def create_connection(db_file):
    try:
        conn = sqlite3.connect(db_file)
        conn.row_factory = dict_factory
        conn.text_factory = str  # allows utf-8 data to be stored
        return conn
    except Error as e:
        log.logger.error('Error creating sqlite3 connection %s' % e)
        raise e


# create table from csv
def load_csv_to_sql_db():
    conn = create_connection(db_file)
    c = conn.cursor()
    path = CSV_FILE_PATH
    with gzip.open(path, 'rb') as f:
            reader = csv.reader(f)
            # tablename will be the filename here
            tablename = os.path.basename(f.name).split('.')[0]
            header = True

            for row in reader:
                if header:
                    header = False
                    sql = "DROP TABLE IF EXISTS %s" % tablename
                    c.execute(sql)
                    sql = "CREATE TABLE %s (%s)" % (
                        tablename, ", ".join(
                            ["%s text" % re.sub('\W+', '', # filter out any non alphanumeric characters
                             column) for column in row])) 
                    c.execute(sql)
                    log.logger.info('Data table %s created' % tablename)
                    index_cols = (‘col1’, ‘col2’) #columns that needs index
                    for column in row:
                        if column.lower() in index_cols:
                            index = "%s__%s" % (tablename, column)
                            sql = "CREATE INDEX %s on %s (%s)" % (
                                index, tablename, column)
                            c.execute(sql)
                        insertsql = "INSERT INTO %s VALUES (%s)" % (
                            tablename, ", ".join(["?" for column in row]))
                    rowlen = len(row)
                else:
                    if len(row) == rowlen:
                        c.execute(insertsql, row)

            conn.commit()
    conn.close()

# read a column and filter out based on value passed
def get_value(table_name, col_name, filter_val):
    conn = create_connection(db_file)
    try:
        cur = conn.cursor()
        cur.execute("SELECT * FROM %s"
                    " WHERE %s=?" % (table_name, col_name), (filter_val,))
        row = cur.fetchone() #fetching single row
        return row or {}
    except Exception as e:
        log.logger.error('Error while fetching %s'
                         ' from table : %s and value : %s' % (
                             col_name, table_name, filter_val))
    conn.close()

{% endhighlight %}

Now we have a loaded the data from the CSV into sqlite table and use the interface `get_value` to access the required columns. We call the 'load_csv_to_sql_db' method in before_first_request in flask which is where we load all our inmemory cache values. `sqlite3` is also a great command line database for data analysis. I use it all the time to load CSVs and to understand the data. You can find more detail here [Command Line Shell For SQLite](https://www.sqlite.org/cli.html). The above methods can optimized to your requirement to fetch multiple values, reusing the connection objects, etc. I have just showcased a pattern to use SQLite as an in-memory cache.

