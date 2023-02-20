---
title: "Comment Toots"
date: 2023-02-19T20:23:53-08:00
description: "An attempt to power blog comments with Mastodon"
images:
  - "/images/comment-toots/mastodon-derek.png"
tags:
 - meta
 - hugo
 - mastodon
 - development
comments:
  host: mas.to
  username: bishma
  id:
draft: true
---

## {{< param description >}}

<img align="left" src="/images/comment-toots/mastodon-derek.png" width="280" hspace="25">

Well, I guess I've landed on [Mastodon](https://mas.to/@bishma) since the bird started shitting everywhere. I was resistant since I don't think it can ever be the real-time discovery engine that Twitter was (at its best). But, seeing as I spent an afternoon linking Mastodon threads as a comment system for this [Hugo](https://gohugo.io/) blog, I guess the choice has been made.

The downside to this setup is that it requires some editing every time I want to have comments on a post. To enabled comments on a post I need to

* Publish the post to my blog
* Wait for it to deploy
* Write a toot about it
* Get the ID of that Toot
* Edit the post adding the TootID to it
* Wait for it to redeploy

Hopefully I'll find a way to automate some or all of this process with [Github Actions](https://github.com/features/actions). But that's a problem for future Derek. And that guy's a jerk.

This whole concept isn't my idea, this is just my take on it. I stumbled on this idea by [Carl Schwan](https://carlschwan.eu/2020/12/29/adding-comments-to-your-static-blog-with-mastodon/) while looking for something else entirely. It's pretty straight forward; A bit of extra YAML each post's [Front Matter](https://gohugo.io/content-management/front-matter/) plus a bit of js, css, and html hacked into my [theme](https://github.com/lxndrblz/anatole). I restyled a number of things and made it so that comments automatically lazy-load, rather than requiring the visitor to click a button. For anyone following along, if you see me reference my `anatole/` directory replace that with the directory of your theme. It should be set up in a similar way, but your milage may vary.

### Step 1: Editing the post head file

Mastodon pulls page information from it Open Graph tags. At the time I got it, my theme didn't have Open Graph incorporated (it does now). Of course, I've done so much hacking at things that I don't dare update for one simple issue like this. Hugo already provides an open graph partial, so I just needed to [add](https://github.com/Bishma/blog/commit/7517756515fbe09d236f76e164b9e91ccf8b20a7) it to the `anatole/layouts/partials/head.html` file.

```golang
{{ template "_internal/opengraph.html" . }}
```

The same `head.html` needs to load the CSS for my comments, so I [added that](https://github.com/Bishma/blog/commit/78b61e93c1eae6f01b32bdc7e92f45c7e3d3fb5d) in the file while I was in there. More about CSS specifics later, for now what you need to know is that we're going to reference a css file that we'll be creating inside the Anatole theme. We also Minify it and Fingerprint in the process. The file itself was put in the theme as `anatole/assets/js/mastodonComments.css`. Get the [latest version of my CSS](https://github.com/Bishma/blog/blob/master/themes/anatole/assets/css/mastodonComments.css).

```html
{{ $mastodonCommentStyle := resources.Get "css/mastodonComments.css" | resources.Minify | resources.Fingerprint }}
  <link rel="stylesheet"
    href="{{ $mastodonCommentStyle.Permalink }}"
    integrity="{{ $mastodonCommentStyle.Data.Integrity }}"
    crossorigin="anonymous"
    type="text/css">
```

### Step 2: Adding Front Matter

Each Hugo post/page has a small section of YAML (I use YAML, TOML is an option) that defines all the meta data about the post. This is where were going to tell the post how to find the right Toot to use as that post's comment thread. This needs to be added somewhere in the YAML block.

```yaml
comments:
  host: <mastodon server I'm signed up on>
  username: <my username on that server>
  id: 
```

The YAML above was also added to my default post template page located in `/archetypes/default.md` so that it would be available automatically on all my future posts.

### Step 3: The Comments Partial

All the interesting bits are on the comments partial. This was placed inside my theme's partials directory `anatole/layouts/partials/comments.html`. You can find the latest version of my `comments.html` partial [here](https://github.com/Bishma/blog/blob/master/themes/anatole/layouts/partials/comments.html).

#### Details of comments.html partial

Replying isn't straight forward with this solution. People need a Mastodon account (somewhere) and then they need to post a reply **from** it. What Corl did in his solution was to add a dialog that would pop up and explain things to people. I didn't didn't want to have to style that modal so I just made it inline text that displays below the Comments subheader, provided there is a toot idea present in the posts Front Matter. If there is no Toot ID in the post's front matter, this paragraph will be hidden.

```golang
{{ if not .id  }}
  <p style="display: hidden;">
{{ else }}
  <p>
{{ end }}
    You can use your Mastodon account to reply to this <a class="link" href="https://{{ .host }}/@{{ .username }}/{{ .id }}" target="_blank">post</a>. <br />
    Or <a href="#" id="copyButton">copy</a> and paste the post URL into the search field of the Mastodon client of your choice.
  </p>
```

The `copyButton` href above has an onclick listener further down the page. It will copy my full mastodon link (sourced from the post's Front Matter) to their clipboard. And it gives a thumbs up for 2 seconds to show it worked.

```javascript
document.getElementById('copyButton').addEventListener('click', (e) => {
  navigator.clipboard.writeText('https://{{ .host }}/@{{ .username }}/{{ .id }}');
  e.preventDefault();

  // Save the original
  const $cpLk = $("#copyButton");      
  const originalText = $cpLk.text();
  console.log(originalText)

  // Change the text for 2 seconds
  $cpLk.text("👍");
  
  setTimeout(function() {
    $cpLk.text(originalText);
  }, 2000);
});
```

When it came to having Toots loaded onto a post, I wanted to remove the need to click. Caaarl used a UI button to let users trigger comment loading themselves, but I'm not a god that believes in free will. So I load the comments by a lazy-load instead, which triggers 100px from the bottom of the page. I pull in the Toot ID from the page's Front Matter, and if it's empty it will say comments are disabled.

```javascript
// If comments havent been loaded yet, load them when the user scrolls to within 100px of the bottom of the page
let limitBottom = document.documentElement.offsetHeight - window.innerHeight - 100;
let commentsLoaded = false;
window.addEventListener("scroll",function(){
  // 3 conditions
  // 1. The user has scrolled to within 100px of the bottom of the page
  // 2. Comments have not been loaded yet
  // 3. The post has a Mastodon ID (from Params.comments)
  if(document.documentElement.scrollTop >= limitBottom && !commentsLoaded && '{{ .id  }}' != ''){
    loadComments();
    commentsLoaded = true;
  } else if ('{{ .id  }}' == '') {
    document.getElementById('mastodon-comments-list').innerHTML = "<em>Comments are disabled for this post.</em>";
  }
})
```

Once the comments partial is [downloaded](https://github.com/Bishma/blog/blob/master/themes/anatole/layouts/partials/comments.html) to the correct directory in the template, the partial can be called from inside the single page layout (in `anatole/layouts/single.html`). Just above the footer content I [added](https://github.com/Bishma/blog/commit/5f1b177ad9d3b65e084e899ff63b103f81b8b137) a test to make sure we're in a blog post, inside of which is a call to the partial.

```golang
{{ if eq .Type "posts"}}
  {{ partial "comments.html" . }}
{{ end }}
```

Now we need to style and js things.

### Step 4: CSS and JS, the terror twins

Most of you looking at this blog instinctively know that I'm a backend guy. I'm a lot happier working with IaaS, than I am working with CSS. But since no one joined by dial-in BBS, here we are.

First I need a 3rd party JS library called [DOMPurify](https://github.com/cure53/DOMPurify/blob/main/dist/purify.js). It's used to sanitize each reply toot to catch anything injecty or XSSy coming through. I downloaded the latest version and put it in my theme's js directory, `anatole/asset/js`. It gets called by the comments partial discussed above.

We also need the CSS that was mentioned above. I already downloaded it and it's called from inside the comments partial discussed above. It is clumsy AF but it gets the job done. It styles the commenters avatar, display name, and username along with their comments. I may not like everyone's avatars being on my site, so this it the place to tweak that display.

### Step 5: Post this and see what happens.

We'll see how long this takes to break or how soon I realize that testing in FF and Chrome in a single OS is not sufficient.

Really this is just a toy that kept me busy for around a day. But I had fun and I learned more about Hugo in the process.
