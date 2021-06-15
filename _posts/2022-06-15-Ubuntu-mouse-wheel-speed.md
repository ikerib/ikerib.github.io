---
layout: single
title: Ubuntu Mate - Mouse wheel speed
excerpt: "Ubuntu Mate - Customize mouse speed"
date: 2021-06-15
classes: wide
header:
  teaser: /assets/images/mouse-wheel.jpg
  teaser_home_page: true
categories:
  - Ubuntu
tags:
  - Ubuntu
  - Linux
---

Install imwheel first:

```bash
sudo apt install imewheel
```

Download this [script]({{ site.baseurl }}{% link /assets/files/mousewheel.sh %}) thanks to [@victoor](https://dev.to/victoor)


Give permissions and run:

```bash
sudo chmod +x mousewheel.sh && sudo sh mousewheel.sh
```

Set your mousew wheel speed (7 for example) and click apply.

If previous and next mouse buttons doesn't work correctly add `imwheel -b "4 5"`

```bash
sudo imwheel -b "4 5"
```