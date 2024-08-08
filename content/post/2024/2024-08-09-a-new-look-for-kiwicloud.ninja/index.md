---
title: "A new look for kiwicloud.ninja" # Title of the blog post.
date: 2024-08-09T01:37:05+12:00 # Date of post creation.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
thumbnail: "hugo.jpg" # Sets thumbnail image appearing inside card on homepage.
codeMaxLines: 25 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Blogging
tags:
  - Wordpress
  - Hugo
# comment: false # Disable comment if false.
---

{{< figure src="hugo.jpg" width="100px" class="fig-centered" >}}

I've been a bit slow posting recently, the main cause of which has been increasing frustration with using [WordPress](https://wordpress.com/) as the content management system (CMS) for this site. When it feels like a chore to write up a post and get it published it really demotivates me and I was spending as much (if not more) time fighting with content and layouts in WordPress than I was in actually getting posts written up and published.

Now before all the WordPress devotees descend on me, this was an issue very much of my own making - I'd progressively added different themes and (especially) plugins to the 'old' site over the 9+ years it's been running and during that time had to deal with many changes - often because a plugin would change or become abandoned by its author and then fail to work with more recent versions of WordPress itself.

This became even worse when looking back at older posts which used older syntax/markup for plugins which had since been retired and removed.

All of this came to a head about a month ago when I was attempting to write a new post series (which hopefully I'll complete soon). I experienced a huge number of issues including:

- Tables randomly losing their formatting and needing to be completely restyled
- Unordered lists losing their bullet points altogether
- Code snippets rendering very oddly (or not at all)
- Broken links to images, and image tags referencing IP addresses directly instead of URLs
- Many other minor annoyances with the block editor and interaction with plugins

Since I already write the majority of my notes and documentation in [Markdown](https://www.markdownguide.org/), I decided it was finally time to look for alternatives, and now this site is running with a static Markdown site generate called [Hugo](https://gohugo.io/). The main features of Hugo that appealed were:

- Native way to work with Markdown files
- Flexible layout options and good/easy to use theme library
- Easy template syntax based on [Go](https://go.dev/)
- Great support for code-snippets (which I tend to use quite heavily in posts)
- No backend SQL database to deal with
- No webserver or hosting to deal with (see below)

There are also some really nice-looking shortcodes available in Hugo which I think really help posts look (and read) more easily
{{% notice info "For example" %}}
These 'notice' shortcodes included in the [Hugo Clarity Theme](https://github.com/chipzoller/hugo-clarity) the site is based on I think look fantastic
{{% /notice %}}
The code blocks look nice too:
```bash
#!/bin/bash
echo "I also like the default code blocks"
echo "available 'out of the box' in this theme
```

Unfortunately, the migration tools available from WordPress to Hugo suffered from many of the issues I was experiencing (again, self-inflicted and caused by the large numbers of plugins I'd used over the years) which is involving a considerable amount of work to fix. While I could easily migrate the text content, many links were broken (especially for images) and styling for code, tables and many other (plugin related) bits simply didn't work.

I've been working to fix these up, but haven't got through all of my older posts yet - but given the time elapsed and the fact that many of the products discussed in those posts are now quite old and no longer used I don't think anyone will mind too much. I'll continue to work my way through them regardless and aim to have everything tidied up over the course of the next few weeks.

I've also not yet tackled migrating WordPress comments across to the Github discussions that I've added here, I'd like to keep these if I can, but that looks like it will also involve a reasonable amount of work.

On the positive side, I'm pretty happy with how the site looks and functions with Hugo, there are still a few rough edges, but I'll work on these as time permits. Most importantly (to me), since Hugo is based around Markdown documents it is considerably easier for me to write and publish new posts compared to the issues I was experiencing.

As a bonus, I also no longer need to provide any hosting for the site, it's entirely built in a Github repository and published via Github Actions to a [Github Pages](https://pages.github.com/) site - effectively this means my posting workflow is now simply to create or edit a new Markdown file and commit it to my Github repository. The Github action which triggers on the 'Push' request then rebuilds the site and publishes everything automatically.

This also means I can work from anywhere - I can carry around the repository on my laptop and write while offline but still able to preview the site/content and then just push the changes to the repository when connected. For any other machine I can clone the repository, make changes and push these back easily too.

Hopefully you'll agree that the site looks better - and more importantly I'm happy that it will provide a platform for me to publish from more easily & quickly in future.

Jon