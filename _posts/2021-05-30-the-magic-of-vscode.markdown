---
title: "The Magic of VSCode"
date: 2021-05-30
categories: vscode
tags:
  - development
  - vscode
  - containers
  - docker
  - efficiency
---

Like so many, I've spent many years building stuff in a million different ways - IDEs, at a raw CLI - on Macs, Linux, and Windows - and so on. It's been a huge struggle, honestly, to keep up with dealing with so many things on a development machine.

Finally, I just got really tired of it, so I started doing some digging into VSCode and the work they've done with Docker for containerized development environments.

On top of that, because I like things to be difficult - I moved to a 16GB Mac Mini M1 some time back. So trying to get everything working on ARM64 for all these different personal+work projects of mine - not fun.

So I did some reading after learning that VSCode's Remote extension family had a Docker implementation. And _boy_, I realized I was on to something. Something fantastic, something magical - something to keep my OCD in check.

# Getting Started

Take this blog itself as an example. It uses Jekyll for themed Github Pages - which has a dependency on Ruby (ew - sorry Ruby folks). There was no way in hell I was going to deal with that on my clean and pristine Big Sur installation.

Two files later - and I'm off to the promised land.

**Note**: If you want to see this for yourself, it's in the public repository that powers this site: [`in2rd/in2rd.github.io`](https://github.com/in2rd/in2rd.github.io/)

## The Bare Bones

Every one of these Remote Docker projects starts in the same place: `.devcontainer/`. Whether you set it up yourself, as I've found myself prone to doing, or use Microsoft's guided walkthrough and default settings - it's all going to go (and stay) here.

With this example, it's a simple one-container custom project. So in order to do this, we'll need the following:

- Docker daemon of some kind:
  - Docker Desktop for [Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
  - Docker Desktop for [Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
  - [Docker CE install](https://docs.docker.com/engine/install/) on a Linux host.
- Visual Studio Code:
  - Note here, if you're following along with what I have make sure you're updated. I use the latest configuration settings, which aren't backwards compatible.
  - Also, you can use the Insider Build as well, which I had to in the beginning for Apple Silicon compatibility. Just be aware if you do end up using, and move back to GA VSCode, you'll have to start from scratch. Their settings are handled differently.
- [VSCode Remote - Containers Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

And really, that's it. Now, let's set up our project by creating an empty folder in your repo called `.devcontainer`, and then the two files you need below.

## The Container Image

Personally, I like to create and use my own `Dockerfile`. Having specific control over the base image, the configuration, environment, and everything else inside the container is nice. You don't have to do that though. If you want to start with the preconfigured Microsoft options, that's perfectly OK too. If you want the easy route, just launch as a container, and [follow the steps it provides](https://code.visualstudio.com/docs/remote/containers-tutorial). You can then skip these next two items.

If you're a control freak like me, then here's how we do it the hard way.

![Dockerfile](/assets/images/posts/the-magic-of-vscode/vscode-dockerfile.png)

Few things of note if you go my way:

- Make sure you install things like `git`, `zsh` if you want it, `man`, `less`, and basic system utils. This will greatly depend on what base image you pick. But since you'll operate inside a containerized terminal session - you'll need these tools.
- You'll have to create the `vscode` user and group settings yourself.
- Locale can also be problematic. In my case, I go generic, but adjust this to your... locale. `;)`

## Setting up your Dev Container

There's a lot going on here, and I'm not going to try and explain it all - [Microsoft does better than I do](https://code.visualstudio.com/docs/remote/devcontainerjson-reference). But the bottom line is you need a JSON file with a few basic things to get going:

- `name` - This is what shows up in the bottom left once you go containerized. I tend to either call it the repo name or something similar. In the case of this blog, I call it `github-page` so I can remember what it is with 400 VSCode windows open.
- `build.dockerfile` - Just the name of the `Dockerfile` you chose. Since you can also do a Docker Compose version of this, which I'll do a blog about later, you might have many in here. For a simple project - just `KISS`.
- `remoteUser` - As I'll mention below, we don't want to run as `root`. So if you run as any other user, just need to let VSCode know who it is. Remember, this is _in_ the container.
- `extensions` - An array of any extensions [by namespace!] that you want to include in this specific containerized project. Some extensions work at the host, some require you to be installed in the container - YMMV. In this regard, I just include whatever I need here and I'll know from that point it's consistent behavior.
- `mounts` - You'll see me mount a few things here from my home folder on the host. The _most_ important one, however, is your `.ssh` folder, as you'll need to do ops w/ `git`. VSCode _can_ mount this for you (as well as `.gitconfig`), but I've had issues with that. And explicit behaviors never hurt so I just do it myself.
- `postCreateCommand` - There's a section below on this. Just keep it in mind for now.
- `settings` - If you want to define different settings for a project, or something specific to the language, you can do it here. It's similar to the `user` vs. `workspace` settings that VSCode already provides, but just scoped specifically to this containerized project.

![.devcontainer.json](/assets/images/posts/the-magic-of-vscode/vscode-devcontainer-json.png)

## Getting Inside

Once you have your `Dockerfile` and `devcontainer.json` ready, we're ready to go:

1. If you've installed the Remote Container extension I linked above, you'll see two little crossing arrows in the bottom left of your VSCode window. Click that.

![VSCode Remote Options](/assets/images/posts/the-magic-of-vscode/vscode-remote-options.png)

2. You'll be presented with a dialog of options, such as the image above. In this case, I am being asked to reopen as a container, because I had the project open that way already. For a first-timer, you may just see an option to open as a container. That's what you want.

3. Well - that's it! Once you open it, your VSCode window will relaunch, and you'll start the build process.

### Build Process

Just like everything else related to Docker, we have to build the image associated with the project (`.devcontainer/Dockerfile`). Once you've decided to reopen as a container, the logs will be accessible (\*\*Starting Dev Container (show log)`) in the bottom of the window. Click those, and you'll be able to follow along with the build process. Depending on your setup, and what you're doing in the `Dockerfile`, this can take anywhere from seconds to minutes. In this case, a clean rebuild for this project takes a minute or so, and then I'm good.

![Docker Build](/assets/images/posts/the-magic-of-vscode/vscode-devcontainer-build.png)

If I decide to change the Dockerfile, or the `devcontainer.json` file, VSCode will notify me that the settings have changed, and give me the option to rebuild the project. I highly suggest keeping an eye out for this popup and rebuilding when you're notified, as that will update your development container.

### Before Launch

Once the Docker build process is complete, and we have a running container, you can tell VSCode to execute a command after create (`postCreateCommand` in `.devcontainer.json`). In this specific project, I tell it to exec `bundle install` to get all of the dependencies Jekyll needs from the `Gemfile` to work on the blog.

![Post Create](/assets/images/posts/the-magic-of-vscode/vscode-devcontainer-postcreate.png)

### And We're Done!

If you look at the image below, you'll see I've spawned an integrated Terminal, and am now running with my `vscode` user under `zsh`, with `oh-my-zsh` installed and ready to go!

Our current working directly has been automatically associated as the terminal as the entry point, and is mapped to our host repository.

From here, everything works just like you'd expect:

- `git` and its commands, including the base `.gitconfig`.
- `ssh` and relevant key-based permissions, based on the associated host volume mount.
- All of the extensions are installed for this project, and project settings in VSCode are ready to use.

![Containerized Terminal](/assets/images/posts/the-magic-of-vscode/vscode-containerized-terminal.png)

Most importantly, and a very important security hygiene feature with containers - we're not running as `root`. We're running as the `vscode` user with `UID:1000` and `GID:1000` - which should map to the vast majority of everyone's host user.

![Non-Root User](/assets/images/posts/the-magic-of-vscode/vscode-devcontainer-nonroot.png)

## But My Extensions!!!

First thing I got asked about when I showed it to some other engineers, and I'll never forget:

`Engineer #1`

> "I like your font and your theme, is that included?"

`Engineer #2`

> "I hate your font and your theme - I'm not using this."

Valid points too. I am really picky about fonts and colors, especially considering I stare at my screens 12 to 16 hours a day.

Here's the good news - You can _override_ your host VSCode Extensions, and install ones you need for your project _just_ in the container itself!

**Awesome**. Now I can:

- Have my theme and font settings that _I_ like to use every day.
- Have the big extensions I need and repeatedly use at my host running VSCode and never think about those again.
- Install extensions that are language-specific or functionally-specific to whatever my project does.

Also, big shout out to Monokai Pro for my theme (_if you like it, **of course**_). It's saved my eyes since I found it. If you do like it, [drop on over to their site](https://monokai.pro) and drop them ~$20. Well worth it, IMO.

# The Big Why?

There's a lot of reasons you might be interested in this. If you've read this far, it's probably pretty easy to figure out I work in a lot of languages, a lot of projects, both for work and on my personal time.

Add Apple Silicon support, horrible OCD, and a former military sentiment of "[dress-right-dress](https://thedrillmaster.org/2014/03/18/dress-right-dress/)", and it's pretty easy to see how I got to this point.

So personal maintenance for each of your projects gets quite a bit better by doing this.

I think the real winner here, however, is corporate environments. Ultimately, I'm stuck working with a medium-sized engineering organization (and growing faster every day, it seems like) - and we all know how developers can be, myself included:

1. "I like this tool, not that one"
2. "I don't know how to use that tool, and I don't want to learn"
3. _"Worked on my machine - don't know why it doesn't work now"_ (my personal favorite)
4. "I like to do things my way"

... and so on. There's a million reasons why. So the support tail as a principal sucks - you get stuck debugging basically everything but real business logic and problems most of the day. And you also get this, because no one can agree:

![XKCD Says it Best](https://imgs.xkcd.com/comics/standards.png)

So once I started rolling this out in places here and there at work, it naturally started to turn on some light bulbs, especially with the more junior engineers. _Everything just worked_.

So - if you're like me and you hate having to deal with a mess every time you do something, give this a shot. Might just make your life a little bit easier.

`:thumbsup:`

# RTFM

- VSCode Remote - Containers Extension - [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

- Microsoft Dev Container Documentation - [https://code.visualstudio.com/docs/remote/devcontainerjson-reference](https://code.visualstudio.com/docs/remote/devcontainerjson-reference)
- Microsoft Remote Development in Containers Walkthrough - [https://code.visualstudio.com/docs/remote/containers-tutorial](https://code.visualstudio.com/docs/remote/containers-tutorial)
