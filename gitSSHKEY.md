---
title: gitSSHKEY
date: 2025-10-09 10:59:13
tags:
---

<h1 center>git clone被墙<h1>


* ssh -keygen -t ed25519 -C "158657247299@163.com"
* cat ~/.ssh/id_ed25519.pub
* copy 2_step generate 
* ssh -T git@guthub.com(hello,* * *, you have successully  * * *)
* git clone git@github.com:microsoft/mattergen.git 即可




* 打开 [GitHub Settings](https://github.com/settings/profile)。
* 点击 **SSH and GPG keys**。
* 点击 **New SSH key** 按钮。
* 在 **Title** 中输入一个描述（如 "My SSH Key"）。
* 在 **Key** 中粘贴你刚才复制的公钥内容。
* 点击 **Add SSH key** 按钮。
