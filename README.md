pcn_ideas
=========

A place for random ideas that may one day become reality

fsync_timing_interposition
--------------------------

Had an interesting experience the other day.  ZooKeeper, which logs
its fsync() completion time when it exceeds the timeout for its
connected clients, reported that a fsync() had taken about 2 minutes.

This was useful because it led us to talk to AWS and get the instance
checked out.  There were hardware errors that weren't automatically
detected, and we had our answer as to the root cause of an error.

This led to the idea that good code calls fsync() or its brethren.
Because it's performance-critical, however, fsync is rarely
instrumented, but there are times when really good tools like
profilers, dtrace, etc. provide you with insight that you didn't
otherwise have and you think "man, I'd like to have that".

So, I've thought about this and I've got a couple of thoughts:

 1. What matters is that an application knows when it's fsync() is
 slow.  What does slow mean? Slow means slower than normal, so it only
 makes any sense when you have data over time.  So this means
 something that you'd preferrably keep running at all times so that
 you can see when you're working outside of the bounds of your
 expectations.

 2. It's also relevant that by logging fsync, you don't necessarily
 invoke fsync again and again.  Think about these options
 (non-exclusive things to explore): 
 
   1. Log to syslog 
   2. Log to tempfs (less likely to block?)  
   3. Write to log periodically
   4. Write to log on signal
   5. Need to log time taken for fsync, as well as approximate overhead of
      the library call.

 3. It costs extra time to do important things.
   a.  like what the  filedes argument maps to.
   b. Need to find the current name of file in a process.
   c. Need to store file->write time.
   d. Should store overall time spent per write interval per file.
      E.g. use a hash map (http://code.google.com/p/sparsehash/?redir=1 maybe?)
      and a linked list of structs. The hash map is useful to 
      limit the space used.  The latter gives raw data.    
 
 4. Look at other examples.  http://dsc.sun.com/solaris/articles/lib_interposers.html

 