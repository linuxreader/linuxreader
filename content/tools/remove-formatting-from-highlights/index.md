---
title: 'How to Remove Formatting From Ebook Highlights With Vim'
summary: 'Removing extra stuff from Bookfusion and Foliate exported highlights'
date: 2026-02-04
---
## Bookfusion
Here are my highlights pulled up in Vim:

![](formattingbookfusion.png)

As you can see, Bookfusion gives you a lot of extra information when you export highlights. First, let's get rid of the lines that begin with `##`

Enter command mode in Vim by pressing `esc`. Then type `:g/^##/d` and press enter.

Much better. 


![](Pasted%20image%2020240414034407.png)

Now let's get rid of the color references:`
```bash
:g/^Color/d
```

![](Pasted%20image%2020240414034652.png)

To get rid of the timestamps, we must find a different commonality between the lines. In this case, each line ends with "UTC". Let's match that: 
```bash
:g/UTC$/d
```

Where `$` matches the end of the line.  

![](Pasted%20image%2020240414035013.png)

Now, I want to get rid of the `> ` on each line: 
`%s/> //g`

![](Pasted%20image%2020240414035218.png)

Almost there, you'll notice there are 6 empty lines in between each highlight. Let's shrink those down into one: 
```bash
:%s/\(\n\)\{3,}/\r\r/g
```


![](Pasted%20image%2020240414035716.png)

The command above matches newline character `n` 3 or more times and replaces them with two newline characters `/r/r`.

As we scroll down, I see a few weird artifacts from the book conversion to markdown.

![](Pasted%20image%2020240414040052.png)

Now, I want to get rid of any carrot brackets in the file. Let's use the substitute command again here:
```bash
%s/<//g
```

![](Pasted%20image%2020240414040338.png)

Depending on your book and formatting. You may have some other stuff to edit. 

## Foliate
Get rid of lines with color reference:
`:g/^**rgb/d`

get rid of the `> ` on each line: 
`%s/> //g`

Get rid of dashed lines:
`:g/^---/d`

Get rid of consecutive empty lines:
`:%s/\(\n\)\{3,}/\r\r/g`