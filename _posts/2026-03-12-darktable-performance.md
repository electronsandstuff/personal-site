---
layout: post
title: Speeding Up Darktable Culling
date: 2026-03-12 11:12:00-0400
description: Pregenerating Darktable thumbnail cache to avoid interruptions durring culling.
tags:
categories:
related_posts: false
---

I use [Darktable](https://www.darktable.org/) to organize and process my photos while avoiding the Adobe tax. While it is a great piece of software, I frequently run into an issue where I outrun the performance of my laptop while culling photos. That is, when going through photos straight out of my camera, I flip through them faster than my computer is able to render thumbnails. This is a problem because it interrupts my flow in filtering through the images and greatly slows me down.

<p align="center">
<img src="/assets/img/tech_notes/2026-03-12-darktable/image.png" alt="Darktable working screen" width="75%" style="max-width: 400px">
</p>

One of the advantages of open source software is being able to see the code and after a quick tour through [Darktable's](https://github.com/darktable-org/darktable), I discovered that the viewer is likely looking for my image in the mipmap thumbnail cache, discovering it isn't there for my display's size, then generating it on the fly.

Fortunately, Darktable ships with a CLI tool to generate the cache allowing you to import raw images from your camera, generate the thumbnail cache all at once (this will take a second), then cull your images with no interruptions. With my Flatpack installation, the command looked like the following, where `--min-imgid` is the image ID of the first image imported (found in the information panel when the first image is selected). Note: this must be run with Darktable closed to avoid database lock issues.

```
flatpak run --command=darktable-generate-cache org.darktable.Darktable --min-mip 6 --max-mip 8 --min-imgid 17697
```

After running overnight, I am able to flip through my ~2k image set from the weekend without issue. I would love to see this built into the GUI to operate on selected images or to run during import.