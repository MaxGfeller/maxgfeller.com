---
layout: post
title: "Your own private npm Repository with sinopia"
date: 2014-02-26 18:26
comments: true
categories: 
---

Having a private repository for your modules is a fair use case. While working on your own projects that should not (yet) be open source and available for everyone you still should not have to miss the comfort of having a big amount of small packages to organize your large codebase.

npm itself offers the possibility to define different repositories to work with via the `--repository` flag. This however requires you to have your own CouchDB instance correctly set up, which [is possible](http://clock.co.uk/tech-blogs/how-to-create-a-private-npmjs-repository) but too complicated in my opinion.

## sinopia

[sinopia](https://github.com/rlidwka/sinopia) is a small package which solves exactly this use case. It does not require CouchDB and saves the packages directly on the file system. And the best part is that it is compatible with the standard npm client.

## Setting up sinopia

Setting up `sinopia` is fairly easy. You need node and npm installed on your host and then you can easily run `npm install -g sinopia`

And then in the directory where you want the packages to be stored simply run `sinopia` and it automatically starts a server.

It then creates an `admin` user and a random password for you which you need to configure in your npm client:

```
npm --registry=http://yourhost:4873 adduser
```

If you want to reuse your existing user simply add your username to the `config.yaml` file in the directory you run sinopia. The password hash can be generated with the following node function: `crypto.createHash('sha1').update('yourpass').digest('hex')`

To enable the server to listen for connections from outside simply add the following line to your config: `listen: 0.0.0.0:4873`

## alias

To have an additional command for accessing your private repository (rather than publish all new modules to it) simply create an alias:

```
alias pnpm="npm --repository=http://yourhost:4873 --always-auth=true --ca=null"
```

Now you can publish a module to your own server using `pnpm publish` and to `registry.npmjs.org` using `npm publish`.