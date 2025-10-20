+++
title = "Solving Sad Servers — Part I"
description = "Unraveling server snags"
date = 2024-05-10T00:00:00+00:00

[taxonomies]
tags = ["linux", "debugging", "troubleshooting", "servers", "command-line"]

[extra]
toc = true
+++

![Sad Servers](https://cdn-images-1.medium.com/max/2406/1*8zl4RiXM18WjY_IOBJCk4Q.png)

Recently, while finding ways to enhance my skills in troubleshooting and debugging servers, I came across [sadservers.com](https://sadservers.com/). In this article, we’ll work on resolving server issues🐞🐞🐞

### Problem 1:

> A developer created a testing program that is continuously writing to a log file _/var/log/bad.log_ and filling up disk. You can check for example with tail -f /var/log/bad.log. This program is no longer needed. Find it and terminate it.

### Solution:

```sh
sudo lsof /var/log/bad.log

or

sudo fuser -v /var/log/bad.log
```

The first step is to identify the writing process to the log file. Command-line tools like [lsof](https://explainshell.com/explain?cmd=sudo+lsof+%2Fvar%2Flog%2Fbad.log) (list open files) or [fuser](https://explainshell.com/explain?cmd=sudo+fuser+-v+%2Fvar%2Flog%2Fbad.log) (identify processes using files or sockets) are used to find out which process has the log file /var/log/bad.log open. And kill the process using `sudo kill -9 PID`

### Problem 2:

> There’s a web server access log file at /home/admin/access.log. The file consists of one line per HTTP request, with the requester’s IP address at the beginning of each line.
> Find what’s the IP address that has the most requests in this file (there’s no tie; the IP is unique). Write the solution into a file /home/admin/highestip.txt

### Solution:

```sh
awk '{print $1}' /home/admin/access.log | sort | uniq -c | sort -nr | head -n 1 | awk '{print $2}' > /home/admin/highestip.txt
```

- Extract the first field (which is the IP address) from each line of the access log file using `awk '{print $1}' /home/admin/access.log`

- Sort the IP addresses in ascending order using `sort`

- Count the occurrences of each unique IP address using `uniq -c`

- Sorts the IP addresses based on the count in descending order, with the IP address that has the most requests at the top using `sort -nr`

- Select the first line (which contains the IP address with the most requests) — `head -n 1`

- Extracts only the IP address from the selected line awk `'{print $2}'`

- Finally, write the IP address to the file — /home/admin/highestip.txt

### Problem 3

> Alice the spy has hidden a secret number combination, find it using these instructions:
>
> 1. Find the number of **lines** with occurrences of the string **Alice** (case sensitive) in the \*_.txt_ files in the _/home/admin_ directory
> 2. There’s a file where **Alice** appears exactly once. In that file, in the line after that “Alice” occurrence there’s a number. Write both numbers consecutively as one (no new line or spaces) to the solution file.

### Solution:

```sh
grep -c 'Alice' /home/admin/*.txt
echo -n 411 > /home/admin/solution
grep Alice -A 1 /home/admin/1342-0.txt
echo 156 >> /home/admin/solution
```

- Find the number of lines with occurrences of the string Alice in the `*.txt` file and add it to the solution

- Search for the matching lines along with the lines that follow the string “Alice” in the file `1342–0.txt` and add it to the solution

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/p/440bb3151de7)

</center>

**References:**

[1] [https://sadservers.com/](https://sadservers.com/)

[2] [https://explainshell.com/](https://explainshell.com/)
