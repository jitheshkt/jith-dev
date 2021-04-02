---
title: Enable SSL on WSL2 Apache Windows 10
date: "2020-08-19T22:40:32.169Z"
template: "post"
draft: false
slug: "enable-ssl-on-wsl2-windows10"
category: "Development"
tags:
  - "Development"
  - "Environment"
description: "When first time I approached this particular situation of enabling SSL on a WSL2 (Windows Subsystem for Linux) installation inside my Windows 10, I thought it will be a in and out job. But to my surprise it took me almost an hour and more than three different articles found on Internet (which are pretty much outdated) to make it working. So that’s the moment I decided to write this one."
socialImage: "/media/42-line-bible.jpg"
---

Before we begin I am assuming that you already have WSL2, Apache/Ngnix configured.

### What we want?

>http://localhost currently served from WSL2 instance, instead of Windows. We are going to enable SSL on this domain, which will become https://localhost

In theory SSL certificates has to be generated on the target server and served from it. So in our case, it should be generated on the WSL instance rather than Windows instance. But, this is the first miss conception of SSL on WSL.

![realizing](https://media1.giphy.com/media/dk0dGQrMiVZZ7rc1Du/giphy.gif?cid=ecf05e47muq23586nj9hgsr9m8wbc3w3o2d08lzpcuiaa0q5&rid=giphy.gif)

We’re actually going to generate certificates on Windows OS and we just link it to the WSL Apache/Ngnix configuration.

We’re going to use the popular tool ```mkcert``` to generate our certificates.

So on WSL2 as root type:

```bash
wget https://github.com/FiloSottile/mkcert/releases/download/v1.4.2/mkcert-v1.4.2-linux-amd64
mv mkcert-v1.4.2-linux-amd64 mkcert
chmod +x mkcert
cp mkcert /usr/local/bin/
```

Ok, now we have `mkcert` installed on WSL2. Lets do the same on Windows.

First install Chocolatey on Windows if you haven’t have it already. Chocolatey is a package manger for Windows and you can get it from here: https://chocolatey.org/install

Once you have Chocolatey, fire up the power-shell with administrative privileges and execute below commands:

```bash
choco install mkcert
mkcert -install
setx CAROOT “$(mkcert -CAROOT)”; If ($Env:WSLENV -notlike “*CAROOT*”) { setx WSLENV “CAROOT/up:$Env:WSLENV” }
```

With the last command, we will set the `CAROOT` environment variable on the WSL2 side to point to the Windows `CAROOT`, so our Windows browser can trust sites running in WSL2.

Back on WSL2, you can verify the constant by typing:

```bash
mkcert -CAROOT
```

You will see a result something like this:

```bash
/mnt/c/Users/Jithesh/AppData/Local/mkcert
```

Now, back on windows power-shell type:

```bash
mkcert localhost 127.0.0.1 ::1 0.0.0.0
```

So, now the certificates will be in the `CAROOT` directory.
Now all left to do is update your apache or ngnix configuration, since I am on apache I am adding my steps:

```bash
vim /etc/apache2/sites-available/default-ssl.conf
```

On the config file,

```bash
SSLCertificateFile      /mnt/c/Users/Jithesh/AppData/Local/mkcert/localhost+3.pem
SSLCertificateKeyFile   /mnt/c/Users/Jithesh/AppData/Local/mkcert/localhost+3-key.pem
```

Enable SSL if you’ve not:

```bash
a2enmod ssl
a2ensite default-ssl.conf
```

and, restart apache:

```bash
service apache2 reload
service apache2 restart
```

That’s it. Now you have a working SSL on your localhost with WSL2.

Happy coding!