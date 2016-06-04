---
layout: post
title:  "Postgres COPY in java"
date:   2016-06-04 09:45:28
categories: java postgres
---

The fastest way do add latge quantity of data to postgres in to use [COPY][copy]. Well you can also read from postgres using COPY, but that's for some other time. Getting back to topic there's implementation of COPY in java's postgresql driver which I'll show you how to use.

Idea of COPY is to use CSV format to write and read to and from postgres. You can use whole table or just selected columns. Writing basically looks like this ```COPY table_name FROM '/path/to/file/file_itself';``` and reading ```COPY table_name TO '/path/to/file/file_itself';```. You can pass ```STDIN``` and ```STDOUT``` respectively if you want to pass from or to console instead of file. You can set CSV separator by adding ```WITH DELIMITER '|'``` if you want to use something different than comma. Successful COPY run will return number of added rows. So that's theory in a nutshell.

In java whatever you use to connect to DB you need to add some JDBC driver. For connecting to postgresql you obviously will use postresql driver, duh. So when you use it your connection will be implementation of ```org.postgresql.jdbc.PgConnection```. Even when you use connection pool ```PgConnection``` will be wrapped in pool connection, so either way you can get to it.

When you have your connection taken from ```DataSource``` or some other way you can unwrap it or just cast to ```PgConnection```

{% highlight java %}
PgConnection copyOperationConnection = connection.unwrap(PgConnection.class);
{% endhighlight %}

{% highlight java %}
PgConnection copyOperationConnection = (PgConnection) connection;
{% endhighlight %}

Remember to close connection when you're done. Preferably use ```closable-try```. When you don't close your connection unicorn dies and you're an asshole.

Basic usage is to create ```org.postgresql.copy.CopyManager``` from connection and call ```copyIn``` method on it. Usually you pass COPY sql query and input stream and it looks like this:

{% highlight java %}
CopyManager copyManager = new CopyManager(copyOperationConnection);
return copyManager.copyIn("COPY my_target_table FROM STDIN WITH DELIMITER '|'", new FileReader("/path/to/file/file_itself"));
{% endhighlight %}

But that's boring. In most cases you don't have file and just do transformations from other data yourself. You could transform JSONs, XMLs or other incoming formats. In such case you need to do what ```CopyManager``` does underneath in ```copyIn```. Then your code will look:

{% highlight java %}
CopyManager copyManager = new CopyManager(copyOperationConnection);
CopyIn copyIn = copyManager.copyIn("COPY my_target_table FROM STDIN WITH DELIMITER '|'");

try {
    for (String row : rowsToAdd) {
        byte[] bytes = row.getBytes();
        copyIn.writeToCopy(bytes, 0, bytes.length);
    }

        return copyIn.endCopy();
} finally {
    if (copyIn.isActive()) {
        copyIn.cancelCopy();
    }
}
{% endhighlight %}

Trick is to end every row with ```\n``` or it won't know where on line ends and next starts. Another advice is that when connection is used to create ```CopyManager``` you cannot use it for other queries like checking if data is already in DB. You need to open another connection for such checks.

Well that's it. Now you can start writing your own postgres ETLs.

[copy]: https://www.postgresql.org/docs/current/static/sql-copy.html