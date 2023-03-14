# globe.decentrafly.com / feed

The decentrafly.com feed client is a toolkit that allows you to install, run and maintain a ADS-B / UAT / MLAT / ACARS / VDL2 feed client.

By default, it feeds MLAT+ADSB to feed.decentrafly.com. You can enable UAT/ACARS/VDL2, and feed to your plane data to FlightRadar24, Radarbox, Piaware, [and more](.env.example)

It is designed to be run on a Raspberry Pi, but can be run on any Linux Debian-like system.

With [a few commands](#feeding-directly-to-other-aggregators), you can easily feed to other community aggregators.

## ADSBExchange-style feeder

If you are looking for a script similar to the ADSBExchange feeder to run on your existing station, [you can see here](https://github.com/decentrafly/feed/tree/master).

Or run:
```
curl -fsL -o /tmp/decentrafly.sh https://raw.githubusercontent.com/decentrafly/feed/masterx/setup.sh && sudo bash /tmp/decentrafly.sh
```

## Quick Start

Run this **as root** on a fresh install of Raspberry Pi OS Lite or similar.

This script gets all the requirements for your system.

**For your own security,** Please consider [analysing](https://github.com/decentrafly/feed/blob/main/bin/decentrafly-init) the `decentrafly-init` script which you are about to run on your system.

```
curl -Ls https://raw.githubusercontent.com/decentrafly/feed/main/bin/decentrafly-init | bash
cd /opt/decentrafly/
cp .env.example .env
```

Then, set the environment variables.

You can either edit the `.env` file, or run `decentrafly-env set <key> <value>`

```
# Altitude in meters
decentrafly-env set FEEDER_ALT_M 542
# Latitude
decentrafly-env set FEEDER_LAT 98.76543
# Longitude
decentrafly-env set FEEDER_LONG 12.34567
# Timezone (see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
decentrafly-env set FEEDER_TZ America/New_York
# SDR Serial Number
decentrafly-env set ADSB_DONGLE_SERIAL 1090
# Site name (shows up on the MLAT map!)
decentrafly-env set MLAT_SITE_NAME "My epic site"
# Would you like to appear on globe.decentrafly.com? Then set this:
decentrafly-env unset DECENTRAFLY_MLAT_CONFIG
```

These are the minimum environment variables you need to set.

Then, run:
```
decentrafly-debug && decentrafly-up
```
Let's check if everything is working:

- [ ] <http://IP:8080> (readsb)
- [ ] <http://IP:8082> (decentrafly)
- [ ] <https://globe.decentrafly.com> (ADSB)
- [ ] <https://globe.decentrafly.com> (MLAT)

## Usage

By default, the client will feed to adsb.lol.

To see the current list of supported aggregators, see the [services.txt](services.txt) file.

The `decentrafly` service supports feeding to multiple aggregators.

If you have an issue with the feed client, please [paste.ee](https://paste.ee) your error logs join our chat on [discord](https://decentrafly.com/discord).


### Restart the stack

```
decentrafly-up
```

## Enabling a service

To enable a service, run `decentrafly-service enable <service>`
To disable a service, run `decentrafly-service disable <service>`
This is a helper command that will edit the `services.txt` file, and run `decentrafly-gen` to generate a new `cmdline.txt`.

You may have to define further environment variables in the `.env` file.

Then, run `decentrafly-gen` to generate a new cmdline.txt.

The cmdline.txt is used by the decentrafly binaries to know what services to start.

Once you have done this, run `decentrafly-up` to start the containers.

## Troubleshooting

To update, run `decentrafly-update`

Running `decentrafly-debug` will tell you about common mistakes.

### I cannot find myself on the MLAT Map

decentrafly.com enables the `--privacy` flag for your MLAT client by default.
This hides you from the MLAT map.

Do you want to appear on the map? Then run:

```
decentrafly-env unset DECENTRAFLY_MLAT_CONFIG && decentrafly-up
```

### Logs

- `decentrafly-logs` - view logs
- `decentrafly-logs -f` - view logs and follow

### Services
- `decentrafly-service enable <service>` - enable a service
- `decentrafly-service disable <service>` - disable a service
- `decentrafly-service list` - list all enabled services

### Environment
- `decentrafly-env list` - list all environment variables
- `decentrafly-env set <key>` - set an environment variable (also updates if it already exists)
- `decentrafly-env unset <key>` - unset an environment variable

### SDR
- `decentrafly-sdr test` - Runs rtl_test
- `decentrafly-sdr dockertest` - Runs rtl_test in a docker container
- `decentrafly-sdr dockerppm` - Runs rtl_test in a docker container with the intent to estimate the PPM

### Reset
- `decentrafly-reset` - reset the /opt/decentrafly directory

## Thank you SDR-Enthusiasts!

This would not be possible without [SDR-Enthusiasts](https://github.com/sdr-enthusiasts/) who have made [the original docker-compose](https://github.com/sdr-enthusiasts/docker-install) file. 

This repo is largely based off of their work plus some command line interface tools to make running the stack a bit simpler.

[Their documentation can be very useful in enabling extra feeders.](https://sdr-enthusiasts.gitbook.io/ads-b/feeder-containers/feeding-flightaware-piaware).


## Feeding directly to other aggregators

The `decentrafly` service can feed to other aggregators.

In this example, we also feed [ADSB.lol](https://adsb.lol). 

This is not an endorsement and decentrafly is not affiliated with adsb.lol.

### Run

**NOTE:** This is using `--privacy`, which excludes you from adsb.lol map, and should exclude you from other aggregators maps too.

```
decentrafly-env set DECENTRAFLY_ADDITIONAL_NET_CONNECTOR "feed.decentrafly.com,30004,beast_reduce_out;feed.adsb.lol,30004,beast_reduce_out"
decentrafly-env set DECENTRAFLY_ADDITIONAL_MLAT_CONFIG "feed.decentrafly.com,31090,39001,--privacy;feed.adsb.lol,31090,39002,--privacy"
decentrafly-env set MLATHUB_NET_CONNECTOR "decentrafly,39000,beast_in;decentrafly,39001,beast_in;decentrafly,39002,beast_in"
```
**If you would like to disable privacy mode, instead, use:**
```
decentrafly-env set DECENTRAFLY_ADDITIONAL_NET_CONNECTOR "feed.decentrafly.com,30004,beast_reduce_out;feed.adsb.lol,30004,beast_reduce_out"
decentrafly-env set DECENTRAFLY_ADDITIONAL_MLAT_CONFIG "feed.decentrafly.com,31090,39001;feed.adsb.lol,31090,39002"
decentrafly-env set MLATHUB_NET_CONNECTOR "decentrafly,39000,beast_in;decentrafly,39001,beast_in;decentrafly,39002,beast_in"
```

