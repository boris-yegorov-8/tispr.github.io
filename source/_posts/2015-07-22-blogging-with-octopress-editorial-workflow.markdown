---
layout: post
title: "Blogging with Octopress: Editorial workflow"
date: 2015-07-22 13:21:06 -0700
comments: true
categories: Octopress
author: Alex Voitau
---
Octopress is an intuitive and simple blogging platform. The official documentation includes numerous examples and tutorials: http://octopress.org/docs. The purpose of this article is to give a quick reference to commands which cover some basic workflows. 

The setup of Octopress is described here: http://octopress.org/docs/setup/ and will not be covered in this article.

With a basic setup of Octopress, GitHub repo will have branches:

- `source` branch for source code.
- `master` branch for generated HTML code.

Basic steps (including responsible participants):

- Create a new blog post with Octopress (content producer)
- Submit a pull request to `source` (content producer)
- Review and merge pull request (content reviewer)
- Generate HTML code from merged code and deploy to `master` with Octopress (automated by Continuous Integration server)

## Create a new blog post
Now let's dig into the details of Octopress and start by creating a blog post stub including title, date, categories and others:
```
$ rake new_post["Blogging with Octopress"]
mkdir -p source/_posts
Creating new post: source/_posts/2015-07-22-blogging-with-octopress.markdown
```

The next step is to edit generated markdown file:
```
$ vim source/_posts/2015-07-22-blogging-with-octopress.markdown
```
More details on editing a post can be found here: http://octopress.org/docs/blogging/. 

After any update you can check what your freshly-baked post looks like:
```
$ rake preview
```

As soon as a new post is ready, you just commit your code and submit a pull request:
```
$ git add source/_posts/2015-07-22-blogging-with-octopress.markdown
$ git commit -m "New post: Blogging with Octopress"
```

## Publish updated site
Let's assume you have a Continuous Integration server, which generates a new HTML code for your blog on update of the `source` branch and deploys it to the `master` afterwards.

As soon as the source code is checked out by CI, you need to setup GitHub as a deployment option, generate the HTML code and deploy:
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

This may be related to the fact that the root directory of the source code and `_deploy` directory for generated HTML have different git repository branches set to `source` and `master` respectively. Thanks to StackOverflow a  workaround was found:

```
cd _deploy
git pull origin master
cd ..
rake deploy
```

However, if you know a cleaner way to resolve this issue, please share in the comments.

Happy blogging!
