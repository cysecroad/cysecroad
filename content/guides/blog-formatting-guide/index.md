---
title: "Blog Formatting Guide"
date: 2024-11-28T16:32:40+01:00
draft: true
ShowToc: true

section: "guides"
url: "/guides/blog-formatting-guide"

tags: [markdown, formatting, blog-writing, documentation]
categories: [guides, technical, web-development]
keywords: [markdown tutorial, blog formatting, hugo shortcodes, technical documentation, writing guide, markdown basics, blog styling, content creation, technical writing, blogging tips, markdown syntax, hugo cms, spanish tech blog, markdown tables, markdown lists, blockquotes, code formatting, content organization, web writing, beginner guide]

params:
  author: "El Viajero"
summary: " "
cover:
    image: "guide_md.png"
    alt: "Guide image"
    relative: false
    hidden: false
    linkFullImages: true
    responsiveImages: false
editPost:
  URL: "mailto:cysecroad@gmail.com"
  Text: "Suggest Changes" # edit text
  appendFilePath: true # to append file path to Edit link
---

# Blog Formatting Guide (because I got tired of googling this stuff)

¡Hola amigos!\
After spending way too many hours fighting with markdown (and accidentally breaking my blog multiple times), I figured I should write down all the stuff that kept tripping me up. If you're here, you probably ran into the same headaches I did.

## Markdown stuff i use

### The Basics (stuff you'll actually use daily)

First things first - let's add some style to our text!

>(One asterisk) -> *italic*\
>(Two asterisks) -> **bold**\
>(When you write something dumb '~') -> ~~crossing stuff out~~\
>(For those \`techy\` bits) -> `code goes here`      

Oh, and if you're wondering why your line breaks aren't working - add a \ at the end of the line.

Also, sometimes, we may need references [^1]
Do it in a following way
> This is a text\[^1]\
>\[^1]: And this would be note

Also, you can do it with \[^bignote] if more space is needed

[^1]: Or notes

---
### Headers (because organization matters... sometimes)

You know what's annoying? When you put #Header without a space and wonder why it's not working. Always put a space after the #:
>\# This works\
>#This doesn't

Pro tip: If you're like me and keep forgetting how many #'s to use, just remember:

- One # is huge (like, way too huge usually)
- Two ## is for main sections
- Three ### is for subsections
- Anything more than that and you might want to rethink your structure


---
### Lists (when your brain needs to dump information)

Two types, depending on how organized you're feeling:
Numbers when order matters:
1. First thing
2. Second thing
   1. Sub-thing (three spaces before the number!)
   2. Another sub-thing

Random stuff (use - or * or +, whatever you remember):
- Thing one
* Thing two
  + Sub-thing (two spaces before the -)
    - Another level (and two more spaces)

---
### Tables (porque sometimes you need them)

Tables in markdown are a pain. Let's be honest - I usually just use an online table generator. But here's the basic structure:

| Header 1 | Header 2 | Header 3 |
|----------|----------|----------|
| Cell 1   | Cell 2   | Cell 3   |
| Cell 4   | Cell 5   | Cell 6   |

| Left | Center | Right |
|:-----|:------:|------:|
| Left | Center | Right |

|Header 1|Header 2|
|-|-|
|Content|Content|

```
| Header 1 | Header 2 | Header 3 |
|----------|----------|----------|
| Cell 1   | Cell 2   | Cell 3   |
| Cell 4   | Cell 5   | Cell 6   |

# Alignment can be decided
| Left | Center | Right |
|:-----|:------:|------:|
| Left | Center | Right |

# And formatting will be done by hugo
|Header 1|Header 2|
|-|-|
|Content|Content|
```

---
### Links

To put a [Link](https://example.com), do it in the following way
> \[Link\](https://example.com)

And to add [Link](https://example.com 'This is the title') title to link, just add it after the URL in parenthesis, with ''
> \[Link\](https://example.com 'This is the title')

To turn URL, or email into a clickable link
- <https://www.example.com>
- <cysecroad@gmail.com>
Just enclose it in <>
> \<https://www.example.com\>\
> \<cysecroad@gmail.com\>

To link internally, lets check out [General Markdown](#general-markdown), just put the link name after #
> \[General Markdown\](#general-markdown)

---
### Images

Embedding an image is *similar* (En espanol) ![Espania](https://cdn.pixabay.com/photo/2012/04/11/15/33/spain-28530_640.png) to adding a link, just add '!' in front
> \!\[Espania\](https://cdn.pixabay.com/photo/2012/04/11/15/33/spain-28530_640.png)

And add the title in the same way, with '' inside the parenthesis
> \!\[Espania\](https://cdn.pixabay.com/photo/2012/04/11/15/33/spain-28530_640.png 'And this is the title)

We can also link images, by adding brackets and then putting a link
[![Espania](https://cdn.pixabay.com/photo/2012/04/11/15/33/spain-28530_640.png)](https://example.com)
>\[\!\[Espania\](https://cdn.pixabay.com/photo/2012/04/11/15/33/spain-28530_640.png)](https://example.com)

---
### Separated text

#### Blockquotes

> You can enclose text in quotation by using > at the beginning of the row
>
>> You can also have the blockquotes nested by using >>
>
> And inside, text can be _decorated_ like normal text


#### Code
```
This is just a simple code block
```
Which can be achieved by enclosing code in pair of ```
>\```\
>Code\
>\```

If we want to highlight syntax (*nosotros queremos*), we can specify the language
```js
console.log('Hello world');
```

>\```js\
>console.log('Hello world');\
>\```

The list of supported languages is vast and can be found [here](https://github.com/jincheng9/markdown_supported_languages)

---
## The hugo stuff i actually use

In hugo, we have some predefined shortcodes that help us style our text better, and embed some things into the content.\
Shortcodes are snippets inside content that are inserted by calling built in or custom templates\
They are always in double braces ({{}}), but for the rendering, i will put them in single braces

Hugo has like a million shortcodes, but here are the ones I actually remember:

### References

Shortcode *ref* returns permalink of given page reference, opens in **new** tab\
[Whoami and why i write]({{% ref "/blog/posts/whoami" %}})\
[Whoami and why i write | What i though Digital Design Verification was]({{% ref "/blog/posts/whoami#what-i-though-digital-design-verification-was" %}})\
[Blog file formatting]({{% ref "/guides/blog-formatting-guide" %}})
>[Whoami and why i write]({% ref "/blog/posts/whoami" %})\
>[Whoami and why i write | What i though Digital Design Verification was]({% ref "/blog/posts/>whoami#what-i-though-digital-design-verification-was" %})\
>[Blog file formatting]({% ref "/guides/blog-formatting-guide" %})

Shortcode *relref* returns permalink of given page reference, but opens in **this** tab\
[Whoami and why i write]({{% relref "/blog/posts/whoami" %}})\
[Whoami and why i write | What i though Digital Design Verification was]({{% relref "/blog/posts/whoami#what-i-though-digital-design-verification-was" %}})\
[Blog file formatting]({{% relref "/guides/blog-formatting-guide" %}})
> [Whoami and why i write]({% relref "/blog/posts/whoami" %})\
> [Whoami and why i write | What i though Digital Design Verification was]({% relref "/blog/posts/whoami#what-i-though-digital-design-verification-was" %})\
> [Blog file formatting]({% relref "/guides/blog-formatting-guide" %})

### Comment 

Of course, we cannot see this, but comment is there
{{% comment %}} TODO: rewrite the paragraph below. {{% /comment %}}
>{% comment %} TODO: rewrite the paragraph below. {% /comment %}

### Image 

To add a figure, for which we can specify parameters, like following:
{{< figure src="https://cdn.pixabay.com/photo/2016/05/28/08/32/elephant-1421167_1280.jpg" caption="An elephant at sunset" height="200vh">}}
do:
>{< figure src="https://cdn.pixabay.com/photo/2016/05/28/08/32/elephant-1421167_1280.jpg" caption="An elephant at sunset" height="200vh">}

Things that can be specified: [*src, link, target, rel, alt, title, caption, class, height, width, loading, attr, attrlink*]

### Highlight

To display a highlighted code sample:
{{< highlight go-html-template >}}
{{ range .Pages }}
  <h2><a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a></h2>
{{ end }}
{{< /highlight >}}

Put it between {highlight}
>{< highlight go-html-template >}\
>{ range .Pages }\
>\<h2>\<a href="{{ .RelPermalink }}">{ .LinkTitle }</a></h2>\
>{ end }\
>{< /highlight >}

There are a lot of [highlight options to apply](https://gohugo.io/functions/transform/highlight/)

### Instagram

To add instagram post, with url: https://www.instagram.com/p/123456/\
Add following 
> {< instagram 123456 >}

### X

To display x post, with url: https://x.com/userUser/status/123456\
Include following
> {< twitter user="userUser" id="123456" >}

### Youtube and Vimeo

To display youtube video, with url: https://www.youtube.com/watch?v=123456\
Include following
>{< youtube 123456 >}

Youtube shortcodes accepts following arguments: [allowFullScreen, autoplay, class, controls, end, loading, loop, mute, start, title]

To display vimeo video, with url: https://vimeo.com/channels/chnlName/123456\
Include following
>{< vimeo 123456 >}

Or
>{< vimeo id="id123" class="my-vimeo-wrapper-class" title="My vimeo video" 123456 >}

