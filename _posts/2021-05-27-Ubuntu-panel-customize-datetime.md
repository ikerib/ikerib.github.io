---
layout: single
title: Ubuntu Mate - Show and Customize datetime on tray
excerpt: "Ubuntu Mate - Show and Customize datetime on tray"
date: 2021-05-21
classes: wide
header:
  teaser: /assets/images/panel-datetime.jpg
  teaser_home_page: true
categories:
  - Ubuntu
tags:
  - Ubuntu
  - Linux
---

We need to use `dconf-editor` to customize our try icon panel. First we need to install `dconf-editor`:

```bash
sudo apt install dconf-editor
```

After that we need to navigate to `com -> canonical -> indicator -> datetime` and make some changes:

    - custom-time-format: %A Y%/%m/%d - %H:%M
    - show-date: enabled
    - show-day: enabled
    - show-year: enable
    - time-format: custom

This is a screenshot with my config:
![dcong-editor]({{ site.baseurl }}/images/dconf-datetime-panel.png)

        