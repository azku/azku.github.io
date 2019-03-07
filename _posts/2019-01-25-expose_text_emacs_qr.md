---
layout: post
title:  "Expose text from Emacs to another device via QR"
author: azku
tags: [emacs, qr]
categories: []
---

Expose text from Emacs to another device via QR I am one of those who uses Emacs extensively. Particularly when it comes to keeping myself organised or simply for annotating and  storing, I use org-mode.

This is not a post about telling how awesome org-mode and Emacs are, for that there are many posts, but for exposing a flaw I had with my setup and explaining a workaround.

I have a couple of desktops in different locations and connected to the internet. In order to keep my agenda and note files synchronised  I keep them encrypted with GNUPG and then post them to a public Git repository. This solves the problem of sharing the data between  different locations as long as I am on those computers.

The problem is that when I am not, I loose track of the things on my calendar. Despite not being at the desktop, I do still have  my phone on and I would like to somehow have the data accesible. Connecting a phone to the PC via USB and start copying file s it's too much hasle.

So I thought: "Wouldn't it be great if I could create a QR code out of the org-agenda view and quickly read it with the phone?".

I did a quick search on the Web and found that there already are some Emacs packages that do that but none (as far as I am aware) are  in Melpa.

Determined to have a solution without much tweaking of my system and having qrencode and feh Linux programs already installed on it, I set to search how to launch them from Emacs. It turns out it's pretty easy and here is the code:

{% highlight elisp %}
(defun region-to-qr()
 (interactive)
 (shell-command
  (format
   "qrencode -o - %s | feh -"
   (shell-quote-argument
    (buffer-substring (region-beginning)
                      (region-end))))))
{% endhighlight %}

I have found the tecnique it's usefull for a miriad of things.
