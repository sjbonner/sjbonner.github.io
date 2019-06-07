---
id: 143
title: Polymode for ESS
date: 2015-06-17T19:06:06+00:00
author: Simon Bonner
layout: revision
guid: http://simon.bonners.ca/bonner-lab/wpblog/?p=143
permalink: /?p=143
---
I wrote my first vignette for an R package the other day and was amazed at the ease of writing Rmarkdown. Of course, I am using emacs and was looking for a mode that would allow me to edit these files smoothly with syntax highlighting and the ability to send code chunks to R automatically. Et voila &#8212; [Polymode](https://github.com/vspinu/polymode). I followed the <a href="http://johnstantongeddes.org/open%20science/2014/03/26/Rmd-polymode.html" target="_blank">John Stanton-Geddes instructions</a> for enabling markdown-mode and polymode, but ran up against a few minor issues. The first was that copying and pasting the LISP code into my .emacs file introduced some errors &#8212; single quotes turned into backticks and doublequotes turned into smart quotes. In the end, I had to reformat these/type them in myself.

I also found that I needed to add the line:

`(add-to-list 'auto-mode-alist '("\\.Rmd" . poly-markdown+r-mode))`

to my .emacs file for polymode to start automatically when I opened a .Rmd file. This was mentioned in the <a href="https://github.com/vspinu/polymode#activation-of-polymodes" target="_blank">polymode instructions</a> which I should probably have read first. The full entries in my .emacs file for markdown-mode and polymode are:

`;;; Markdown mode<br />
(autoload 'markdown-mode "markdown-mode" "Major mode for editing Markdown files" t)<br />
(setq auto-mode-alist (cons '("\\.markdown" . markdown-mode) auto-mode-alist))<br />
(setq auto-mode-alist (cons '("\\.md" . markdown-mode) auto-mode-alist))<br />
(setq auto-mode-alist (cons '("\\.ronn?" . markdown-mode) auto-mode-alist))`

`;;; Polymode<br />
(setq load-path (append '("/home/sbonner/.emacs.d/polymode/" "/home/sbonner/.emacs.d/polymode/modes") load-path))`

`(require 'poly-R)<br />
(require 'poly-markdown)`  
`(add-to-list 'auto-mode-alist '("\\.Rmd" . poly-markdown+r-mode))`

Looking great!