---
layout:     post
title:      SQL Server Lifesavers - The Dedicated Administrator Connection
date:       2015-09-02
tags:       [sqlserver, dac, troubleshooting]
---
One of the "emergency" tools Microsoft SQL Server provides to DBAs is the [DAC](https://msdn.microsoft.com/en-us/library/ms189595(v=sql.110).aspx), or Dedicated Administrator Connection. The DAC is a dedicated connection listener that SQL Server reserves for system administrators. If a runaway process is consuming all of the connections to your server, or something else is seriously wrong, the DAC can provide you a way to access SQL Server and clean up the mess.

The trouble, though, is that the DAC is only enabled by default for local connections (using named pipes) to non-clustered instances of SQL Server. If you connect to your SQL servers over TCP/IP, or if you have clustered instances, you need to enable Remote Admin Connections to access the DAC.

Enabling Remote Admin Connections
---------------------------------
Enabling the Remote Admin Connections setting is a simple matter of running the following SQL query on your server(s):

{% highlight sql %}
sp_configure 'remote admin connections', 1;
GO

RECONFIGURE;
GO
{% endhighlight %}

You can use a SQL Server [Central Management Server](https://msdn.microsoft.com/en-us/library/bb895144(v=sql.110).aspx) (CMS) to enable this feature across multiple instances, or write a script to accomplish the same purpose on a list of servers you specify. If your SQL Server installation is scripted, you can add this step to your install script to ensure it's enabled across your environment.

To verify that the setting is enabled on a given server, run this query:

{% highlight sql %}
sp_configure 'remote admin connections';
GO
{% endhighlight %}
