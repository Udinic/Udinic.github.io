---
author: udinic
comments: true
date: 2012-05-08 22:54:02+00:00
layout: post
slug: sqlite-drop-column-support
title: SQLite DROP COLUMN support
wordpress_id: 251
categories:
- Android
- Database
tags:
- Android
- database
- drop column
- onupgrade
- sqlite
- upgrade
---

The new Any.DO version was released to beta testers this week. It’s been a long journey..

On the last couple of months I was working hard on that version. Lots of infrastructure changes, Some of them are Database changes, which I hate because then I have to support upgrading from older DB version. It’s irritating to write it, and even more irritating to test it afterwards (no matter how many useful scripts I wrote to make it easier).

In Android we use SQLite Database, and even though it supports many SQL commands as the more functional Databases around (MySql, Oracle…) it lacks a few useful commands. The ALTER TABLE DROP COLUMN is one of the things I miss the most. There were times where I considered leaving a redundant column in the table, just to avoid the whole process of removing it while upgrading :).

The Internet is full with with people asking about alternatives, and some got useful answers in the shape of pseudo code to get the table on its way to salvation. I got myself into a situation where I needed to drop 4 column on 3 different tables. Damn.

![](http://cdn.memegenerator.net/instances/250x250/20163274.jpg)

That’s it, no more excuses! I’m dealing with this once and for all!

From the [SQlite’s FAQ page](http://sqlite.org/faq.html), one must follow these steps to get this done:


{% highlight sql %}
BEGIN TRANSACTION;
CREATE TEMPORARY TABLE t1_backup(a,b);
INSERT INTO t1_backup SELECT a,b FROM t1;
DROP TABLE t1;
CREATE TABLE t1(a,b);
INSERT INTO t1 SELECT a,b FROM t1_backup;
DROP TABLE t1_backup;
COMMIT;
{% endhighlight %}




The main problem here – not a generic solution. I need to know the list of column I want to keep, instead of just the columns I want to remove. I solved this problem by using the following SQL command:


{% highlight sql %}
PRAGMA table_info(table_name);
{% endhighlight %}




This will query the list of columns this table has, among with their properties (type, default value etc.). Using this, I wrote a function to get all the columns of a certain table:


{% highlight java %}
public List<String> getTableColumns(String tableName) {
    ArrayList<String> columns = new ArrayList<String>();
    String cmd = "pragma table_info(" + tableName + ");";
    Cursor cur = getDB().rawQuery(cmd, null);

    while (cur.moveToNext()) {
        columns.add(cur.getString(cur.getColumnIndex("name")));
    }
    cur.close();

    return columns;
}
{% endhighlight %}




Now we can proceed to the the dropColumn() implementation:


{% highlight java %}
private void dropColumn(SQLiteDatabase db,
		ConnectionSource connectionSource,
		String createTableCmd,
		String tableName,
		String[] colsToRemove) throws java.sql.SQLException {

	List<String> updatedTableColumns = getTableColumns(tableName);
	// Remove the columns we don't want anymore from the table's list of columns
	updatedTableColumns.removeAll(Arrays.asList(colsToRemove));

	String columnsSeperated = TextUtils.join(",", updatedTableColumns);

	db.execSQL("ALTER TABLE " + tableName + " RENAME TO " + tableName + "_old;");

	// Creating the table on its new format (no redundant columns)
	db.execSQL(createTableCmd);

	// Populating the table with the data
	db.execSQL("INSERT INTO " + tableName + "(" + columnsSeperated + ") SELECT "
			+ columnsSeperated + " FROM " + tableName + "_old;");
	db.execSQL("DROP TABLE " + tableName + "_old;");
}
{% endhighlight %}




So what do we have here?

This function is receiving the **db** and the **connectionSource** as arguments, as for the onUpgrade() method, which is the main entry point for the whole Database upgrading process.
The **createTableCmd** is the “CREATE TABLE… “ command to create the new table. You must have that command in order to create the table on a new installation of the app.
The **tableName** is...well...the name of the table.
The **colsToRemove** is an array of columns we want to remove from the table. We can also easily create a convenient function to receive only a String of one column name.

That’s it! Let the columns’ dropping frenzy begin! You can also use this method to support the missing RENAME COLUMN command. The badly-named columns days are over, no more columns like “task_title_which_is_shared_via_email”. Just rename them!

On my next posts I’ll publish some of my useful scripts to help me develop and test more efficiently.
