---
author: Ryan Jordan
pubDatetime: 2015-12-31T15:03:00Z
title: Ghost & Docker
draft: false
tags:
  - ghost
  - docker
description: Since I have had this blog up and running, I've been using ghost with a [Digital Ocean](https://www.digitalocean.com/) virtual machine. If you're looking to go that route, I have written a quick tutorial on that [here](http://mycodingblog.com/up-and-running-with-ghost-and-digital-ocean/). I have not had any issues, but since I have been using [Docker](https://www.docker.com/) more and more, I decided it was finally time to throw my blog into a container. I have to admit, it took a few days and a bit of research. Maybe I'm still new at this whole coding thing, I know I have a ton to learn, but seriously?
---

Since I have had this blog up and running, I've been using ghost with a [Digital Ocean](https://www.digitalocean.com/) virtual machine. If you're looking to go that route, I have written a quick tutorial on that [here](http://mycodingblog.com/up-and-running-with-ghost-and-digital-ocean/). I have not had any issues, but since I have been using [Docker](https://www.docker.com/) more and more, I decided it was finally time to throw my blog into a container. I have to admit, it took a few days and a bit of research. Maybe I'm still new at this whole coding thing, I know I have a ton to learn, but seriously?

I ran into a few hiccups, but I did finally find a solution. I am using the [Official Docker Ghost Image](https://hub.docker.com/_/ghost/). It briefly describes how to get it up and running. Well, almost... The commands given will end up giving you an error:

<!--more-->

```none
ERROR: Unable to access Ghost's content path:
	EACCES: permission denied, open '/usr/src/ghost/content/apps/ebc538ae3ae45772'

Check that the content path exists and file system permissions are correct.
Help and documentation can be found at http://support.ghost.org.

npm info ghost@0.7.2 Failed to exec start script
npm ERR! Linux 3.10.94-1.el6.elrepo.x86_64
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v4.2.3
npm ERR! npm  v2.14.7
npm ERR! code ELIFECYCLE
npm ERR! ghost@0.7.2 start: `node index`
npm ERR! Exit status 235
npm ERR!
npm ERR! Failed at the ghost@0.7.2 start script 'node index'.
npm ERR! This is most likely a problem with the ghost package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     node index
npm ERR! You can get their info via:
npm ERR!     npm owner ls ghost
npm ERR! There is likely additional logging output above.
npm ERR! Linux 3.10.94-1.el6.elrepo.x86_64
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v4.2.3
npm ERR! npm  v2.14.7
npm ERR! path npm-debug.log.7da2e9feb6af80ea1c45d89105a542fd
npm ERR! code EACCES
npm ERR! errno -13
npm ERR! syscall open

npm ERR! Error: EACCES: permission denied, open 'npm-debug.log.7da2e9feb6af80ea1c45d89105a542fd'
npm ERR!     at Error (native)
npm ERR!  { [Error: EACCES: permission denied, open 'npm-debug.log.7da2e9feb6af80ea1c45d89105a542fd']
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'open',
npm ERR!   path: 'npm-debug.log.7da2e9feb6af80ea1c45d89105a542fd' }
npm ERR!
npm ERR! Please try running this command again as root/Administrator.

npm ERR! Please include the following file with any support request:
npm ERR!     /usr/src/ghost/npm-debug.log
```

Reading the comments didn't help me much, but did get me pointed in the right direction. It appears that ghost has permission issues when running in production. However, [@snnd](https://github.com/snnd) posted this little [gem](https://github.com/snnd/docker-ghost-bundle).

So I did what he said and it didn't work. I had to make some more changes to the <code class="language-none">start.sh</code> file. In all fairness, it is most likely my fault, but I was able to get it to work.

## Let's Make a Ghost Container!

Be sure to copy your <code class="language-none">content</code> folder if you already have a ghost blog up and running. If you don't, you can add these [folders](https://github.com/TryGhost/Ghost/tree/master/content) or use the ones from the [repo](https://github.com/snnd/docker-ghost-bundle) mentioned earlier and place them where ever you want your ghost files to live on your host. You will also need a config file. I did a <code class="language-git">git clone https://github.com/snnd/docker-ghost-bundle.git</code> and changed the directory name to <code class="language-none">mycodingblog/</code>.

I also added all of my data, images and theme files to the appropriate directories. Next, I edited the <code class="language-none">mycodingblog/config/config.js</code> file:

```javascript
var path = require("path"),
  config;

config = {
  production: {
    url: "http://mycodingblog.com/",
    mail: {},
    database: {
      client: "sqlite3",
      connection: {
        filename: path.join("/usr/src/ghost/content/data/ghost.db"),
      },
      debug: true,
    },

    server: {
      host: "0.0.0.0",
      port: "2368",
    },
  },
};

module.exports = config;
```

Then I edited the <code class="language-none">start.sh</code> file:

```bash
#!/bin/bash

docker run -d --name mycodingblog \
	-p 8080:2368 \
	-e VIRTUAL_HOST=mycodingblog.com \
	-e NODE_ENV=production \
	-v /mycodingblog/content/themes:/usr/src/ghost/content/themes \
	-v /mycodingblog/content/apps:/usr/src/ghost/content/apps \
	-v /mycodingblog/content/images:/usr/src/ghost/content/images \
	-v /mycodingblog/content/data:/usr/src/ghost/content/data \
	-v /mycodingblog/config:/var/lib/ghost \
	ghost
```

Then I typed <code class="language-none">./start.sh</code>. Of course this was after playing around with it for a few hours. Make sure to use the <code class="language-none">-d</code> flag to run in detached mode. It started up and told me to point my browser to <code class="language-none">http://mycodingblog.com</code>. So now you should have a ghost container up and running.
