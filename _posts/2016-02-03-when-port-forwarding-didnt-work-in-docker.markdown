---
layout: post
title: When port forwarding didn't seem to work with Docker
date: 2016-02-03
comments: true
---


I just had an issue with Docker which took up a couple of hours, and
which, like most such problems, was just plain stupidity on my
part. And being new to Docker didn't help either.

I had a spray server app that I needed to run inside a docker
container. Don't ask why. The spray app ran on localhost at
port 9090. Now, I just wanted to make the service available from
outside the container. And it's pretty simple. I built the image, and
then just ran

```

sudo docker run -p 9090:9090 my-spray-app

```

This should work. And it does work, but I'll come to that later. So,
after running this, when I tried to access the service via
localhost:9090, all I got was "Remotely Closed" errors.

I thought maybe I had made some mistake, which I had, to be
honest. So, I looked around, Google and stuff, and I was sure that the
service was running inside the container after connecting to the
docker in an interactive terminal and checking stuff out. Nope, no
issues there. The service was running fine.

Then, there was
[this issue on github](https://github.com/docker/docker/issues/13914),
which was exactly what was happening with me too. Using `--net=host`
worked, but nothing else did. And just to be clear, this is probably
not an issue. The Docker developers are being very nice in trying to
address the issue, but it seems like those people are making the same
mistake I was.

Then, there was
[this issue](https://github.com/docker/docker/issues/2174). I could
see the same kind of output in `netstat`, but that clearly is not an
issue. I am at least a little familiar with how networking works in
linux.

The issue is that the app that is running in the container is bound on
localhost. Which is not accessible from outside at all. As
[this answer by a docker staff member](https://forums.docker.com/t/expose-localhost-port-in-container/3281/2)
says,

> It isn't really possible to map the container's localhost to the
> outside. The process in the container must bind to the container's
> eth0

So, that was all. And like the idiot that I am, this was plainly
visible and I still didn't think of it. When I saw the output of
`docker ps`, the `port` column clearly said that the binding was to
the "0.0.0.0:9090", and not "localhost:9090". I saw it, but I didn't
observe. That's what happens when you don't listen to Sherlock Holmes.

The only thing I needed to do was set the host of the spray app to
"0.0.0.0". I am sure there are other ways of doing this, but this was
all I needed for the time being.

In my defense... Ah, forget it, it was just stupid. Putting this up in
case it helps some fellow morons.
