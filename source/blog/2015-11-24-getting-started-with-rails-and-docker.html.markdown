---
title: Getting Started with Rails and Docker in OS X 10.11
tags:
  - Docker
  - Rails
  - Quickstart
  - OS X
  - El Capitan
---

Docker has a few notable benefits over both installing and running dev services locally on
your host machine and using a self-contained VM. This post is a quickstart and not a
Docker introduction, so I'll skip the pros and cons of that ([but here's a pretty good
post on the relative strengths of Docker and Vagrant][1])

That said, quickly getting up to speed on how best to setup Docker in OS X can be a
daunting task.  There are a lot of different tools and methods of doing things, and the
ecosystem is still evolving very rapidly. This post goes through how I setup my local dev
environment for a straightforward Rails site using Postgres and Redis.

### Homebrew
First up, if you're on OS X and not using [Homebrew](http://brew.sh), you're
missing out on a lot. Homebrew is the real successor to MacPorts, and a great package
manager in its own right. So go ahead and install it. Then install these packages:

```bash
$ brew update
$ brew install Caskroom/cask/virtualbox docker \
	docker-compose docker-machine
```


Yes, VirtualBox. Above is an article about Docker vs Vagrant (which uses VirtualBox as its
hypervisor), but in actuality you're going to be running Docker in a VM on OS X no matter
what. Docker uses a Linux kernel feature called
[cgroups](https://en.wikipedia.org/wiki/Cgroups) to drive its containers which OS X (being
a BSD derivative) doesn't support. It's possible that Docker will natively support OS X at
some point, but it doesn't look to be a priority right now. The Docker CLI tools will be
communicating with the Docker daemon running in a virtual machine over a TCP socket.

Also of note is that we're installing VirtualBox using Homebrew instead of from the
installer. I like doing this because I almost never open the VirtualBox app at all, and so
miss the upgrade checks and download prompts. Installing it from the package manager
ensures I can keep it up-to-date more easily.

### Docker Machine and Launchd
[Docker Machine](https://github.com/docker/machine) is the
successor to the old way of running Docker on OS X: [boot2docker](http://boot2docker.io/).
At least, it is for our purposes; Docker Machine [does much more than
this](https://docs.docker.com/machine/) and in fact uses the boot2docker image on OS X
behind-the-scenes.

First, you'll need to create a new VM. You can replace `dev` with whatever you want your
machine to be called:

```bash
$ docker-machine create -d virtualbox dev
```

I like to have that VM start when I login to my laptop so that I can put the shell init
invocation in my shell rc without wrapping it in a bunch of cumbersome checks. The VM is
very lightweight, starts up pretty much instantly, and hasn't noticably affected my
startup time, but you can skip this part if you don't want it to auto-start.

Create a new [launch agent][2] at `~/Library/LaunchAgents/local.docker-machine.dev.plist`
(again replacing `dev` with your custom name if necessary):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Label</key>
		<string>local.docker_machine.dev</string>
		<key>ProgramArguments</key>
		<array>
			<string>
              /usr/local/bin/docker-machine-wrapper.sh
            </string>
			<!-- Change to your machine name -->
			<string>dev</string>
		</array>
		<key>KeepAlive</key>
		<true/>
		<key>RunAtLoad</key>
		<true/>
		<key>EnvironmentVariables</key>
		<dict>
			<key>PATH</key>
			<string>
              /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
            </string>
		</dict>
	</dict>
</plist>
```

This will tell `launchd` (OS X's init program) to run that wrapper script at load, passing
it the name of your machine, and keeping it alive if it dies. Here's the wrapper script:

```bash
#!/bin/bash

if [ -z "$1" ]; then
	echo "Must pass the name of a machine to this script"
	exit 1
fi

stop_docker_machine() {
	docker-machine stop $1
}

trap "stop_docker_machine $1" INT TERM

docker-machine start $1

while true; do
	[[ "$(docker-machine status $1)" == "Running" ]] || break
	sleep 1
done
```

Put this script in `/usr/local/bin` and make sure it's executable.

The reason we need this is that `docker-machine` does not have a "foreground" option: the
way we are using it is essentially a wrapper around `VBoxManage`, so `launchd` has no
process to monitor.

Tell `launchd` to load and start your process with:

```bash
$ launchctl load ~/Library/LaunchAgents/local.docker_machine.dev.plist
```

Finally, add this to your `.bashrc` or equivalent:

```bash
eval "$(docker-machine env dev)"
```

Open a new shell or source your rc file, and running `docker version` should successfully
return both client and server information.

### Docker Compose and Rails
Now that we have a running VM with Docker on it and are able to control it from our host,
we can make our new Rails project. Assuming you already have Rails installed, just make a
new project, specifying that we want to use pgsql instead of sqlite:

```bash
$ rails new docker_fun -d postgresql
$ cd docker_fun
$ bundle install
```

After bundler is done, we need to make a file to configure [Docker
Compose](https://docs.docker.com/compose/). Formerly known as Fig, Docker compose is a
tool for automating the sometimes-complex commands required to create, start, link, and
otherwise manage docker containers. You can create your own images from Dockerfiles easily
with this, but we're just going to use a couple stock Postgres and Redis images. The
config file is simple YAML (and the link above has more details):

```yaml
---
db:
  image: "postgres:9.4"
  environment:
    POSTGRES_USER: docker_fun
    POSTGRESS_PASSWORD: password
  ports:
    - "5432:5432"

redis:
  image: redis
  # --apendonly so that our data persists across container stops
  command: redis-server --appendonly yes
  ports:
    - "6379:6379"
```

Put this in a file called `docker-compose.yml` in the Rails app root, and then run
`docker-compose up -d`. You should see Docker download all the images it needs, then
create new containers from those images and start them in the background. You can omit the
`-d` if you want to see it for yourself.

After it's done, run `docker-compose ps` to see the images you've created and what they're
doing. Specifically, you can see in the "Ports" heading that we've forwarded the normal
pgsql and redis ports from the containers to the "host". In this case the host is our VM,
and so to connect to our new services we need to know our VM's IP.

To configure Rails for this, I use a gem called
[dotenv](https://github.com/bkeepers/dotenv). This reads a `.env` file in the Rails root
and sets environment variables from it. So add this to your Gemfile and re-bundle:

```ruby
gem "dotenv-rails"
```

Then create a `.env` file with this:

```bash
DOCKER_IP="$(docker-machine ip dev)"
DATABASE_URL="postgres://docker_fun:password@$DOCKER_IP:5432/docker_fun"
REDIS_URL="redis://$DOCKER_IP:6379"
```

And in your `config/database.yml` file, change the `development` environment config so it
looks like this:

```yaml
development:
  <<: *default
  url: <%= ENV['DATABASE_URL'] %>
```

Nothing else should be needed for Redis, as the redis gem already checks for a `REDIS_URL`
environment variable when you instantiate a new client.

That's it! You should now be able to generate a new model, and run `bin/rake db:setup` on
your Docker database.

When you're done working and ready to shut down, just run `docker-compose stop` from your
rails root. This will halt the containers while keeping the data in them intact.

[1]: https://medium.com/@_marcos_otero/docker-vs-vagrant-582135beb623#.725omahj9
[2]: https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html

