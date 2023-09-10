---
title: "Automate the Shit"
date: 2019-04-11T21:27:13+05:30
draft: false
toc: false
images:
tags:
  - RegEx
  - Bash
  - ReGex
---

In this inaugural blog post, I want to share with you the magic of automation - a concept that can transform your daily tasks from mundane to marvelous.

Automation is the art of streamlining processes, reducing manual labor, and saving precious time. Why waste hours on repetitive chores when you can automate them effortlessly?

{{< image src="/can-this-be.jpg" alt="can this be automated" size="400x450" align="center" >}}

---

# Story
`[UPDATE]: wallpaperplay.com is not functional anymore.`

One fateful day, I found myself on a quest for minimalist wallpapers to adorn my Elementary OS desktop. My search led me to the promising trove of wallpapers at wallpaperplay.com. However, there was a caveat—the tedious download process:

1. Click the Download button.
2. Wait for a 5-second countdown.
3. Click the generated download link.
4. Finally, right-click and save the image.

I asked myself, "Who has the patience for these four steps just to download a wallpaper?" Certainly not me.

It was then that I had a idea to automate this time-consuming process.

---

# Approach
My approach was straightforward:

1. Inspect the source code of the current page.
2. Extract the wallpaper links embedded within it.
3. Save all the links to a file.
4. Use the file as an argument for the wget command.

Now, all I needed to do was translate this concept into working code.

---

# Getting Started
Let's say we want to download over 84 wallpapers from this page:

https://wallpaperplay.com/board/minimalist-desktop-wallpapers

1. First, download the HTML page using the wget command:

```shell
wget https://wallpaperplay.com/board/minimalist-desktop-wallpapers
```
This command downloads and saves the HTML file with the path name "minimalist-desktop-wallpapers."

---

2. Next, extract the filename from the URL using Regular Expressions:

```shell
echo "$1" | grep -Eoi 'board/.*' | cut -d'/' -f2'
```
Let’s decode this RegEx:-

`echo "$1"` means print the URL as output.

`grep -Eoi 'board/.*'` means match everything that includes and comes after “board/” and using cut command we’ll further cut out board from output and rest will be our filename.

For ex:\
https://wallpaperplay.com/board/blue-car-wallpapers –> “blue-car-wallpapers”.

https://wallpaperplay.com/board/hipster-galaxy-wallpapers –> “hipster-galaxy-wallpapers”.

---
3. Now, we need to extract the links from the downloaded HTML page using Regular Expressions:

```shell
cat "minimalist-desktop-wallpapers" | grep -Eoi '<a[^>]+>'
```
This command extracts all HTML anchor tags containing links.

`grep -Eoi '<a[^>]+>'`

This is a Regular Expression which extracts the anchor tags `<a>`. It means “Match everything that starts with “<a” except “>” in between and upto “>”

E = Extended
o = Show only the matched string
i = Case insensitive search

---
4. Extract links from href attribute.

```shell
grep -Eoi '/.*jpg'
```
This Regular Expression will extract the value of href attribute. It means “Match everything that starts with “/” and contains anything afterwards and ends with “.jpg”

Till here we will get output like this- /walls/full/2/5/9/20629.jpg

which is a relative URL. We can’t really use this as an argument for wget command. We need to append “https://wallpaperplay.com” at the beginning of the URL to make it absolute URL.

---
5. Making relative URL into absolute URL.
```shell
sed 's/^/https:\/\/www.wallpaperplay.com/g'
```
This Regular Expression will add “https://www.wallpaperplay.com” at the beginning of URL.

sed is stream editor and is basically use for editing text.

“^” means at the beginning of each line.

After these operations, we have valid URLs. Redirect all links to a text file using the ">" operator, and then pass the text file as an argument to wget using the "-i" flag, specifying the destination folder with "-P."

---

# Bringing It All Together
```shell
filename=$(echo "$1" | grep -Eoi 'board/.*' | cut -d'/' -f2)

cat "$filename" | grep -Eoi '<a[^>]+>' | grep -Eoi '/.*jpg' | sed 's/^/https:\/\/www.wallpaperplay.com/g' > links.txt

wget $2 $3 -i links.txt -P wallpapers/

rm $filename

rm links.txt
```
---
GitHub: https://github.com/iamnihal/wallpaperplay

That’s all for this blog. See you soon with the new one. ;-)





