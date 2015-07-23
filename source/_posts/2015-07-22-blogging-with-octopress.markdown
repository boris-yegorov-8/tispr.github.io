---
layout: post
title: "Blogging with Octopress"
date: 2015-07-22 13:21:06 -0700
comments: true
categories: Octopress
author: Alex Voitau
---
Octopress is an intuitive and simple blogging platform. The official documentation includes numerous examples and tutorials: http://octopress.org/docs. The purpose of this article is to give a quick reference to commands that cover some basic workflows. 

The setup of Octopress is described here: http://octopress.org/docs/setup/ and will not be covered in this article.

## Editorial workflow
With a basic GitHub setup, you will encounter:

- `source` branch for source code.
- `master` branch for generated HTML code.

Basic steps (including responsible participants):

- Create a new blog post with Octopress (content producer)
- Submit a pull request to `source` (content producer)
- Review and merge pull request (content reviewer)
- Continuous Integration: Generate HTML code from merged code and deploy to `master` with Octopress

## Create a new blog post
1. Now let's dig into the details of Octopress and start by creating a blog post stub including the following information: Title, date and categories:
```
$ rake new_post["Blogging with Octopress"]
mkdir -p source/_posts
Creating new post: source/_posts/2015-07-22-blogging-with-octopress.markdown
```

2. The next step is to add content to a stub, which is done by editing the generated markdown file:
```
$ vim source/_posts/2015-07-22-blogging-with-octopress.markdown
```

More details on editing a post can be found here: http://octopress.org/docs/blogging/. 

3. After any update you can check what your freshly-baked post looks like:
```
$ rake preview
```

4. As soon as a new post is ready, you just commit your code and submit a pull request:
```
$ git add source/_posts/2015-07-22-blogging-with-octopress.markdown
$ git commit -m "New post: Blogging with Octopress"
```

## Publish updated site
Let's assume you have a nifty Continuous Integration system, which can on update of the `source` branch, generate a new HTML code for your blog and properly deploy it to the `master`.

As soon as the source code is verified by CI, you need to setup GitHub as a deployment option, generate the HTML code and deploy:
```
rake setup_github_pages["https://github.com/tispr/tispr.github.io"]
rake generate
rake deploy
```

### Issues
#### Unable to deploy updated content to master
You may experience issues with doing `rake deploy`:

```
## Pushing generated _deploy website
To https://github.com/tispr/tispr.github.io
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/tispr/tispr.github.io'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
Deploy does not work.
```

This may be related to the fact that the root directory of the source code and `_deploy` directory for generated HTML have
different git repository branches set: `source` and `master` respectively. A workaround can be found in Stackoverflow:

```
cd _deploy
git pull origin master
cd ..
rake deploy
```

However, if you know a cleaner way to resolve this issue, please share in the comments.

Happy blogging!
