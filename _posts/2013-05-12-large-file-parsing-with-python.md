---
layout: post
title: Large file parsing with python
---

This week I came across to an interesting problem. While I was parsing a big file using sed I came across with an error "Argument list too long". I knew that error and sed was right I was doing my job on a big file with long lines. Also I had unnecessary loops in my code which I wanted to eliminate, maybe with a hash like structure.

So I started looking for a solution. My friend offered me Perl&rsquo;s Tie::File module, but I was too lazy to install needed RPM&rsquo;s on my machine; also I needed hash like structure(Tie::File::AsHash maybe solves that problem, whatever).

And then I came across with Python&rsquo;s shelve module. That was my solution. Doing parsing using Python&rsquo;s dictionary without memory concerns because everything is kept in a DB like file, great!

Basically what shelve module does is you use dictionary like object in your code and that dictionary is kept on the file-system, not memory.

Enough talk let&rsquo;s show some code. As an example I will implement a basic version of GNU join command to illustrate how shelve works.

Assume we have two files.

First one has COURSE_NAME | STUDENT_NAME1, STUDENT_NAME2
Second one has STUDENT_NAME | COURSE_STUDENT_TAKES

Here is our code
{% highlight python %}
import shelve
 
# Files to merged
FILE1 = '/tmp/file1'
FILE2 = '/tmp/file2'
 
# Shelve file
SHELVE = '/tmp/shelve'
 
# Open shelve file
my_dict = shelve.open(SHELVE)
 
# Open first file and create the dict
with open(FILE1, 'r') as f1:
    for line in f1:
        s_line = line.strip().split('|')
 
        # Append file&rsquo;s elements to dict
        my_dict[s_line[0]] = [s_line[1]]
 
# Open second file and create the dict
with open(FILE2, 'r') as f2:
    for line in f2:
        s_line = line.strip().split('|')
 
        # Append file&rsquo;s elements to dict&rsquo;s list
        my_dict[s_line[1]] = my_dict[s_line[1]] + [s_line[0]]
 
# Print out the dictionary
for key in my_dict.keys():
    print key + '|' + ','.join(my_dict[key])
 
# Close shelve file
my_dict.close()
{% endhighlight %}

Example input/output will be like below.
{% highlight python %}
======== File1 ========
math|osman
phys|ayse,fatma
 
======== File2 ========
hasan|phys
huseyin|math
 
======== Output ========
math|osman,huseyin
phys|ayse,fatma,hasan
{% endhighlight %}

Happy hacking.

Edit 1:
It&rsquo;s really slow with big data, be careful.
