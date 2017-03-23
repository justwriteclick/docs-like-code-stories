# Case Study - Balsamiq Docs: Static Doesn't Have to Be a Limitation

Leon Barnard describes how the second version of the Balsamiq Docs static site went beyond purely functional to do things he didn't think markdown and static sites could do.


## Background

A year and a half ago [we ditched our flaky content management system](https://blog.balsamiq.com/new-documentation-site/) and converted our documentation site over to a "docs like code" system using [Hugo](http://gohugo.io/), [Gulp](http://gulpjs.com/), and [GitHub](https://github.com/), with content written in markdown. It was a long process and we were happy just to see it up and running.

After the dust settled, we started imagining what we wanted to do for the next version and realized that our system had limitations that we would have to overcome. This is a story of three challenges and what we did to solve them.

***Note:*** *These are specific to [Hugo](http://gohugo.io/), the static site generator we use, but should be adaptable to others.*

## Challenge #1: Documenting multiple product versions

Our product, [Balsamiq Mockups](https://balsamiq.com/), comes in multiple "flavors," as we often call them. There are two versions of the core product (the wireframe editor) that are sold as: a stand-alone desktop version, a hosted web version, and plugins for Google Drive and Atlassian Confluence and JIRA. 

![](https://media.balsamiq.com/images/docslikecode/select-product.png)

There are seven different products that we sell, catered to different types of users. This is great for our customers, but a challenge for our docs.

The majority of our documentation is how to use one of the two core wireframe editors. Very little has to do with platform-specific features. So, seven products that we sell, but only two main sets of documentation. 

For historical reasons, most of the common documentation is in the Desktop version docs. If you used our product on different platform, you had to know to go to the Desktop docs. 

This is what the content folder in our [GitHub repo](https://github.com/balsamiq/docs.balsamiq.com) looked like, which meant that a lot of customers had this experience:

![](https://media.balsamiq.com/images/docslikecode/structure2.png)

For the next update to our docs, we decided that we needed to finally address this issue. We wanted a complete set of docs for each product version and platform.

![](https://media.balsamiq.com/images/docslikecode/structure3.png)

But, of course, we didn't want to maintain seven copies of each article. We wanted to be able to write the docs once and have them show up somehow across multiple versions.

A simple solution, I suppose, would have been to switch from markdown to a file format that supports including content from one file in another (like [reStructuredText](http://docutils.sourceforge.net/rst.html) or [AsciiDoc](http://asciidoc.org/)). **But, I'm stubborn** (and Hugo doesn't support them well). I like how simple markdown is and I wanted to see if we could achieve what we wanted without abandoning it.

Eventually, we came up with a way to do just what we wanted, creating articles for each core version of the product ("v2" and "v3") that would get inserted into files in multiple documentation folders.

![](https://media.balsamiq.com/images/docslikecode/structure5.png)

But instead of changing file formats, we leveraged the flexibility of Hugo to use the meta data (front matter) from the markdown file as a trigger to tell Hugo to copy the content from one file to another.

Here's the code:

```
{{ if .Params.include }}
  {{ $include := .Params.include }}
  {{ if eq .Params.editorversion 3 }}
    {{ range where .Site.Pages "Params.menu" "menub3editor" }}
      {{ $file := .File.BaseFileName }}
      {{ if eq $file $include }}
        {{ .Content }}
      {{ end }}
    {{ end }}
  {{ else if eq .Params.editorversion 2 }}
    {{ range where .Site.Pages "Params.menu" "menub2editor" }}
      {{ $file := .File.BaseFileName }}
      {{ if eq $file $include }}
        {{ .Content }}
      {{ end }}
    {{ end }}
  {{ else }}
    {{ end }}
{{ else }}
  {{ .Content }}
{{ end }}

```

Hugo looks for two pieces of front matter. The first is the presence of a parameter called `include`. If that exists, it looks to see which version of the editor to use. Then it copies the content from the file with the same name from the core documentation folders.

![](https://media.balsamiq.com/images/docslikecode/structure6.png)

If the `include` parameter isn't present, it just uses the content of the file without copying content from elsewhere. This allows us to mix and match docs that are unique to one product with docs that share functionality with multiple products.

It took some effort to get it set up, but now we don't even think about it. It's really easy for the content writers, and we'll be able to extend it when we add yet more versions in the future.


## Challenge #2: Adding play/pause to animated GIFs

We noticed a while ago that our support team was getting great feedback when they would send animated GIFs to customers to explain solutions to problems they were having. The classic case of "show, don't tell." We realized that adding them to our documentation could be helpful too, and we added a few of them in the first version of our docs.

But it can be distracting to read an article when there's an endlessly-looping animation in your line of sight. But if you only play it once, readers could miss it.

I came across several sites that gave users an option to play the animated GIF when they clicked on it and I thought that would be great for our docs.

I found a great jquery plugin that I wanted to use called [**gifplayer**](http://rubentd.com/gifplayer/).

[![You know you want to click](https://media.balsamiq.com/images/docslikecode/gifplayer.png)](http://rubentd.com/gifplayer/).


There was just one problem. Markdown. Again.

Triggering the gifplayer code is as easy as adding a CSS class to an image. Like this:

```
<img class="gif" src="http://rubentd.com/img/banana.png">
```

But native markdown doesn't support adding CSS. Of course, markdown *does* support inline HTML, and we *could have* just written HTML like the line above whenever we wanted to add an animated GIF. But, again, **I'm stubborn**. I like my markdown to be clean. I didn't want to go down a slippery slope of adding special cases for departing from pure markdown.

Now, there are two parameters that you give an image in markdown: the file name, which I needed to tell it where the file is, and alt text, which we often leave blank (I know... accessibility). So I decided to customize the jquery function that initialized gifplayer. Instead of looking for images with a CSS class called 'gif', it would look for images with  alt text set to 'gif'. 

Like this:

```
<script>
  $(document).ready(function(){
    $("img[alt='gif']").gifplayer({ label: 'Play' });
	});
</script>
```

So, now, if we write:

```
![gif](http://rubentd.com/img/banana.png)
```
it'll trigger the gifplayer script!

<img src="http://rubentd.com/img/banana.gif" width=100 alt="party">

You can see it in action on our new article on [common workarounds](https://support.balsamiq.com/tutorials/workarounds/#link-to-alternates). 

(Sorry, it doesn't work on this site. You're gonna have to keep scrolling. ;-))


## Challenge #3: The table of contents pages shouldn't be so boring

**TBD:** is it already too long?
