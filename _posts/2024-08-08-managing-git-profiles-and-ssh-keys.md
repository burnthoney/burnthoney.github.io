---
layout: post
title: managing multiple git accounts with ssh keys
category:
- git
- ssh
date: 2024-08-08 12:56 +1000
---
# The problem 
Recently, I needed to create a separate GitHub account for university. To streamline my workflow, I wanted to use SSH 
keys, just like with my existing account. However, I quickly discovered that Git has difficulty determining which SSH 
key to use when both accounts are on the same platform (GitHub in this case). After some research, I found that the most 
prominent solution was to edit the .ssh/config file and create a unique host configuration for each account. Similar to
the example below.
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
Now, every time we clone a repository, we have to use a command like `git clone git@github.com-personal:username/repository`. 
This process is tedious because it requires editing the clone URL each time instead of simply running it in the terminal. 
Additionally, I usually sign my commits with my SSH key to ensure that others can verify the commits are genuinely from 
me and not from someone with unauthorized access to my account

# The Solution
This is where Git profiles come to the rescue. By using conditional imports with includeIf based on the directory, Git 
profiles allow you to load a specific configuration into your main config file. Even better, you can override the SSH 
command to use a designated SSH key—exactly what I’ll be doing.

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
Your output should now hopefully contain something similar to those lines. Make sure that the `user.email` matches
the email set for your profile, which should be your personal_email in this scenario.

## Troubleshooting Tips
- Verify your SSH setup by running `ssh -i ~/.ssh/<ssh_key> git@<provider>.com` (replace <provider> with GitHub, Bitbucket , etc.).
- Ensure your gitDir ends with a slash, e.g., `"gitDir:~/Development/<folder>/"`.
- Double-check that you're in the correct directory, such as /Development/personal for personal projects.
If issues persist, feel free to leave a comment below.
