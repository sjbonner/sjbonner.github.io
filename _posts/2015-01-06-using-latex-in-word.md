---
id: 114
title: Using LaTeX in Word
date: 2015-01-06T17:57:10+00:00
author: admin
layout: post
guid: http://simon.bonners.ca/bonner-lab/wpblog/?p=114
permalink: /?p=114
categories:
  - LaTeX
tags:
  - LaTeX
  - Word
---
I use LaTeX when writing my own papers. However, when collaborating with other authors, particularly those in biology, I have to used Microsoft Word. In the past I have converted LaTeX equations into images to embed into the Word document, but this is cumbersome and makes it difficult to edit the equations later.

I recently found Mr. Sparkle wonderful [LaTeX in Word](http://sourceforge.net/projects/latexinword/) project which makes this a lot easier! The heart of this is Word document that contains embedded macros which use a remote server to process LaTeX code into embedded images. To use the package you simply download and extract the tarball, save it to a new file, and start typing away. The Alt-L keystroke will open a new window that you can type LaTeX commands into, and pressing the convert button will turn these commands into images. The best thing is that the code stays embedded in the document. With the same keystroke you can reopen the command window and edited the existing code. Fantastic!