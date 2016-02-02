# HashiCorp Base Images

This repository contains scripts to build base images for HashiCorp products packaged in Docker. Our goal with these is to make the most minimal image possible to reduce the surface area for security vulnerabilities, but to balance that with user experience and usefulness. Consul, for example allows for the configuration of health check scripts which often need access to various utilities. We've chosen BusyBox as the base for this purpose.

There's a base BusyBox image (`base-busybox`) that gets root certificates installed enough for Go programs to get to hashicorp.com properties.

We also build and sign some custom binaries, but they aren't included in the base image:
* [dumb-init](https://github.com/Yelp/dumb-init), which makes it easy to configure child process reaping and gives us signal forwarding to all processes running under it
* [gosu](https://github.com/tianon/gosu), which makes it less of a pain to switch to other users without introducing a `su` or `sudo` intermediate process
* [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html), used by Consul to avoid running as root just to get port 53

There are app-specific (`app-base-*`) images that pull in what they need from above. This isn't too much stuff, so we might want to get rid of that layer and just pull those into the base, but it would be weird to ship dnsmasq with Vault, for example, so this lets us be selective for each product.

Finally, we have builder (`builder-*`) images that are hermetic build environments for the binaries. These aren't intended for any other use other than as transitory containers for building, which are then thrown away.

## Building the Images

There's a top-level `Makefile` that does all the building. If you are developing locally and don't have the HashiCorp signing key, you can run `NOSIGN=1 make` to avoid the signing step.
