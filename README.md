# qudini.github.io

This is a development blog managed by qudini's technical team. 

## How to Submit a Blog Post

* Sign up for a free GitHub account: https://github.com/join
* On GitHub, fork https://github.com/qudini/qudini.github.io
* Use `git clone` to clone your new fork onto your development device.
* `git checkout -b feature/new-blog-post-2017-03-12`
* Add the post to the _posts directory, the file being markdown but with a YAML
  header describing the title, author, and categories.
* `git add _posts`
* `git push`
* Use GitHub's UI to make a pull request from your fork's branch into upstream
  master.
* Just once, `git remote add upstream
  https://github.com/qudini/qudini.github.io.`
* Once the PR is accepted, pull master from upstream back into your fork and
  delete the feature branch in the now-merged PR.

A post can have either an `author` or an `authors` property. The former is just
a shorthand for the latter, specifying only one element.
