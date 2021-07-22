---
title: "Asciidoctor PDF: Override the document theme to make tables the full page width"
date: 2021-05-16
draft: false
tags: ['Asciidoc','Asciidoctor PDF']
---

### TL;DR

If you have a Asciidocotor-Pdf theme that indents text, but you want some of the tables to span the full page width, margin to margin, skip to [here](#solution)

### The Problem

I started toying around with Asciidoc around 14 months ago, and I have been impressed so far. I got sick of Microsoft Word, and I wanted to find a method that worked well with source control. So far, all has been good. Like all tools, it has pros and cons, and this blog is about a con that turned into a pro.

Firstly, Asciidoc-Pdf, the tool for converting Asciidoc directly to PDF files, has a nice theming setup. You prepare your theme in a YAML file, and then you forget about it! All you need to concentrate on now is your text; no more pesky Word Styles! The downside is that sometimes you can't quite get the output you need.

So I have a theme that indents all body text about an inch, including tables, which is fine most of the time, but I might have about one or two tables that should be the full width of the document, margin to margin. There is no way to override the theme for tables. So I was stuck with indented tables.

It turns out a guy called Alexandre had posted a GitHub issue Support [.pagewide] role for tables and images · Issue #1415 · asciidoctor/asciidoctor-pdf (github.com), for this exact problem, he said he extended the convertor. Great! Now how do I do that?

I knew from reading some of the documentation that Asciidoctor had [extensions](https://docs.asciidoctor.org/asciidoctor/latest/extensions/), so I set about learning all about them. I got a bit stuck and asked the community for some pointers, luckily Dan Allen who the AsciiDoctor project lead, he pointed out that I was going down the wrong route and I should extend the convertor. Umm... that's what I was trying to do. I'm saying Dan wasn't helpful, I'm super grateful for his work on Asciidoc and has responded promptly to my calls for help before. But I just didn't know what that meant.

### The Solution {#solution}

A little bit more googling and I figured out both Alexandre and Dan were referring to the Convertor class in Asciidoctor-Pdf, which does some of the heavy lifting. Fortunately, there is a super simple way to extend a class in Ruby (I had never written any Ruby code up to this point) and include it in the Asciidoctor build process.

So I wrapped Alexandre's code and appended it to the Convertor class, shown in the below gist:

{{< gist auslavs d7c0ebba3ceca093ccda678808d3740e >}}

Then when creating my pdf file I just needed to reference the ruby file like below:

```
asciidoctor-pdf -r ./page-wide.rb -a pdf-theme=./theme.yml main.adoc
```

Boom! Worked first go, and then the con of not being to override the theme became a pro thanks to Asciidocotors extensibility!

Thanks Alexandre, Dan and the Asciidoc community!