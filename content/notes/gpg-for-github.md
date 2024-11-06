---
author: Ke Xue
title: GPG for GitHub
date: 2024-11-05T21:38:04-05:00
draft: false
layout: docs
description: Use GPG keys to sign GitHub commits.
tags: 
- git
---


GPG, or GNU Privacy Guard, is a free-software replacement for Symantec's PGP cryptographic software suite.
GPG keys are not based on a new cryptographic algorithm, but simply those generated by the GPG software.
The algorithms used by GPG are the commonly used ones.

`gpg` is a CLI tool for managing keys, and the keys it generates have more features. For example, GPG keys can
expire or be revoked. With GitHub, SSH keys are used for authentication, while GPG keys are used for
signing commits and tags. Commit-signing makes sure the committer is indeed who he/she claims to be ---
otherwise any one can claim to be any one by setting username and password with `git config`.

## Managing GPG Keys

The commands in this section applies to `gpg` version `2.1.17` or greater.

## List existing keys

```bash
gpg --list-secret-keys --keyid-format=long
```

If the output of the command above is something like,

``` 
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot <hubot@example.com>
ssb   4096R/4BB6D45482678BE3 2016-03-10
```

the `$KEYID` that we can use in most other commands is `3AA5C34371567BD2`.

## Export public key to base64-encoded format

```bash
gpg --armor --export $KEYID
```

The output can be copy-pasted to GitHub web UI. For signing purposes, public key suffices.

## Generate a new key

```bash
gpg --full-generate-key
```

- For the "kind" of key, "RSA (sign only)" is enough for *signing* commits and tags.
- For expiration, GitHub recommends taking the default of no-expiration date.
- For email address, use one that has been verified with the GitHub account.
- Passphrase won't be a hassle. Many Linux desktops offer to store it in a password manager.

## Export/import public and private keys

The above base64-encoded export format is readable text and suitable for copy-pasting into GitHub web UI.
To transfer keys across machines, we should export and import them in raw binary format.

To export:

```bash
gpg --export $KEYID > public.key 
gpg --export-secret-key $KEYID > private.key
```

To ZIP them with encryption:

```bash
zip -e gpg.zip public.key private.key
```

To import on the target machine:

```bash
gpg --import public.key
gpg --import private.key
```

## Configure `git` to use key

To use a specific key:

```bash
git config --global user.signingkey $KEYID
```

Add the following to `~/.zshrc`:

```
export GPG_TTY=$(tty)
```

## Sign with `git`

To sign a commit:

```bash
git commit -S -m "YOUR_COMMIT_MESSAGE"
```

To sign a tag:

```bash
git tag -s $MYTAG
```

To sign all commits by default:

```bash
git config --global commit.gpgsign true
```

## GitHub Settings

To display verification status with commits and tags, the following
should be done in GitHub web UI.

- [Turn on display verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits).
- [Add key to GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).