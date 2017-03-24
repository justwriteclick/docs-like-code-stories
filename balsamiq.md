# Case Study - Balsamiq Docs: Static Doesn't Have to Be a Limitation

Leon Barnard describes how the second version of the Balsamiq Docs static site went beyond purely functional to do things he didn't think Markdown and static sites could do.


## Background

A year and a half ago [we ditched our flaky content management system](https://blog.balsamiq.com/new-documentation-site/) and converted our documentation site over to a "docs like code" system using [Hugo](http://gohugo.io/), [Gulp](http://gulpjs.com/), and [GitHub](https://github.com/), with content written in Markdown. It was a long process and we were happy just to see it up and running.

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

A simple solution, I suppose, would have been to switch from Markdown to a file format that supports including content from one file in another (like [reStructuredText](http://docutils.sourceforge.net/rst.html) or [AsciiDoc](http://asciidoc.org/)). **But, I'm stubborn** (and Hugo doesn't support them well). I like how simple Markdown is and I wanted to see if we could achieve what we wanted without abandoning it.

I enlisted the help of our resident [Go](https://golang.org/) expert, [Luis](https://balsamiq.com/company/#luis), and he wrote some code to do just what we wanted. This code inside one of the Hugo templates allowed us to put help files in two hidden folders (called "\_v2" and "\_v3") that could get injected into files across multiple documentation folders each time the site was generated. The end result was that users could now find the information where they expected to find it.

![](https://media.balsamiq.com/images/docslikecode/structure5.png)

Instead of changing file formats, we leveraged the flexibility of Hugo to use the meta data (front matter) from the Markdown file as a trigger to tell Hugo to insert content from one file to another.

Here's the code:

```go
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

Hugo looks for two pieces of front matter. The first is the presence of a parameter called `include`. If that exists, it looks to see which version of the editor to use. Then it copies the content from the file with the same name from the core documentation folder ("\_v2" or "\_v3") specified in the editor version parameter.

![](https://media.balsamiq.com/images/docslikecode/structure6.png)

If the `include` parameter isn't present, Hugo just uses the Markdown content in the file without copying content from elsewhere. This allows us to mix and match docs that are unique to one product with docs that share functionality with multiple products.

It took some effort to get it set up, but now we don't even think about it. It's really easy for the content writers, and we'll be able to extend it when we add yet more versions in the future.


## Challenge #2: Adding play/pause to animated GIFs

In a very different challenge, we noticed that our support team was getting great feedback when they would send animated GIFs to customers to explain solutions to problems they were having. The classic case of "show, don't tell." We realized that adding them to our documentation could be helpful too, and we added a few of them in the first version of our docs.

But it can be distracting to read an article when there's an endlessly-looping animation in your line of sight. But if you only play it once, readers could miss it.

I came across several sites that gave users an option to play the animated GIF when they clicked on it and I thought that would be great for our docs.

I found a great jquery plugin that I wanted to use called [**gifplayer**](http://rubentd.com/gifplayer/).

[![You know you want to click](https://media.balsamiq.com/images/docslikecode/gifplayer.png)](http://rubentd.com/gifplayer/).


There was just one problem. Markdown. Again.

Triggering the gifplayer code is as easy as adding a CSS class to an image. Like this:

```html
<img class="gif" src="http://rubentd.com/img/banana.png">
```

But native Markdown doesn't support adding CSS. Of course, Markdown *does* support inline HTML, and we *could have* just written HTML like the line above whenever we wanted to add an animated GIF. But, again, **I'm stubborn**. I like my Markdown to be clean. I didn't want to go down a slippery slope of adding special cases for departing from pure Markdown.

Now, there are two parameters that you give an image in Markdown: the file name, which I needed to tell it where the file is, and alt text, which we often leave blank (I know... accessibility). So I decided to customize the jquery function that initialized gifplayer. Instead of looking for images with a CSS class called 'gif', it would look for images with  alt text set to 'gif'. 

Like this:

```javascript
<script>
  $(document).ready(function(){
    $("img[alt='gif']").gifplayer({ label: 'Play' });
	});
</script>
```

So, now, if we write:

```markdown
![gif](http://rubentd.com/img/banana.png)
```
it'll trigger the gifplayer script!

<img src="http://rubentd.com/img/banana.gif" width=100 alt="party">

You can see it in action on our new article on [common workarounds](https://support.balsamiq.com/tutorials/workarounds/#link-to-alternates). 

(Sorry, it doesn't work on this site. You're gonna have to keep scrolling. ;-))


## Challenge #3: Giving the list pages a makeover

It's hard to make a documentation site look pretty beyond the home page, whether it's a static site or not.

Let's look at the [Dropbox help site home page](https://www.dropbox.com/help) as an example:

![](https://media.balsamiq.com/images/docslikecode/dropbox-help.png)

I love the illustrations and layout. But as soon as you go a level deeper you get this:

![](https://media.balsamiq.com/images/docslikecode/dropbox-help-toc.png)

It's clear to me that this page is automatically generated by some kind of content management system (CMS) template. As a designer for a page like this because you don't want to have to manually update it every time a new article is added, so you cede control of the placement of the articles to the CMS, perhaps using some metadata to rank articles within each category. 

You're limited because you don't know how many articles there will be in each category, so most sites just end up creating a simple unordered list. Even awesome documentation sites like [Mailchimp](http://kb.mailchimp.com/) and [Zapier](https://zapier.com/help/) do this once you go beyond the first level.

And this is exactly what we do on our documentation site currently:

![](https://media.balsamiq.com/images/docslikecode/desktop-toc-old.png)

But I really loved the look of those initial landing pages, like on the Dropbox site. The illustrations, the grid layout, the way it direct you to the information that's most important. I wanted to see if we could make our second-level pages more like that.

Here's what the upcoming version of our docs site will look like. 

![](https://media.balsamiq.com/images/docslikecode/desktop-toc.png)

It feels more like a documentation site landing page, right? It has a featured articles section with the articles that are more important for new users and the rest of the articles are split evenly in three columns. Yet none of it is hard-coded, even the featured "Getting Started" articles.

I'll start by explaining the "Everything else..." section and what makes it different from our previous version.

The links there are automatically placed by Hugo and styled using the [Bootstrap List group](http://getbootstrap.com/components/#list-group) component. That part is pretty easy. The challenge was putting them in columns and making sure that the columns were equal heights, regardless of how many articles there were.

Here's the code I wrote inside the Hugo template to define some variables to use further down:

```go
{{ $featuredRows := 1 }} // number of rows to feature in Getting Started section
{{ $totalCount := len (where .Site.Pages "Section" "desktop") }} // total number of articles in this section
{{ $rowCount := (sub (div (add (add $totalCount 1) (mod $totalCount 3)) 3) $featuredRows) }} // number of rows in each column
```

And when it comes time to populate the columns:

```html
<div class="row mt1">
  <div class="col-xs-12 col-md-4">
    <div class="list-group">
      {{ range first $rowCount (after 3 .Data.Pages.ByWeight) }}
      <a href="{{ .Permalink }}" class="list-group-item">{{ .Title }}</a>
      {{ end }}
    </div>
  </div>
  
  <div class="col-xs-12 col-md-4">
    <div class="list-group">
      {{ range first $rowCount (after (add $rowCount 3) .Data.Pages.ByWeight) }}
      <a href="{{ .Permalink }}" class="list-group-item">{{ .Title }}</a>
      {{ end }}
    </div>
  </div>
  
  <div class="col-xs-12 col-md-4">
    <div class="list-group">
      {{ range first $rowCount (after (add (mul $rowCount 2) 3) .Data.Pages.ByWeight) }}
      <a href="{{ .Permalink }}" class="list-group-item">{{ .Title }}</a>
      {{ end }}
    </div>
  </div>
</div>
```

The variable called `$featuredRows` helps determine where the count should start for columns. And then Hugo counts the number of remaining articles and divides that number by 3 (rounded up to the nearest whole number). It then creates the number of items in each column so that they are as even as possible. It takes a bit of math to do it, but that's what computers are good at anyway.

And now to that "Getting Started" section...

I wanted different images for each of the featured articles, but I just didn't like the idea of hard-coding any links or resources. Yes, because I'm stubborn. So, the top part of the code picks the first three articles (ordered by weight), lists their names and URLs and then grabs an image for each of them using the following code:

```html
<img src="http://media.balsamiq.com/img/support/docs/m4d/b3/toc-button-{{ .File.BaseFileName }}.svg">
```

The last part of the file name for the SVG images is the same as the name of the file that the article links to. So, the article called `intro.md` uses an image called `toc-button-intro.svg`. That way, if we want other articles to show up in the "Getting Started" section we don't need to touch this page, just adjust the weights in the front matter and add a new image that corresponds to the file that will get put into that area.

