---
title: Switch blog theme to FLUID
author: Ryan LI
toc: true
declare: true
date: 2022-04-30 16:05:02
tags:
  - hexo
  - blog
---

> The former "yilia" theme starts to be buggy since it was no longer maintained. I switch to this "FUILD" theme, for now, hopefully it will stand.

<!-- more -->

### Reference docs

[Docs](https://hexo.fluid-dev.com/docs/en/), [Preview](https://hexo.fluid-dev.com/posts/fluid-hitokoto/), [Github repo](https://github.com/fluid-dev/hexo-theme-fluid)

### Switch theme to Fluid

```shell
npm install --save hexo-theme-fluid
```

Edit `_config.yml` in the blog root directory as follows:

```yaml
theme: fluid
```

Create the about page manually:

```bash
hexo new page about
```

Then edit `/source/about/index.md` and add `layout` attribute.

Execute the command in your blog directory：

```bash
npm update --save hexo-theme-fluid
```

### Customise

create `_config.fluid.yml` in the blog directory and copy the content of [_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)

