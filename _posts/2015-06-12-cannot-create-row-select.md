---
layout:     post
title:      “Cannot create a row” Error From a SELECT Query
date:       2015-06-10 12:31:19
tags:       [sqlserver]
---
Rather than doing the obligatory “hi, this is me, welcome to my blog” post, I thought I’d start things off here with a quick post about an interesting problem I heard from a developer the other week. The actual problem was pretty logical, once I figured out what was going on, but it wasn’t totally obvious at first.

<!-- more -->

The email from the developer said, “I’m getting an INSERT error from SQL Server, but I don’t understand why, because this is a SELECT query – I’m not inserting anything!” I’ve changed some table and column names around, but this is more or less what the query looked like:

{% highlight sql %}
select
     tfn.NoteText, r.EntryDate, s.LastName,
     tfn.fnID, s.FirstName, s.DisplayName
from
     tfn
     left join r on tfn.RefNum = r.RefNum
     left join s on s.StaffID = t.StaffID
order by
     tfn.fnID desc;
{% endhighlight %}

I loaded up the query in SQL Server Management Studio and tried to run it. Sure enough, I got the following error:

> Msg 511, Level 16, State 1, Line 1
> Cannot create a row of size 8100 which is greater than the allowable 
> maximum row size of 8060.

Why was SQL Server trying to “create a row”? This was a relatively simple SELECT query! When I dug into the table structure, I found the answer. For reasons known only to some long-ago developer, the field “tfn.NoteText” in the query was declared to be of type “nchar(4000)”. This meant it would occupy 8,000 bytes in the fixed-width portion of the Worktable SQL Server was creating to process the joins. The additional fields being included in the join were long enough to exceed the 8,060 byte maximum size for a single data row in SQL Server. Voila, error.

The “correct” solution to the problem, of course, is to change the definition of the tfn.NoteText field to use an nvarchar(4000) data type. Since the overwhelming majority of the values in this field are 100 characters or less, this would be far more sensible all the way around – and would eliminate the error the developer was seeing. The short-term fix the developer came up with (casting tfn.NoteText as an nvarchar(4000) in her SELECT) works too, because it makes the column variable-width in the worktable, but it probably isn’t ideal from a performance standpoint.

So, the moral of the story is that even a SELECT query can insert, and those inserts still have to fit on the data pages.