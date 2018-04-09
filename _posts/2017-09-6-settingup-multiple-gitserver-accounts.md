---

title: Settingup Multiple Git Source Accounts
excerpt: "Configure Git to access multiple accounts like Bitbucket and Github"
tags: [Github, Git, Bitbucket, SSH_RSA]
categories: [ DevOps ]
comments: true
published: true
share: true
toc : true
toc_label : "Setting Multiple Git SSH Keys"
toc_icon : "cloud"
---

# Introduction

If you want to have private repositories at Bitbucket and public repositories at Github, then this is the guide for you.

This guide assumes:

* Git is set up for a single github user using https, not ssh
* There is only a known_hosts file in ~/.ssh/
* The computer is running Mac OSX 10.7 or greater
* You already have an account at Github and another at Bitbucket

# Setting up Multiple Git Login Accounts

We are going to setup Github as the main account and bitbucket as the secondary. This means the global git config will have Github Username and Email as default.


## Create SSH Keys  

* Generate the SSH Key pair for Github, provide `username` or email address ( private `username@users.noreply.github.com` or public)

	`ssh-keygen -t rsa -C "github <username>/<email>" `

	* Enter passphrase when prompted. If you see an option to save the passphrase in
your keychain, **do it** for an easier life.

	* Save the keys to  `~/.ssh/id_rsa`  and `~/.ssh/id_rsa.pub`

* Repeat for bitbucket:

	`ssh-keygen -t rsa -C "bitbucket <username>/<email>"`

	* Save bitbucket keys to `~/.ssh/id_rsa_bb` and `~/.ssh/id_rsa_bb.pub`


## Attach Keys  

* Copy the public key to clipboard

```shell
pbcopy < ~/.ssh/id_rsa.pub
pbcopy < ~/.ssh/id_rsa_bb.pub
```
Obviously run each `pbcopy` command **individually.**

* Login to git server or UI and add ssh public key:

	* Paste into text area, under ssh settings or repo deploy keys, in your github or bitbucket account.
	* Also give the ssh key a title like "Full_name@Laptop_name".

### Add in Github Cloud
![Add Public SSH Key in Github]({{ site.url }}{{ site.baseurl }}/images/git_keys/add_github_sshkey.gif)

### Add in BitBucket CLoud
![Add Public SSH Key in Bitbucket]({{ site.url }}{{ site.baseurl }}/images/git_keys/add_bitbucket_sshkey.gif)

## Create Config file  

Edit the SSH Config for your user home.

`vim ~/.ssh/config`

Create your git aliases like so:

```vim
# Github (default)
  Host github.com
  HostName github.com
  User <username>
  IdentityFile ~/.ssh/id_rsa

# Bitbucket (secondary)
  Host bitbucket.org
  HostName bitbucket.org
  User <username>
  IdentityFile ~/.ssh/id_rsa_bb
```  

## Add the identities to SSH

```shell
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_bb
```

Enter passphrase if prompted.

Check keys were added:

`ssh-add -l`  

## Check that repo recognizes keys

```shell
ssh -T gh
ssh -T bb
```  

# Using the dual setup
Now lets work on how to use the keys setup to checkout and commit to repositories in github and bitbucket.

## Setting Repositories

### Github (default)
Create a repo online called testgh(or one of your choosing), then in Terminal,
create your repository:

```shell
mkdir ~/testgh
cd ~/testgh
touch readme.md
git init
git add .
git commit -am "first commit"
git remote add origin git@gh:username/testgh.git
git push origin master
```

Add some text to readme on github.com, then:

```shell
git fetch origin master
git diff master origin/master
git merge origin/master
git push origin master
```

### Bitbucket (secondary)
Create a repo online called testbucket and then in Terminal:

```shell
mkdir ~/testbucket
cd ~/testbucket
touch readme.md
git init
```

*This being the secondary account, the username and email have to be
overwritten, using secondary account (bitbucket) values, at the repo level:*

```shell
git config user.name "Full Name"
git config user.email email_address
```

This must be done **once** for every *bitbucket* (or secondary) repo, it is not
needed for github (or primary) repos because **the global is used** in that scenario.
There may be a cleaner way to do this but right now it works okay.  

Just to be clear you do not need to change these values back afterwards because the
global values (which apply to all future repos created) will be set.  

```shell
git add .
git commit -am "first commit"
git remote add origin git@bb:username/testbucket.git
git push origin master
```  

Add some text to readme on bitbucket.org, then:  

```shell
git fetch origin master
git diff master origin/master
git merge origin/master
git push origin master
```

## Use case: Repo already exists on secondary remote repo (bitbucket)
So you have a repository that already exists and you want to want to `clone` it
but also you want to make sure when you `push` it, the correct user, the bitbucket
user pushes. Let's say the repo is called `booker`.

```shell
git clone git@bb:bb_username/booker.git
git config user.name "bitbucket user name"
git config user.email email_address
```

Remember, because we are using the secondary account, we have to over-ride the global configuration of username and email. The git clone command is using the ssh keyfile you set up earlier and is doing the transfer over ssh.

Now, change into the cloned directory and modify one of the files. Use `git status` to check the current state, then use `git add 'filename'` and `git commit -m "commit message"`.

Finally push your changes using `git push origin master`.

# Conclusion
This allows to multiple git logins (One per Git Server) but it does come with issues

## SSH-Agent in macOS Sierra

So it looks like Apple changed the behaviour of the ssh-agent in macOS Sierra. Now it does not autoload all the keys in the keychain that were added with ssh-add -K, so you must explicitly call ssh-add -A.

So I've just added this to my .zshrc
`ssh-add -A &> /dev/null`

Everything works great but for some odd reason, none of my passphrases or keys appear in the keychain. Not sure why, but it seems like ssh-agent does not store into the keychain anymore.

The solution is to add `AddKeysToAgent yes` and IdentityFile ~/.ssh/id_ed25519 (obviously replace the path with the path to your key) in ~/.ssh/config.

For example, your ~/.ssh/config might look like:
``` vim
# Github (default)
  Host github.com
  HostName github.com
  User <username>
  IdentityFile ~/.ssh/id_rsa
  UseKeychain yes
  AddKeysToAgent yes

# Bitbucket (secondary)
  Host bitbucket.org
  HostName bitbucket.org
  User <username>
  IdentityFile ~/.ssh/id_rsa_bb
  UseKeychain yes
  AddKeysToAgent yes
```

## Drawbacks

* The keys are prone to corrupted when present in the `~/.ssh/`
* Lot of commands and googling to fix when its broken like above with OS updates etc...
* Can only use one account per host or git server

There are better ways to access multiple git accounts using clients like  [Source Tree](https://www.sourcetreeapp.com/) and [Github Desktop](https://desktop.github.com/). While Source Tree allows mutiple Bitbucket/GitHub/Mercurial/Server accounts. Github only allow Github and Github Entriprise and only one each.
