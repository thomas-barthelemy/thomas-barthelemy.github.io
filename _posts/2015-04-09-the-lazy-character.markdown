---
layout:     post-no-pic
title:      "The lazy character"
subtitle:   "The story of a character that would always show up at the end"
date:       2015-04-09 15:04:00
author:     "Thomas Barthelemy"
---

A minor bug report came up to me today:
A character is misplaced in one of an app label introduced with the latest feature in development for one of our .NET / WPF app.

The screen-shot was pretty clear: in the Chinese local of the app, a Chinese character was at the end instead of the beginning of the string.
My (simplified) Chinese reading is fairly limited but we do have a translating process that usually avoids this kind of mistake, so yes I was a bit surprised.

I quickly ran the app in the appropriate local while checking the resource at the same time:

- Bug reproduced, the character is indeed misplaced
- The resource string is... correct?

Whatever I would input in this resources, this specific character would always show at the end.
While double-clicking to select the string I realized it would leave that character out of the selection, it was definitely special.

The character in question (sheng: 生) is fairly common, more than other characters of the string so it wasn't an issue of "rare character" that I would have copied.

At this point I already realized it was an encoding issue, the first character having a different encoding(read not UTF-8, Big5 and GB2312).
Out of curiosity I decided to throw both the original and the same character I wrote myself in Google translate:
Although it was giving me the same pinyin (The official phonetic system for transcribing Mandarin) the translation were different.

I've dealt countless time with encoding issue in China, the mean reason being that government related software are forced to use the official encoding (GB2312) and writing CSV files for excel is usually a great pain as Chinese local would only read GB2312 CSV file while other local would only read UTF8 CSV file.

I will leave here the faulty and corrected strings:
    
    ⽣成带水印预览
    生成袋水印预览

And the google translate output:

    DRAMATIC a watermarked preview
    Generate bag watermark preview
