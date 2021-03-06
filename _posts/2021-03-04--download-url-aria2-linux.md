---
title: 'Download url with aria2'
date: 2021-03-04
permalink: /posts/2021/03/aria2-linux/
tags:
- linux
- aria2
---

## Prerequisites

- aria2
- POSIX shells such as `bash`, `zsh`.. and CLI.

## Introduction

Today, I was given a task to download a list of a lot of URL store in the CSV file, and in this file, there are two columns called respectively `Name` and `URL` as shown below (Thanks to [online CSV tool](https://www.convertcsv.com/csv-to-markdown.htm)):

|Name         |URL                        |
|-------------|---------------------------|
|shakira.jpg  |https://www.biography.com/.image/t_share/MTE5NDg0MDU0NzExNDA0MDQ3/shakira-189151-1-402.jpg|
|eminem.jpg   |https://i.pinimg.com/474x/19/63/da/1963daa666a8030047e2a9f13beb6975.jpg|
|cristiano-ronaldo.jpg|https://files.thehandbook.com/uploads/2019/03/ronaldo.jpg|

My task requirement is, in each row, I have to download file from the given `URL` and save it with the corresponding name on the left side. This is such an interesting task, as you can see if you do it manually with a series of repeated actions include enter, copy and paste, you even can't estimate how long does it take to finish a long list of URLs, and thus the idea is using the automation tool in order to address the tedious one. Ok, let's get started.
    
## Tooling

The first idea that came to my mind was using one of the programming languages such as `Java` or `Bash` but it seems to take more time for a clumsy solution. 

```shell
BufferedReader reader = new BufferedReader(new FileReader("urls.csv"));
List<String> data = reader.lines().collect(Collectors.toList());

data.forEach(i -> {
    String[] line = i.split(",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)", -1);
    String name = line[0];
    String urlStr = line[1];
    try(InputStream in = new URL(urlStr).openStream()){
        Files.copy(in, Paths.get(name), StandardCopyOption.REPLACE_EXISTING);
    } catch (IOException e) {
        e.printStackTrace();
    }
});
```

So after spent a couple of hours tirelessly googling, I eventually found out a tool that can address my problem by `one-liner command`. This is [aria2c](https://github.com/aria2/aria2), the ultra-fast download utility.

## Solution

`aria2c` supports so-called `option lines` feature in input files. From man page
> -i, --input-file=
> 
> Downloads the URIs listed in FILE. You can specify multiple sources for a single entity by putting multiple URIs on a single line separated by the TAB character. Additionally, **options can be specified after each URI line**. Option lines must start with one or more white space characters (SPACE or TAB) and must only contain one option per line.

Later on

> These options have exactly same meaning of the ones in the command-line options, **but it just applies to the URIs it belongs to**. Please note that for options in input file -- prefix must be stripped.

Later on

>  -o, --out=<FILE>
> 
>The  file  name  of  the downloaded file.  It is always relative to the directory given in --dir option.  When the --force-sequential option is used, this option is ignored.

If you still don't have any idea about which solution would be, let me summarize all it up into step by step

- Firstly, in order to use `option lines` feature, we need convert our `CSV` file to the new format.

    |https://www.biography.com/.image/t_share/MTE5NDg0MDU0NzExNDA0MDQ3/shakira-189151-1-402.jpg|                                                                                        |
    |------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
    |   out=shakira.jpg                                                                           |                                                                                              |
    |https://i.pinimg.com/474x/19/63/da/1963daa666a8030047e2a9f13beb6975.jpg                   |                                                                                              |
    |   out=eminem.jpg                                                                            |                                                                                              |
    |https://files.thehandbook.com/uploads/2019/03/ronaldo.jpg                                 |                                                                                              |
    |out=cristiano-ronaldo.jpg                                                                 |         
  
    - Luckily, there are linux command called `sed` aka **Stream Editor** can convert the old one to the right new format
    
        ```shell
        sed [options] commands [file-to-edit]
        ```
        ```shell
        sed -E 's/([^,]*),(.*)/\2\n  out=\1/' file.csv
        ```
  
    - Let’s take a moment to examine this command in detail:
        - `-E` [POSIX Extended Regular Expression](https://www.regular-expressions.info/posix.html)
        - `'s/([^,]*),(.*)/\2\n  out=\1/'` indicate that we will substitute text string in each line in the CSV file to another using `s` command (as in substitute) with the following form: `s/SEARCH_REGEX/REPLACEMENT`. In our example, `SEACH_REGEX` is `([^,]*),(.*)` contains two parts delimited by comma, so-called respectively the first captured group (matched name's value) `([^,]*)` and the second captured group (matched `URL`'s value) `(.*)`, `REPLACEMENT` is `\2\n  out=\1`, we just rearranged these captured group in appropriate positions.


- Next step, we use `aria2c` finished our task
    ```shell
    sed -E 's/([^,]*),(.*)/\2\n  out=\1.jpg/' file.csv | aria2c -i -
    ```
    - The above command indicate that the output of command (aka `STDOUT`) `sed -E 's/([^,]*),(.*)/\2\n  out=\1.jpg/' file.csv` will be use as an input (aka `STDINT`) of `aria2c -i` command.
        > Every thing in Linux is a file, therefore `STDIN`, `STDOUT` and `STDERR` is every important to using CLI effectively. 
  

---------------------

### For further reading:

- https://gist.github.com/GAS85/79849bfd09613067a2ac0c1a711120a6
-----------------------------------

Thank you :blush:.