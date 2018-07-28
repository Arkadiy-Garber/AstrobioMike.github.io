---
layout: main
title: Introduction to Unix
categories: [bash, tutorial]
permalink: /stamps2018/unix_intro
---

{% include _unix_toc.html %}

> **Things covered here**  
*  Why familiarity with the command line is important
*  Running commands and general syntax
*  File-system structure and how to navigate
*  Viewing, creating, and manipulating "plain-text" documents
*  Pipes and redirectors
*  Intro to wildcards

<br>

---
<br>

# Some terminology

| Term     | What it is          |
|:-------------:|------------------|
| **`Unix`** | a family of operating systems |
| **`command line`** | a text-based environment capable of taking input and providing output |
| **`shell`** | ambassador to the operating system; translates between us and the computer |
| **`bash`** | a specific type of shell, bash is the most common programming language used at a Unix command-line |

<br>

---
<br>

# Why learn the command line?

*  enables use of non-GUI (Graphical User Interface) tools
*  perform operations on large files quickly
*  reproducibility
*  automation of repetitive tasks
	*  need to rename 1,000 files?
	*  need to assemble 300 genomes?
*  enables use of higher-powered computers elsewhere (server/cloud)
*  it's the foundation for most of bioinformatics

> **The conundrum**  
> A) There is quite a bit of a learning curve, so it takes some time to get to the point where the command line becomes useful and powerful.  
> B) It's hard to know how it will be useful and powerful before you get to that point.  
> C) It's hard to have the motivation and make the time to learn it before you know it's useful and powerful.  
> <center><b>This is our struggle.</b></center>

<br>

---
<br>

# Practice data!
For right now, and only for right now, I would like you to blindly copy and paste the following commands into your terminal window. This is so that we're all working in the same place with the same files. **I promise this will be the only time I ask you to do this.** At the end of this page, we'll revisit this and break it down :) 


```
cd ~
curl -O https://AstrobioMike.github.io/tutorial_files/unix_intro.tar.gz
tar -xvf unix_intro.tar.gz
rm unix_intro.tar.gz
cd unix_intro
```

<br>

---
<br>

# Running commands

The general syntax of a command is as follows: `command argument`. **Spaces are special!** The command line uses spaces to know how to properly break apart things. 

Arguments (often also referred to as "flags" or "options") can be **optional** or **required** based on the command. 

`date` is a command that prints out the date and time. It does not require any arguments:

```bash
date
```
But you can provide optional arguments to `date`. Here we are adding the `-u` argument to tell it to report UTC time instead of local (the "default"): 

```bash
date -u
```

If we tried to enter this without the "space" separating `date` and the argument `-u`, the computer won't know how to break apart the command:

```bash
date-u
```

Some commands require arguments and won't work without them. `head` is a command that prints the first lines of a file, so it **requires** us to provide the file we want it to act on: 

```bash
head text.txt
```

Here "text.txt" is the **required** argument, and in this case it is also what's known as a **positional** argument. This is because we aren't identifying what it is with a preceding flag or anything. We are just listing it after the command, and the `head` command knows what to do with it. But this depends on how the command was written. Sometimes you need to specify the input file to a command (e.g. some commands will use the `-i` flag, but it's often other things as well).

If we ran `head` with no file to act on, it would get stuck. We know the terminal is still doing something (or trying to in this case) because our "prompt" hasn't returned. You can cancel an operation by pressing the "control" key and the "c" key simultaneously (`ctrl + c`). 

There are also optional arguments for the `head` command. The default for `head` is to print the first 10 lines of a file. We can change that by specifying the `-n` flag, followed by how many lines we want:

```bash
head -n 5 text.txt
```

Note that when we provided the `-u` flag to the `date` command (to get the command to print UTC time instead of local), we didn't need to provide any arguments to that particular flag (it's just on/off). But with the `-n` flag to `head`, we are specifying the number of lines we want to print out, so we have to provide a number in this case.

> This is the framework for how all things work at the command line! Multiple commands can be strung together, and some commands can have many options, inputs, and outputs and can grow to be quite long, but this general framework is underlying it all. **Becoming familiar with the baseline rules is important, memorizing particular commands and options is not!**

<br>

---
<br>

# The Unix file-system structure

Your computer stores file locations in a hierarchical structure. You are likely already used to navigating through this stucture by clicking on various folders (aka directories) in a Windows Explorer window or a Mac Finder window. Just like you need to select the appropriate files in the appropriate locations there, you need to do the same when working at the command line. What these means in practice is that each file and directory has its own "address", and that address is called it's "**path**". 

Here is an image of an example file-system structure:

<center><a href="https://raw.githubusercontent.com/AstrobioMike/AstrobioMike.github.io/master/images/file_system_structure.png"><img src="https://raw.githubusercontent.com/AstrobioMike/AstrobioMike.github.io/master/images/file_system_structure.png" width="500" height="551"></a></center>


<br>
There are two special locations in all Unix-based systems: the "**root**" location and the "**home**" location. "Root" is where the address system of the computer starts; "home" is where the current user's location starts.

We tell the command line where files and directories are located by providing their address, their "path". If we use the `pwd` command, we can find out what the path is for the directory we are sitting in. Whatever directory we are currently sitting in is called the "current working directory". And if we use the `ls` command, we can see what directories and files are in the current directory we are sitting in.

```
pwd
ls
``` 

## Absolute vs relative path
There are two ways to specify the path of the file we want to do something to: the absolute path and the relative path. 

* An **absolute path** is an address that starts from an explicitly specified location: either the "root" `/` or the "home" `~/` location. (When using the "root" as the start of the absolute path, it is referred to as the "full path".)
* A **relative path** is an address that starts from wherever you are currently sitting.

For example, let's look again at the `head` command we ran above:

```bash
head text.txt
```

What we are actually doing here is using a **relative path** to specify where the "text.txt" file is located. This is because the command line automatically looks in the current working directory for a file or directory if you don't specify anything else about it's location. 

We can also run the same command on the same file using the **absolute path**. Here is doing so starting from the "home" `~/` location. 

```bash
head ~/unix_intro/text.txt
```

> **Quick Practice**  
> Use `pwd` again to see the full path of your current working directory, and then try running the `head` command on the "text.txt" file by specifing the full path.


It is important to be comfortable thinking about where you are in your computer when working at the command line. One of the most common errors/easiest mistakes to make is trying to do something to a file that isn't where you think it is. Let's run `head` on the "text.txt" file again, and then let's try it on another file: "yet\_another\_text\_file.txt":

```bash
head text.txt
head yet_another_text_file.txt
```

Here the `head` command works fine on "text.txt", but we get an error message when we call it on "yet\_another\_text\_file.txt" telling us no such file or directory. If we run the `ls` command to list the contents of the current working directory, we can see the computer is absolutely right – spoiler alert: it usually is – and there is no file here named "yet\_another\_text\_file.txt". 

The `ls` command by default operates on the current working directory if we don't specify any location, but we can tell it to list the contents of a specific directory by providing it as a positional argument: 

```
ls another_directory
```

We can see the file we were looking for is located in the "subdirectory" called "another\_directory". Here is how we can run `head` on "yet\_another\_text\_file.txt" by specifying the **relative path**:

```bash
head another_directory/yet_another_text_file.txt
```

If we had been using **tab-completion**, we would not have made that mistake.

### BONUS ROUND: Tab-completion is your friend!
Tab-completion is a huge time-saver, but even more importantly it is a perpetual sanity-check that helps prevent mistakes. 

If we are trying to point to a file or directory that's in our current working directory, we can begin typing the file or directory name and then press `tab` to complete it. If there is only one possible way to finish what we've started typing, it will complete it for us. If there is more than one possible way to finish what we've started typing, it will complete as far as it can, and then hitting `tab` twice will show the possible options. And most importantly, if tab-complete does not do either of those things, then we are either confused about where we are, or we're confused about where the file or directory is that we're trying to do something to – this is invaluable.

> **Quick Practice**  
> Try out tab-complete! Run `ls` first to see what's in our current working directory again. Then type `head te` and then press `tab`. Since "text.txt" is the only possibility, that finishes for us and we can now press `enter` to run our command. Now try `head ano` and press `tab`. This should finish up to "another_" but then stop. This is because there isn't just one unique way to finish what we've provided. If you quickly press `tab` twice, it will show the possibilities. If we add a `t`, so `head another_t` and press `tab` again, now it will complete up to the extension. Add another `t` and press `tab` and it will finish completely for us. 

<center><b>Use tab-completion whenever you can!!</b></center>
<br>

## Moving around
We can also move into the directory containing the file we want to work with by using the `cd` command (**c**hange **d**irectories). The `cd` command takes a positional argument that is the path (address) of the directory you want to change into. This can be a relative path or an absolute path. So here we'll use the relative path of the subdirectory, "another_directory", to change into it:

```
cd another_directory/
pwd
ls
head yet_another_text_file.txt
```

Great. But now how do we get back 'up' to the directory above us? One way would be to provide an absolute path, like `cd ~/unix_intro/`, but there is also a handy shortcut. `../` is a relative path that specifies "up" one level – one directory – from wherever we currently are. So we can provide that as the positional argument to `cd` to get back to where we started, and then double check with `pwd` to show where we are and `ls` to list what's here:

```
cd ../
pwd
ls
```

> **Note on `cd`**  
> If you run the `cd` command with no arguments, it will use your home `~/` location as the default. So entering `cd` by itself will function the same as `cd ~/`. 

Moving around the computer like this may feel a bit cumbersome at first, but after spending a little time with it and getting used to tab-completion you'll soon find yourself slightly frustrated when you have to scroll through a bunch of files and click on something by eye with a mouse or trackpad in a Finder window 🙂

> **Quick Practice**  
Try to get used to regularly thinking about "where" you are in the computer when working at the command line. Let's take a quick trip. First run `pwd` so we remember where we are. Then let's change directories into our computer's root directory `cd /` and find our way back using tab-completion.

<br>

---
<br>

<h4><i>Terms presented in the previous section:</i></h4>

| Term     | What it is          |
|:----------:|------------------|
| **`path`** | the address system the computer uses |
| **`root`** | where the address system of the computer starts, **`/`** |
| **`home`** | where the current user's location starts, **`~/`**|
| **`absolute path`** | an address that starts from a specified location, i.e. root, or home |
| **`relative path`** | an address that starts from wherever you are sitting |
| **`tab-completion`** | our best friend |


<h4><i>Commands presented in the previous section:</i></h4>

|Command     |Function          |
|:----------:|------------------|
|**`pwd`**       |tells you where you are in the computer (**p**rint **w**orking **d**irectory)|
|**`ls`**        |lists contents of a directory (**l**i**s**t)|
|**`cd`**| **c**hange **d**irectories |


<h4><i>Special characters presented in the previous section:</i></h4>

|Characters     | Meaning          |
|:----------:|------------------|
| **`/`** | the computer's root location |
| **`~/`** | the user's home location |
| **`../`** |specifies a directory one level "above" the current working directory|

<br>

---
<br>

# Working with plain-text files and directories
The most common command line tools are mostly only useful for operating on what are known as **plain-text files** – also referred to as "flat files". There are a few definitions you can check out at the [wiki](https://en.wikipedia.org/wiki/Plain_text){:target="_blank"} if you'd like, but a good-enough working definition of what a plain-text file is might be something like: a text file that doesn't contain any special formatting characters or information, and that can be properly viewed and edited with any standard text editor. 

Common types of flat-text files are those ending with extensions like ".txt", ".tsv" for **t**ab-**s**eparated **v**alues, or ".csv" for **c**omma **s**eparated **v**alues. Some examples of common file types that are *not* flat-text files would be ".docx", ".pdf", or ".xlsx". This is because those types contain special types of compression and formatting information that are only interpretable by the programs that work with them.

> **Note on file extensions**  
> File extensions do not actually do anything to the file format. They are *mostly* there just for our convenience/organization – "mostly" because some programs require a specific extension to be present for it to interact with a file. 
> 
> The command `file` will tell you what type of file something is. Run `file excel_file.xlsx` and then `file another_text_file.xlsx`. The "excel\_file.xlsx" is actually an Excel file and has all kinds of special formatting for Excel that only makes sense to Excel. But "another\_text\_file.xlsx" is just a plain-text file that happens to have the extention ".xlsx". Try running `head` on each of these files.  

<br>
## Ways to probe plain-text files
We've already used a very common tool for peeking at files, the `head` command. There is also `tail`, which prints the last 10 lines of a file by default:

```
head text.txt
tail text.txt
```

Commands like `head` and `tail` are useful when you are working at the command line and you want to get an idea about the structure of a file. This is especially helpful if a file is particularly large, as `head` will just print the first ten lines and stop. This means it will be just as instantaneous whether the file is 10KB or 10GB. 

Another useful command for just viewing a file is `less`. This opens a searchable read-only program that allows you to scroll through the document: 

```bash
less text.txt
```

To exit the `less` program you need to press the "q" key. 

The `wc` command is useful for counting how many lines, words, and characters there are in a file: 

```bash
wc text.txt
```

Adding the optional flag `-l` will print just how many lines are in a file: 

```bash
wc -l text.txt
```

<br>
## Ways to manipulate files and directories

<div class="warning">
<center><h2>WARNING!</h2></center>
<b>Using commands that do things like create, copy, and move files at the command line will overwrite files if they have the same name. And using commands that delete things will do so permanently. Use caution while getting used to things – and then forever after</b> 🙂
</div>


The commands `cp` and `mv` (**c**o**p**y and **m**o**v**e) have the same basic structure. They both require two positional arguments – the first is the file you want to act on, and the second is where you want it to go (this includes the name you want to give it). 

To see how this works, let's make a copy of the "text.txt" file:

```
ls
cp text.txt text_copy.txt
ls
```

Remember we are actually providing a *relative path* when we provide the file names here. So to make make a copy of the "text.txt" file and put it somewhere else, like in our subdirectory "another\_directory", we would change the second positional argument:

```
ls another_directory/
cp text.txt another_directory/text_copy.txt
ls another_directory/
```

If we want to copy something from somewhere else to our current working directory, we can use another special character, a period – `.`. The period specifies the current working directory – just like `../` specifies one directory above us.

```
ls
cp another_directory/yet_another_text_file.txt .
ls
```

And now we have a copy of that file in our current working directory. 

Notice that in the first `cp` example we provided a new name in the target location, ("text_copy.txt"), but here when we used `.` to specify copying to the current working directory it simply copied the file with the same name as the original. This would be a time when an accidental overwrite could occur. 

The `mv` command is used to move files **and** to rename them if wanted. Remember that the path of a file actually includes its name. Here we will move a file from our current working directory into our subdirectory. At the moment our current working directory and our subdirectory contain these files:  

```
ls
ls another_directory/
```

Let's move "another\_text\_file.xlsx" into the subdirectory:

```bash
mv another_text_file.xlsx another_directory/
ls
ls another_directory/
```

Notice that we didn't provide a file name for the second positional argument of the `mv` command (where we were sending it). Just like when we used `cp` to copy a file to our current working directory with `.` above, this keeps the name of the original file.  

Remember that "another\_text\_file.xlsx" was actually a text file and not a ".xlsx". Let's rename it so that it has a ".txt" extension:

```
ls another_directory/
mv another_directory/another_text_file.xlsx another_directory/another_text_file.txt
ls another_directory/
```

Notice here that we did this from a different directory using a relative path to specify the starting file and the ending file. 

Now we'll see an example of how easy it can be to accidentally overwrite a file:

```
head text.txt
tail text.txt
wc -l text.txt

head another_directory/yet_another_text_file.txt
wc -l another_directory/yet_another_text_file.txt
```

```
cp another_directory/yet_another_text_file.txt text.txt
```

```
head text.txt
tail text.txt
wc -l text.txt

head another_directory/yet_another_text_file.txt
wc -l another_directory/yet_another_text_file.txt
```

And our original "text.txt" file is gone forever 😬

To delete files (intentionally) there is the `rm` command (**r**e**m**ove). This requires at least one argument specifying the file you want to delete. But again, caution is warranted. There will be no confirmation or retrieval from a waste bin afterwards.

```
ls
rm text.txt
ls
```

You can make a new directory with the command `mkdir`:

```bash
ls
mkdir our_new_directory
ls
```

And similarly, directories can be deleted with `rmdir`:

```bash
rmdir our_new_directory/
ls
```

Things are a little more forgiving when trying to delete a directory. If the directory is not empty, `rmdir` will give you an error. 

```bash
rmdir another_directory/
```

<br>
## Making and editing plain text files
It is often very useful to be able to generate new plain-text files quickly at the command line, or make some changes to an existing one. One way to do this is using a text editor that operates on the command line. Here we're going to look at a program called `nano`.

When we run the command `nano` it will open a text editor in our terminal window. If we give it a file name as a positional argument, it will open that file if it exists, or it will create it if it doesn't. Here we'll make a new file:

```bash
nano sample_names.txt
```

Now we can type as usual. Afterwards, to save the file and exit we need to use some of the keyboard shortcuts listed on the bottom. "Write Out" will save our file, and the `^O` represents hitting `ctrl + o` (doesn't need to be a capital "O"). This will ask you to either enter or confirm the file name, we can just press `enter`. Then to exit we can press `ctrl + x`. And now our new file is in our current working directory:

```
ls
head sample_names.txt
```

<br>

---
<br>

<h4><i>Commands presented in the previous section:</i></h4>

|Command     |Function          |
|:----------:|------------------|
|`head`      |prints the first few lines of a file|
|`tail`      |prints the last few lines of a file|
|`less`      |allows you to browse a file (exit with "q" key)|
|`wc`       |count lines, words, and characters in a file|
|`cp`      |copy a file or directory (use with caution)|
|`mv`      |mv a file or directory (use with caution)|
|`rm`      |delete a file or directory (use with caution)|
|`mkdir`       |create a directory|
|`rmdir`     |delete an empty directory|
|`nano`     |create and edit plain text files at the command line|

<br>

---
<br>

# Pipes and redirectors

Now we're going take our first look at what makes the Unix command-line environment so powerful: pipes and redirectors! 

A pipe `|` is used to connect multiple commands. It takes in the output from the previous command and pipes it into the input of the following command. Let's look at an example. 

`ls` as we've seen lists the files and directories in our current working directory:

```
ls
```

If we pipe `|` that command into `wc -l`, instead of printing the output from `ls`, it will go into `wc -l` which will print out items there are:

```
ls | wc -l
```

Another important operator is the greater than sign, `>`. This tells the command line to "redirect" the output to a file rather than just printing it to the screen. For an example of this we will write the output of `ls` to a new file called "directory_contents.txt":

```
ls > directory_contents.txt
```

Notice that nothing printed to the screen this time. If we run `ls` we'll see the file we just created is there. 

```
ls
head directory_contents.txt
```

It's important to remember that the `>` redirector will overwrite the file you are pointing to if it already exists. If we use two of them instead, `>>`, this will append whatever you're sending to the target file:

```
ls >> directory_contents.txt
head directory_contents.txt
```

Okay, so far that isn't all that impressive. But this basic, "modular" structure really is what makes the Unix command-line rock. We'll get to more practical uses/examples later with [six glorious commands](/bash/six_commands) and [why is this all worth it?](/bash/why).

<br>

---
<br>

<h4><i>Special characters presented in the previous section:</i></h4>

|Characters     |Function          |
|:----------:|------------------|
|`|`      | a "pipe" allows stringing together multiple commands |
|`>`      |sends output to a file (**overwrites** target file)|
|`>>`      |sends output to a file (appends to target file)|

<br>

---
<br>

# Wildcards

Wildcards in *bash* are also an essential component to understand if you want to maximize your capabilities at the command-line. Basically a wildcard allows you to specify multiple items at once based on how you use it to identify your targets. The `*` and `?` are the most often used, and we'll get a glimpse of how they work with the `ls` command again.

As we've seen so far, `ls` lists the contents of the current working directory. By default, `ls` assumes you want everything, so when providing no further arguments it prints everything. But we can specify what we want it to operate on by giving it a positional argument. To make this example slightly more meaningful, we'll also provide the `-l` flag to the `ls` command, which will give us some more information about the file. Here we'll specify "yet_another_text_file.txt":

<center><img src="{{ site.url }}/images/ls_l_one_item.png"></center>

<br>
Now that we get the format when we specify a single file, let's look at it when specifying multiple. Say we want to see all files that start with the word "text". One way to do this is to list out each file name one after the other:

<center><img src="{{ site.url }}/images/ls_ex_test.png"></center>

<br>
But we can also do this with the `*` wildcard much more easily:

```
ls -l text*
```

<center><img src="{{ site.url }}/images/ls_ex_test_star.png"></center>

<br>
Here, we are giving what's unique about what we are looking for (the "text" beginning), and then with the `*` saying accept anything after that. 

Let's look at this another way where we want to see all files that end with the extension ".txt":

```
ls -l *.txt
```

<center><img src="{{ site.url }}/images/ls_ex_star_txt.png"></center>

<br>
The `?` wildcard can represent any character that appears only one time. To demonstrate this I'm going to make a couple of blank files with the `touch` command. This command basically just creates a blank file if one doesn't exist, or if it does exist, it updates the time stamp of when it was last accessed (check out the [wiki](https://en.wikipedia.org/wiki/Touch_(Unix)) if interested).

<center><img src="{{ site.url }}/images/ls_question_mark_ex.png"></center>

<br>
In the last command there we can see it acted on files that had only a single character where the `?` wildcard was placed. Here is what the output is like when we use the `*` in the same position:

<center><img src="{{ site.url }}/images/ls_star_mark_ex.png"></center>

<br>

---
<br>

<h4><i>Special characters presented in the previous section:</i></h4>

|Character     |Function          |
|:----------:|------------------|
|`*`      |an asterisk represents any character appearing any number of times|
|`?`      |a question mark represents any character that appears just once|

<br>

---
---
<br>
<h1>Congrats on getting through the basics!</h1>
Some of these things may seem trivial as used in these examples here, and I know that can make the effort required to learn to use them harder to come by, but these basics really are the foundation to doing everything at the command line. Understanding the general rules of commands, their arguments, and the required syntax will make everything you try to do easier.  

There are a lot of base commands in *bash*, and a dizzying number of optional arguments for most of them. Again, google is our friend. If you end up working in the terminal window enough, you will remember some things, but also you will often do a quick search to remember what the flag is for a specific argument, or how exactly a specific command works. This really isn't about memorization overall.

<a id="bottom"></a>

Remember the set of commands you ran at the beginning to get the files needed for this walkthrough?

```
cd ~
curl -O https://AstrobioMike.github.io/tutorial_files/bash_basics_temp.tar.gz
tar -xvf bash_basics_temp.tar.gz
rm bash_basics_temp.tar.gz
cd bash_basics_temp
```

I promised you'd have a better idea of what's going on in here by the end, so let's break it down.

We start by changing directories to our "home" location, and we do this with the special character `~`. Then we use two commands you aren't familiar with yet, `curl` and `tar`. We won't get into these here other than to say `curl` allows you to download things from the internet from the command line, and `tar` is a tool for packing and unpacking files and directories. So with the `curl` command we download the files in a compressed format, and then with `tar` and some options we expand them into a directory called "bash_basics_temp". Then all we do is delete the uncompressed files ("bash_basics_temp.tar.gz"), and change directories into "bash_basics_temp".  

As you can see here, while the commands change, the general structure of how to operate in the command line stays the same. This is part of why the learning curve is steep at the start but it gets better. Once you become comfortable with the general framework, everything is gets easier.  

Now that you're comfortable with the basics, head on over to [6 commands worth getting to know](/bash/six_commands) for a walkthrough of some stellar commands I wouldn't want to live without.



