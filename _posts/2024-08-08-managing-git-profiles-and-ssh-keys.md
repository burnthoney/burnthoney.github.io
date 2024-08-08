---
layout: post
title: managing multiple git accounts with ssh keys
category:
- git
- ssh
date: 2024-08-08 9:56 +1000
---
# The problem 
Managing multiple git accounts can be a pain when using ssh key's especially when using the same git provider. Luckily there is an easy fix to this problem.

## Step 1 Create Your SSH keys
```bash
$ ssh-keygen -t ed25519 -C "personal@example.com"
$ ssh-keygen -t ed25519 -C "university@example.com"
```

## Step 2 Setup directory based git profiles
Edit your `.gitconfig` file to something like this
```toml
# Global settings go here
[gpg]
format = "ssh"
[gpg "ssh"]
program = "/Applications/1Password.app/Contents/MacOS/op-ssh-sign"
[commit]
gpgsign = true
[push]
autoSetupRemote = true
[user]
name = "your_name"

# Based on the directory we include our gitconfig
[includeIf "gitDir:~/Development/personal/"]
path = "~/.gitconfig.personal"
[includeIf "gitDir:~/Development/university/"]
path = "~/.gitconfig.university"
```
{: file='~/.gitconfig'}
> You don't need to set up GPG but since I use it to sign my commits, and it's also a great example of how you benefit from using git profiles I will keep it here
{: .prompt-info }

## Step 3 Configure you git profiles
```toml
[user]
email = "your_personal_email"
signingkey = "<your signing key"

[core]
sshCommand = "ssh -i ~/.ssh/<sshkey>"
```
{: file='~/.gitconfig.personal'}
```toml
[user]
email = "your_university_email"
signingkey = "<your signing key>"

[core]
sshCommand = "ssh -i ~/.ssh/<sshkey>"
```
{: file='~/.gitconfig.university'}
> If you're using 1 password like I am to manage your ssh keys. Download your public key and link to it instead. 
{: .prompt-warning }

## Step 4 Verify your config
```bash
~> cd Development/personal
~/Development/personal> git config -l
user.name=your_name
user.email=your_personal_email
user.signingkey=your_signing_key
```
Your output should now contain something like this.
