---
layout: post
title: "Blogging with Octopress"
date: 2015-07-22 13:21:06 -0700
comments: true
categories: Octopress
author: Alex Voitau
---
Octopress is a nice and simple blogging platform. Official documentation is great, includes examples and tutorials:
http://octopress.org/docs/. The purpose of this article is to give a quick reference to commands that cover some basic
worfklows. Setup of Octopress is described at http://octopress.org/docs/setup/ and is not covered here.

## Editorial workflow
With a basic and probably common setup with GitHub you will have:

- `source` branch for source code.
- `master` branch for generated HTML code.

Basic steps (including responsible participants):

- create a new blog post with Octopress (content producer)
- submit a pull request to `source` (content producer)
- review and merge pull request (content reviewer)
- generate HTML code from merged code and deploy to `master` with Octopress (Continuous Integration)

## Create a new blog post
Now let's dig into details of Octopress related steps and start with creating a blog post stub with a related
information, including title, date and categories:
```
$ rake new_post["Blogging with Octopress"]
mkdir -p source/_posts
Creating new post: source/_posts/2015-07-22-blogging-with-octopress.markdown
```

Next step is to add some content to a stub, which is done by editing generated markdown file:
```
$ vim source/_posts/2015-07-22-blogging-with-octopress.markdown
```

More details on editing a post: http://octopress.org/docs/blogging/. After any update we can check how the freshly
baked post looks like:
```
$ rake preview
```

As soon as a new post is ready, we just commit our code and submit a pull request:
```
$ git add source/_posts/2015-07-22-blogging-with-octopress.markdown
$ git commit -m "New post: Blogging with Octopress"
```

## Publish updated site
Let's assume that we have a nice and beautiful Continuous Integration system, which on update of the `source` branch
will generate new HTML code for blog and properly deploy it to `master`.

As soon as source code is checked out by CI, we need to setup GitHub as a deployment option, generate HTML code and deploy:
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

It may be related to the fact that root directory of source code and `_deploy` directory for generated HTML have
different git repository branches set: `source` and `master` respectively. Thanks to StackOverflow this workaround was
found:

```
cd _deploy
git pull origin master
cd ..
rake deploy
```

However, if you know a cleaner way to resolve this issue, please share in comments.

Happy blogging!
