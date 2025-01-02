---
title: Emacs
---

# Emacs
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Convert Text File Format

Open a file with emacs
```sh
$ emacs <file>
```

Reopen the file with a correct file format if necessary
```sh
C-x C-m r dos
# or
C-x C-m r unix
```

Convert the file format
```sh
C-x <RET> f unix
# or
C-x <RET> f dos
```
