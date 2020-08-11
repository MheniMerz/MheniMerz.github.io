---
layout: post
section-type: post
title: "The path to LFCE Day2 : Manipulating Files programaticaly"
category: 'linux'
tags: [ 'linux', 'lfce' ]
---

## manipulate files programatically

### How to compare files
The diff command enables us to compare two files, the output of the example below essentially tells us that the first line is the same in both files and that file2 has  two additional lines.

```
    $ diff -u file1 file2
    --- file1       2020-07-28 20:13:22.788000000 +0000
    +++ file2       2020-07-28 20:25:18.644000000 +0000
    @@ -1 +1,3 @@
    this is a file
    +this is also a file
    +this is a bigger file
```

The next example, shows that in order to get to file2 from file1 we need to remove the first line (preceded by -) and add the next two lines (preceded by +)

```
    $ diff -u file1 file2
    --- file1       2020-07-28 20:13:22.788000000 +0000
    +++ file2       2020-07-28 20:44:54.916000000 +0000
    @@ -1 +1,2 @@
    -this is a file
    +this is also a file
    +this is a bigger file
```
### merging different versions of a file with patch
`patch` is used in the scenario where multiple people are working on the same file and have different version, for instance a team of researchers working on a paper might devide the work on members for each to work on different sections.
##### example scenario
researcher1 started working on the introduction and section1 and shared the document with researcher2 who will add section2 and the conclusion, while researcher2 is doing that researcher1 added section3

let's put some content in the files, use vim and go in insert mode to add content to the files
    $vim researcher1-file
    $cp researcher1-file researcher2-file
    $vim researcher2-file

now let's check how different our files are, looks like researcher1 is missing section2 and researcher2 is missing section3.

    $ diff -u researcher1-file researcher2-file
    --- researcher1-file    2020-07-30 12:04:47.640000000 +0000
    +++ researcher2-file    2020-07-30 12:01:57.340000000 +0000
    @@ -9,8 +9,13 @@
    remotely. Physicians can monitor their patients anytime and
    anywhere and can update prescription when needed.

    +2. Section2
    +In our proposedmodel we focused on tracking and monitoring
    +of human diseases with respect to prescribed medicine
    +in healthcare domain. User Interface (UI), Semantic Interoperability
    +(SI), and Cloud Services (CS) are three major
    +constituents of IoT-SIM. InUser Interface, doctor and patient
    +interact with each other with the help of IoT devices.

    -3. Section3
    -IoT devices are communicating through sensors.
    -Every device has sensor network API which is used to filter
    -data according to the domain.
    +4. Conclusion
    +In this paper, we have blah blah ...

now we can save the output od the diff command in a file `diff-file` using linux redirects (`>`) and use it to merge our two versions of the research paper.

    $diff -u researcher1-file researcher2-file > diff-file


    $ diff -u researcher1-file researcher2-file > diff-file
    $ patch < diff-file
    patching file researcher1-file
    $ diff -u researcher1-file researcher2-file
    $

and just like that our files are now identical. version conrol systems use `diff` and `patch` extensively to keep the code base updated and avoid conflicts.
[read more about version control systems here...](https://merzouki.com/linux/2020/07/29/The-path-to-LFCE-Day1.html)

### Stream editing files with sed
`sed` stands for stream editor and it's most commonly used to find and replace text in a file but it also has other uses.
we are starting with a simple bash script.
the script simply tests ICMP connectivity to a given address (not that usefull but it's just for demonstration)

    $cat test.sh
     #!/bin/bash
     ping 10.0.0.1

say we want to change the address to test connectivity to the internet instead of our internal gateway, we can use a regular expression `s` to substitute the local address `10.0.0.1` by the google DNS `8.8.8.8`.

   $ sed 's/10.0.0.1/8.8.8.8/' test.sh
    #!/bin/bash
    ping 8.8.8.8

but this only displays the change, and it does not write to the file, to do so use the `-i` flag

    $ sed -i 's/10.0.0.1/8.8.8.8/g' test.sh
    $ cat test.sh
    #!/bin/bash
    ping 8.8.8.8

we can also insert and delete lines, to add a line we use `a` in the regular expression and the `$` sign means we want to add it at the end of the file (we can specify a line number instead)
    $ sed -i '$ a adding line to the end' test.sh
    $ cat test.sh
    #!/bin/bash
    ping 8.8.8.8
    adding line to the end

and to delete the line we just added we use `d` with `sed`
    $ sed -i '$d' test.sh
    $ cat test.sh
    #!/bin/bash
    ping 8.8.8.8
