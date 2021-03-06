
## Welcome

Thanks for stopping by. This is my website where I host my bio and blog posts. Thank you to Mark Otto for authoring this theme.

Feel free to look around. The website is [here](https://jerry-ye-xu.github.io/)

## Contents

- [Introduction](#introduction)
- [About Me](#about-me)
- [Installing Jekyll](#installing-jekyll)
- [Using MathJax](#using-mathjax)
- [Adding ToC](#adding-toc)
- [Shadow](#shadow)
- [Author](#author)
- [About Hyde](#about-hyde)
  - [Acknowledgements](#acknowledgements)
  - [License](#license)

## About me

Jerry is a student living down under and fully utilising the privileges of Github Pages.

This is a Jekyll based website and pages are written in Markdown and fit into the html structure specified in `_layouts`.

This means that writing blog posts is relatively easier because we simply use markdown for all our blog posts and not have to worry.

## Installing Jekyll

Instructions may be different for non-MacOS users.

To test locally, you first need to install jekyll
```
gem install jekyll
```
If you run into issues with `FilePermissionsError`, then you will need to set up `rbenv` properly.
```
brew install rbenv
```
and follow the instructions [here]("https://github.com/rbenv/rbenv").

If you run into issues with `shims`, you may need to manually export it in your terminal.
```
export PATH="/Users/Jerry/.rbenv/shims:${PATH}"
```
Which is copied directly from Line 1 of the `init` file

You can view the `init` file using
```
rbenv init -
```
## Testing Locally

Once you no longer run into `FilePermissionsError`, you should be able to test your build locally useing `

```bash
jekyll serve
```

## Using MathJax

In order to use MathJax to type math, you will need to add the script below to `head.html` inside the `_includes/` directory.

```
<!-- MathJax -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    inlineMath: [['$','$']]
  }
});
</script>
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
```
__Defining Macros__

This [post](https://stackoverflow.com/questions/24628668/how-to-define-custom-macros-in-mathjax) may be useful for getting started.

The [docs](http://docs.mathjax.org/en/latest/tex.html) are still the best for a detailed walkthrough.

## Adding ToC

Check these guys out [here](https://github.com/toshimaru/jekyll-toc) and follow instructions =).

UPDATE: The above does not work with Kramdown & Github Pages... the solution is to use [this](http://www.seanbuscay.com/blog/jekyll-toc-markdown/)

## Shadow

The shadow I used can be found [here](https://codepen.io/ibrahimjabbari/pen/ozinB).

---

## Author

Jerry Xu

---

Lastly I wanted to thank the author of Hyde & Poole for making such a wonderful theme for the community. Details can be found below.

## About Hyde

Hyde is a brazen two-column [Jekyll](http://jekyllrb.com) theme that pairs a prominent sidebar with uncomplicated content. It's based on [Poole](http://getpoole.com), the Jekyll butler.

![Hyde screenshot](https://f.cloud.github.com/assets/98681/1831228/42af6c6a-7384-11e3-98fb-e0b923ee0468.png)

### Acknowledgements

**Mark Otto**
- <https://github.com/mdo>
- <https://twitter.com/mdo>

Mark is the author of this theme upon which I used to build my own website.

### License

Open sourced under the [MIT license](LICENSE.md).

<3

---

### WORKLOG
- 15/12/19: 0.0.7 - Added responsive image for `about_me` page
- 12/10/19: 0.0.6 - Updated tagline and description in the side bar
- 04/7/19: 0.0.5 - Added blog post page to show all past blog posts, added tags and categories.
- 27/6/19: 0.0.4 - Get ToC working, added shadow to separate posts in home page.
- 25/6/19: 0.0.3 - Added MathJax, updated about me page with photo.
- 24/6/19: 0.0.2 - Added data science for students page, revised README.md page.
- 23/6/19: 0.0.1 - Initial commit, removed all relative links and updated about me page.
