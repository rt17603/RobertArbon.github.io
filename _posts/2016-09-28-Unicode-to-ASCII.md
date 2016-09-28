---
layout: post
title:  "How to convert file to ASCII encoding"
category: linux
---

## Problem

While trying to load a text file (`toppar_water_ions_fixed.str`) using a module called [ParmEd](https://github.com/ParmEd/ParmEd) with Python 3 I was getting the following error:

    UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 1009: ordinal not in range(128)


## Solution

I checked the encoding of the file using `file`: 

    (py35) MD ➤ file toppar_water_ions_fixed.str

    toppar_water_ions_fixed.str: UTF-8 Unicode English text

So I attempted to convert it to ASCII using `iconv`: 

    iconv -f UTF-8 -t ASCII toppar_water_ions_fixed.str

but I got the following error: 

    iconv: toppar_water_ions_fixed.str:305:43: cannot convert

Luckily the offending character wassn't really necessary so deleted the line and attempted to convert again: 

    iconv -f UTF-8 -t ASCII toppar_water_ions_fixed_v2.str

and voila, it worked: 

    (py35) MD ➤ file toppar_water_ions_fixed_v2.str
    toppar_water_ions_fixed_v2.str: ASCII English text

