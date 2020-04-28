---
layout: post
title: Syntax highlighting in Jekyll
description: Highlighting code using markdown in Jekyll
author: bitsmonkey
tags: code highlight, syntax, jekyll
---

Here is a way to highlight your code on Jekyll site using the same Markdown syntax that Github supports. I have been using [github markdown](https://guides.github.com/features/mastering-markdown/) day-in & day-out mostly with [github gists](http://gist.github.com/).

![syntax-hghlight](/img/syntax-highlight.jpg)

To use the same markdown syntax to highlight the code syntax in your Jekyll site. First, install the rouge gem.

```shell
    gem install rouge
```

Now in your `_config.yml file`

```yml
    markdown: kramdown
    highlighter: rouge
    kramdown:
      input: GFM
```
The above setting sets [kramdown](https://kramdown.gettalong.org/) as my markdown and for syntax highlighting using the `rouge` the gem you just installed. Also, GFM says to use the [Github Flavoured Markdown](https://guides.github.com/features/mastering-markdown/).


Now download the default CSS for the code styling from [Github](https://raw.githubusercontent.com/mojombo/tpw/master/css/syntax.css) and copy to the CSS directory of your site and add the reference in `_layout/default.html`.

```html
    <link rel="stylesheet" href="/css/syntax.css" type="text/css" />
```

Done!

*Image by [Lorenzo Cafaro](https://pixabay.com/users/3844328-3844328/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1857236) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1857236)*