+++
title = "go-get and Private Repositories"
author = ["Rodrigo Machuca"]
description = "org-mode markup content to be parsed and rendered into CommonMarkdown by ox-hugo"
date = 2021-02-15T20:29:00-06:00
tags = ["golang"]
draft = false
+++

Under the hood, `go get` is using the `git` client to _pull_ the specified
packages. That means that the client should have the appropriate authentication
methods setup in advance to access any private repositories.

If using a private repository, you will get the following error when trying to
download a package:

```text
$ go get github.com/rmachuca89/go-lab
# cd .; git clone -- https://github.com/rmachuca89/go-lab /home/rmachuca/go/src/github.com/rmachuca89/go-lab
Cloning into '/home/rmachuca/go/src/github.com/rmachuca89/go-lab'...
fatal: could not read Username for 'https://github.com': terminal prompts disabled
package github.com/rmachuca89/go-lab: exit status 128
```

To make it work as we expect, we will need to make `go get` change the default
authentication protocol, using one of the options below.


## Force Terminal Prompt with HTTPS Authentication {#force-terminal-prompt-with-https-authentication}

`go get` disables the "terminal prompt" by default, but can be re-enabled by
configuring the `GIT_TERMINAL_PROMPT` environmental variable, either at command
runtime , or by updating our `.bashrc` or equivalent depending on the shell you
are using (`zsh`, `fish`, etc.)

A simple example would be:

```shell
env GIT_TERMINAL_PROMPT=1 go get github.com/<USERNAME>/<MY_PRIVATE_REPO>
```

Replace `USERNAME` and `MY_PRIVATE_REPO` accordingly.

This would ask for your provider Username and Password (or Access Token):

```text
$ env GIT_TERMINAL_PROMPT=1 go get github.com/rmachuca89/go-lab
Username for 'https://github.com': rmachuca89
Password for 'https://rmachuca89@github.com':
```

**WARNING**: Making this change persistent may cause undesirable effects as it
would be considered for every `go get` command, which is designed to be
**non-interactive**. I would suggest you use it only for _ad-hoc one time tasks_
and configure other options mentioned in this article for long-term solutions.

**INFO**: If using [Two Factor Authentication (2FA)](https://docs.github.com/en/github/authenticating-to-github/configuring-two-factor-authentication) you will have to use a [Personal
Access Token (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) or use [SSH authentication](#use-ssh-instead-of-https) instead. Reference: [Using
two-factor authentication with the command line - docs.github.com](https://docs.github.com/en/github/authenticating-to-github/accessing-github-using-two-factor-authentication#using-two-factor-authentication-with-the-command-line)

Personal access tokens function like ordinary OAuth access tokens. They can be
used instead of a password for Git over HTTPS. We require at least the [`repo`
scope](https://docs.github.com/en/developers/apps/scopes-for-oauth-apps) for it to work.


## Use SSH instead of HTTPS: {#use-ssh-instead-of-https}

You can configure `git` client to authenticate using `SSH` for matching _URL
prefix_ as default. The exact command may vary depending on which provider you
are using (GitLab, BitBucket), so make sure you _adjust the domain name_ as
required.

GitHub:

```text
git config --global url."ssh://git@github.com/".insteadOf "https://github.com/"
```

This is actually updating the `.gitconfig` file under the hood.

In addition to that, configure your SSH as usual (i.e. make `git clone` work if
called manually). This again may vary per git provider, but an example is
documented here: [Connecting to GitHub with SSH](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)

After doing that the command should be working now:

```text
$ git config --global url."ssh://git@github.com/".insteadOf "https://github.com/"
$ go get github.com/rmachuca89/go-lab
```


## The `.netrc` File with HTTPS Authentication {#the-dot-netrc-file-with-https-authentication}

This is actually an official recommended way to setup authentication options
provided on the [golang faq section](https://golang.org/doc/faq#git%5Fhttps), and uses the [`.netrc` file](https://www.gnu.org/software/inetutils/manual/html%5Fnode/The-%5F002enetrc-file.html) for network
authentication configuration.

To authenticate over HTTPS, you can add a line to the `$HOME/.netrc` file
that `git` consults:

```text
machine github.com login USERNAME password APIKEY
```

**INFO**: For GitHub accounts, the password can be a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

**WARNING**: Remember that `.netrc` is a plain-text file, so make sure that its
corresponding file access control lists (ACLs) are restricted (i.e. `chmod 600
~/.netrc`).


## References: {#references}

-   [Why does "go get" use HTTPS when cloning a repository? | Golang Docs](https://golang.org/doc/faq#git%5Fhttps)
-   [go get results in "terminal prompts disabled" error for github private repo | Stack Overflow](https://stackoverflow.com/questions/32232655/go-get-results-in-terminal-prompts-disabled-error-for-github-private-repo)
