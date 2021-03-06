# OpenTTD in K8s - Helper Functions
__Brought to you by /r/openttd__

This project contains the following Go programs:

* _sidecar_ is a sidecar for extracting bans on a regular basis from a running copy of OpenTTD.
* _banread_ is a small script that (optionally) runs during the startup of the main OpenTTD image to merge bans from the file generated by the sidecar into `openttd.cfg`.
* _bananasync_ is a useful pre-run script that scrapes the config of your server and ensures that any defined NewGRFs are available
* _preinit_ combines all of the above into an executable you can run before your main container.

## How sidecar Works

This container will regularly poll the openttd.cfg mounted at `/config/openttd.cfg` and write any bans it sees to a flat file in `/config/bans.txt`.

## Using the Sidecar Container
```
docker run -d -v /home/{username}/.openttd:/config:rw redditopenttd/bans-sidecar:latest
```

Please ensure that the directory mounted at /config is the same one that is mounted in your main OpenTTD container (the script polls /config/openttd.cfg for data).

## Using the Preinit container
The Preinit container combines bananasync, banread, and a read-only to read-write config move into one step.

See the [manifest in the main repository](https://github.com/ropenttd/docker_openttd/blob/master/k8s/statefulset.yaml) for example usage.

## Using banread

_banread_ takes the path to your openttd.cfg and a flat file containing a list of IP addresses, and merges that list into the given openttd.cfg.

It will merge duplicates automatically if a ban exists in bans.txt but not in your config.

## Using bananasync

_bananasync_ parses a given openttd.cfg for defined NewGRFs, and checks the content download directory for their existence.
If any of them are not available, it downloads them from BaNaNaS and extracts them to the right place, ready for usage by the game.

Please don't abuse this tool: it asynchronously pulls from the OpenTTD CDN, and they only have limited bandwidth, so make sure you cache your downloads in persistent storage!