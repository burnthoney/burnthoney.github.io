---
layout: post
title: managing multiple git accounts with ssh keys
category:
- git
- ssh
date: 2024-08-08 12:56 +1000
---
# The problem 
Recently I've had to make another GitHub account for university and to make things easier I wanted to make use of ssh key's like I do with my current account. 
However, it turns out when you use the same provider for git (GitHub in my case) git has trouble deciding what ssh key to use.
After scouring the web the first method that was suggested to me was to edit my .ssh config and create a host for each account e.g
```text
Host github.com-personal
  User git
  HostName github.com
  IdentityFile ~/.ssh/<personal_ssh_key>

Host github.com-university
  User git
  HostName github.com
  IdentityFile ~/.ssh/<university_ssh_key>
```
{: file='~/.ssh/config' }
But now everytime we clone our repository we have to do `git clone git@github.com-personal:username/repository` which seems pretty tedious especially 
since it means everytime we copy the clone url we have to edit it instead of just running it in your terminal of choice.
I also normally sign my commits with my ssh key so other people can be sure that the commits came from me and not someone who got access to my account. 

# The Solution
Here comes git profile to the rescue. By making use of conditional imports via `includeif` based on the directory. Git profiles let the user load a specific config into the main config. 
To make things even better you can override the ssh command to use a specific ssh key which is exactly what I'm going to be doing.

## Step: 1 Create Your SSH keys
```bash
$ ssh-keygen -t ed25519 -C "personal@example.com"
$ ssh-keygen -t ed25519 -C "university@example.com"
```

## Step: 2 Setup directory based git profiles
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
> Make sure your gitDir path ends with a `/` otherwise your config will not load 
{: .prompt-info }

## Step 3: Configure you git profiles
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
> If you're using 1 password like I am to manage your ssh keys. Download your public key and point your command to use that instead.
{: .prompt-warning }

## Step 4: Verify your config
```bash
~> cd Development/personal
~/Development/personal> git config -l
user.name=your_name
user.email=your_personal_email
user.signingkey=your_signing_key
```
Your output should now hopefully contain those lines.

## Troubleshooting Tips
- Verify whether your ssh works by doing `ssh -i ~/.ssh/<ssh_key> git@<provider>.com` where provider is something like Github or bitbucket
- Make sure your gitDir ends with a slash e.g `"gitDir:~/Development/<folder>/"`
- Verify whether you are in the expected directory e.g /Development/personal for personal projects
If you still have problem feel free to leave a comment below.
