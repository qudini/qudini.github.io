---
layout: post
title:  "git merge vs. git rebase"
author: christophe_maillard
date:   2017-03-22 16:42:00 +0000
categories: qudini git merge rebase
---
## Preamble

This post is widely oriented around [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/). The tables have been generated with the nice [ASCII Table Generator](https://ozh.github.io/ascii-tables/), and the history trees with this wonderful command ([aliased](https://githowto.com/aliases) as `git lg`):

```text
git log --graph --abbrev-commit --decorate --date=format:'%Y-%m-%d %H:%M:%S' --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%ad%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)'
```

Tables are in reverse chronological order to be more consistent with the history trees. See also the difference between `git merge` and `git merge --no-ff` first (you usually want to use `git merge --no-ff` as it makes your history look closer to the reality):

#### `git merge`

Commands:

```text
Time          Branch "develop"             Branch "features/foo"
------- ------------------------------ -------------------------------
15:04   git merge features/foo
15:03                                  git commit -m "Third commit"
15:02                                  git commit -m "Second commit"
15:01   git checkout -b features/foo
15:00   git commit -m "First commit"
```

Result:

```text
* 142a74a - YYYY-MM-DD 15:03:00 (XX minutes ago) (HEAD -> develop, features/foo)
|           Third commit - Christophe
* 00d848c - YYYY-MM-DD 15:02:00 (XX minutes ago)
|           Second commit - Christophe
* 298e9c5 - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

#### `git merge --no-ff`

Commands:

```text
Time           Branch "develop"              Branch "features/foo"
------- -------------------------------- -------------------------------
15:04   git merge --no-ff features/foo
15:03                                    git commit -m "Third commit"
15:02                                    git commit -m "Second commit"
15:01   git checkout -b features/foo
15:00   git commit -m "First commit"
```

Result:

```text
*   1140d8c - YYYY-MM-DD 15:04:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/foo' - Christophe
| * 69f4a7a - YYYY-MM-DD 15:03:00 (XX minutes ago) (features/foo)
| |           Third commit - Christophe
| * 2973183 - YYYY-MM-DD 15:02:00 (XX minutes ago)
|/            Second commit - Christophe
* c173472 - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

---

## `git merge` vs `git rebase`

First point: **always merge features into develop, never rebase develop from features**. This is a consequence of the [Golden Rule of Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing):

> The golden rule of `git rebase` is to never use it on *public* branches.

[In other words](https://git-scm.com/book/en/v2/Git-Branching-Rebasing):

> Never rebase anything you've pushed somewhere.

I would personally add: *unless it's a feature branch AND you and your team are aware of the consequences*.

So the question of `git merge` vs `git rebase` applies almost only to the feature branches (in the following examples, `--no-ff` has always been used when merging). Note that since I'm not sure there's one better solution ([a debate exists](https://www.atlassian.com/git/articles/git-team-workflows-merge-or-rebase)), I'll only provide how both commands behave. In my case, I prefer using `git rebase` as it produces a nicer history tree :)

### Between feature branches

#### `git merge`

Commands:

```text
Time           Branch "develop"              Branch "features/foo"           Branch "features/bar"
------- -------------------------------- ------------------------------- --------------------------------
15:10   git merge --no-ff features/bar
15:09   git merge --no-ff features/foo
15:08                                                                    git commit -m "Sixth commit"
15:07                                                                    git merge --no-ff features/foo
15:06                                                                    git commit -m "Fifth commit"
15:05                                                                    git commit -m "Fourth commit"
15:04                                    git commit -m "Third commit"
15:03                                    git commit -m "Second commit"
15:02   git checkout -b features/bar
15:01   git checkout -b features/foo
15:00   git commit -m "First commit"
```

Result:

```text
*   c0a3b89 - YYYY-MM-DD 15:10:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/bar' - Christophe
| * 37e933e - YYYY-MM-DD 15:08:00 (XX minutes ago) (features/bar)
| |           Sixth commit - Christophe
| *   eb5e657 - YYYY-MM-DD 15:07:00 (XX minutes ago)
| |\            Merge branch 'features/foo' into features/bar - Christophe
| * | 2e4086f - YYYY-MM-DD 15:06:00 (XX minutes ago)
| | |           Fifth commit - Christophe
| * | 31e3a60 - YYYY-MM-DD 15:05:00 (XX minutes ago)
| | |           Fourth commit - Christophe
* | |   98b439f - YYYY-MM-DD 15:09:00 (XX minutes ago)
|\ \ \            Merge branch 'features/foo' - Christophe
| |/ /
|/| /
| |/
| * 6579c9c - YYYY-MM-DD 15:04:00 (XX minutes ago) (features/foo)
| |           Third commit - Christophe
| * 3f41d96 - YYYY-MM-DD 15:03:00 (XX minutes ago)
|/            Second commit - Christophe
* 14edc68 - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

#### `git rebase`

Commands:

```text
Time           Branch "develop"              Branch "features/foo"           Branch "features/bar"
------- -------------------------------- ------------------------------- -------------------------------
15:10   git merge --no-ff features/bar
15:09   git merge --no-ff features/foo
15:08                                                                    git commit -m "Sixth commit"
15:07                                                                    git rebase features/foo
15:06                                                                    git commit -m "Fifth commit"
15:05                                                                    git commit -m "Fourth commit"
15:04                                    git commit -m "Third commit"
15:03                                    git commit -m "Second commit"
15:02   git checkout -b features/bar
15:01   git checkout -b features/foo
15:00   git commit -m "First commit"
```

Result:

```text
*   7a99663 - YYYY-MM-DD 15:10:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/bar' - Christophe
| * 708347a - YYYY-MM-DD 15:08:00 (XX minutes ago) (features/bar)
| |           Sixth commit - Christophe
| * 949ae73 - YYYY-MM-DD 15:06:00 (XX minutes ago)
| |           Fifth commit - Christophe
| * 108b4c7 - YYYY-MM-DD 15:05:00 (XX minutes ago)
| |           Fourth commit - Christophe
* |   189de99 - YYYY-MM-DD 15:09:00 (XX minutes ago)
|\ \            Merge branch 'features/foo' - Christophe
| |/
| * 26835a0 - YYYY-MM-DD 15:04:00 (XX minutes ago) (features/foo)
| |           Third commit - Christophe
| * a61dd08 - YYYY-MM-DD 15:03:00 (XX minutes ago)
|/            Second commit - Christophe
* ae6f5fc - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

### From `develop` to a feature branch

#### `git merge`

Commands:

```text
Time           Branch “develop"              Branch "features/foo"           Branch "features/bar"
------- -------------------------------- ------------------------------- -------------------------------
15:10   git merge --no-ff features/bar
15:09                                                                    git commit -m “Sixth commit"
15:08                                                                    git merge --no-ff development
15:07   git merge --no-ff features/foo
15:06                                                                    git commit -m “Fifth commit"
15:05                                                                    git commit -m “Fourth commit"
15:04                                    git commit -m “Third commit"
15:03                                    git commit -m “Second commit"
15:02   git checkout -b features/bar
15:01   git checkout -b features/foo
15:00   git commit -m “First commit"
```

Result:

```text
*   9e6311a - YYYY-MM-DD 15:10:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/bar' - Christophe
| * 3ce9128 - YYYY-MM-DD 15:09:00 (XX minutes ago) (features/bar)
| |           Sixth commit - Christophe
| *   d0cd244 - YYYY-MM-DD 15:08:00 (XX minutes ago)
| |\            Merge branch 'develop' into features/bar - Christophe
| |/
|/|
* |   5bd5f70 - YYYY-MM-DD 15:07:00 (XX minutes ago)
|\ \            Merge branch 'features/foo' - Christophe
| * | 4ef3853 - YYYY-MM-DD 15:04:00 (XX minutes ago) (features/foo)
| | |           Third commit - Christophe
| * | 3227253 - YYYY-MM-DD 15:03:00 (XX minutes ago)
|/ /            Second commit - Christophe
| * b5543a2 - YYYY-MM-DD 15:06:00 (XX minutes ago)
| |           Fifth commit - Christophe
| * 5e84b79 - YYYY-MM-DD 15:05:00 (XX minutes ago)
|/            Fourth commit - Christophe
* 2da6d8d - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

#### `git rebase`

Commands:

```text
Time           Branch “develop"              Branch "features/foo"           Branch "features/bar"
------- -------------------------------- ------------------------------- -------------------------------
15:10   git merge --no-ff features/bar
15:09                                                                    git commit -m “Sixth commit"
15:08                                                                    git rebase development
15:07   git merge --no-ff features/foo
15:06                                                                    git commit -m “Fifth commit"
15:05                                                                    git commit -m “Fourth commit"
15:04                                    git commit -m “Third commit"
15:03                                    git commit -m “Second commit"
15:02   git checkout -b features/bar
15:01   git checkout -b features/foo
15:00   git commit -m “First commit"
```

Result:

```text
*   b0f6752 - YYYY-MM-DD 15:10:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/bar' - Christophe
| * 621ad5b - YYYY-MM-DD 15:09:00 (XX minutes ago) (features/bar)
| |           Sixth commit - Christophe
| * 9cb1a16 - YYYY-MM-DD 15:06:00 (XX minutes ago)
| |           Fifth commit - Christophe
| * b8ddd19 - YYYY-MM-DD 15:05:00 (XX minutes ago)
|/            Fourth commit - Christophe
*   856433e - YYYY-MM-DD 15:07:00 (XX minutes ago)
|\            Merge branch 'features/foo' - Christophe
| * 694ac81 - YYYY-MM-DD 15:04:00 (XX minutes ago) (features/foo)
| |           Third commit - Christophe
| * 5fd94d3 - YYYY-MM-DD 15:03:00 (XX minutes ago)
|/            Second commit - Christophe
* d01d589 - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

---

## Side notes

#### `git cherry-pick`

When you just need one specific commit, `git cherry-pick` is a nice solution (the `-x` option appends a line that says "*(cherry picked from commit...)*" to the original commit message body, so it's usually a good idea to use it - `git log <commit_sha1>` to see it):

Commands:

```text
Time           Branch “develop"              Branch "features/foo"                Branch "features/bar"           
------- -------------------------------- ------------------------------- -----------------------------------------
15:10   git merge --no-ff features/bar                                                                            
15:09   git merge --no-ff features/foo                                                                            
15:08                                                                    git commit -m “Sixth commit"             
15:07                                                                    git cherry-pick -x <second_commit_sha1>  
15:06                                                                    git commit -m “Fifth commit"             
15:05                                                                    git commit -m “Fourth commit"            
15:04                                    git commit -m “Third commit"                                             
15:03                                    git commit -m “Second commit"                                            
15:02   git checkout -b features/bar                                                                              
15:01   git checkout -b features/foo                                                                              
15:00   git commit -m “First commit"    
```                                                                          

Result:

```text
*   50839cd - YYYY-MM-DD 15:10:00 (XX minutes ago) (HEAD -> develop)
|\            Merge branch 'features/bar' - Christophe
| * 0cda99f - YYYY-MM-DD 15:08:00 (XX minutes ago) (features/bar)
| |           Sixth commit - Christophe
| * f7d6c47 - YYYY-MM-DD 15:03:00 (XX minutes ago)
| |           Second commit - Christophe
| * dd7d05a - YYYY-MM-DD 15:06:00 (XX minutes ago)
| |           Fifth commit - Christophe
| * d0d759b - YYYY-MM-DD 15:05:00 (XX minutes ago)
| |           Fourth commit - Christophe
* |   1a397c5 - YYYY-MM-DD 15:09:00 (XX minutes ago)
|\ \            Merge branch 'features/foo' - Christophe
| |/
|/|
| * 0600a72 - YYYY-MM-DD 15:04:00 (XX minutes ago) (features/foo)
| |           Third commit - Christophe
| * f4c127a - YYYY-MM-DD 15:03:00 (XX minutes ago)
|/            Second commit - Christophe
* 0cf894c - YYYY-MM-DD 15:00:00 (XX minutes ago)
            First commit - Christophe
```

#### `git pull --rebase`

Not sure I can explain it better than [Derek Gourlay](https://www.derekgourlay.com/blog/git-when-to-merge-vs-when-to-rebase/)... Basically, use `git pull --rebase` instead of `git pull` :) What's missing in the article though, is that [you can enable it by default](https://coderwall.com/p/tnoiug/rebase-by-default-when-doing-git-pull):

```text
git config --global pull.rebase true
```

#### `git rerere`

Again, nicely explained [here](https://git-scm.com/blog/2010/03/08/rerere.html). But put simple, if you enable it, you won't have to resolve the same conflict multiple times anymore.

---

*Originally posted on [Stack Overflow](http://stackoverflow.com/a/42165259/1225328).*
