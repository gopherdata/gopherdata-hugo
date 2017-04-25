# gopherdata.io

Hugo source and content for [gopherdata.io](http://gopherdata.io).

## Building

Ensure you have a recent version of [hugo](http://gohugo.io) installed.


    git clone --recursive https://github.com/gopherdata/gopherdata-hugo
    cd gopherdata-hugo
    hugo server -w
    # Browse to http://localhost:1313


## Adding a blog post

You can use the `hugo new` command to generate a Markdown template for your content.


    cd gopherdata-hugo
    hugo new /post/my_descriptive_blog_post.md
    # Modify the title section and add your content with a text editor
    vim content/post/my_descriptive_blog_post.md
