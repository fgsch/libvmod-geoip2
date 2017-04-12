libvmod-geoip2
==============

[![Join the chat at https://gitter.im/fgsch/libvmod-geoip2](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fgsch/libvmod-geoip2?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/fgsch/libvmod-geoip2.svg?branch=master)](https://travis-ci.org/fgsch/libvmod-geoip2)

## About

A Varnish 4.1 and 5.x VMOD to query MaxMind GeoIP2 DB files.

For Varnish master refer to the devel branch.

## Requirements

To build this VMOD you will need:

* make
* a C compiler, e.g. GCC or clang
* pkg-config
* python-docutils or docutils in macOS [1]
* varnish-dev in Debian/Ubuntu, varnish-devel in CentOS/RedHat or
  varnish in macOS [1]
* libmaxminddb-dev in recent Debian/Ubuntu releases, maxminddb in
  macOS [1]. See also https://github.com/maxmind/libmaxminddb

If you are building from Git, you will also need:

* autoconf
* automake
* libtool

In addition, to run the tests you will need:

* varnish

If varnish is installed in a non-standard prefix you will also need
to set `PKG_CONFIG_PATH` to the directory where **varnishapi.pc** is
located before running `autogen.sh` and `configure`.  For example:

```
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

Finally, to use it you will need one or more GeoIP2 or GeoLite2 databases in MaxMind DB binary format.
See https://dev.maxmind.com/.

## Installation

### From a tarball

To install this VMOD, run the following commands:

```
./configure
make
make check
sudo make install
```

The `make check` step is optional but it's good to know whether the
tests are passing on your platform.

### From the Git repository

To install from Git, clone this repository by running:

```
git clone --recursive https://github.com/fgsch/libvmod-geoip2
```

And then run `./autogen.sh` followed by the instructions above for
installing from a tarball.

Finally place your MaxMind DB binary download (for example GeoLite2-Country.mmdb) in a path readable by Varnish.

## Example 1: Check the country name (in English)

```
import geoip2;

sub vcl_init {
        # You need to supply the full path to the mmdb file, for example /etc/maxmind/GeoLite2-Country.mmdb
	new country = geoip2.geoip2("GeoLite2-Country.mmdb");
}

sub vcl_recv {
	if (country.lookup("country/names/en", client.ip) != "Japan") {
		...
	}
}
```

## Example 2: Pass the ISO code to the backend servers

```
import geoip2;

sub vcl_init {
        # You need to supply the full path to the mmdb file, for example /etc/maxmind/GeoLite2-Country.mmdb
	new country = geoip2.geoip2("GeoLite2-Country.mmdb");
}

sub vcl_recv {
	
	...
	set req.http.X-Country-Code = country.lookup("country/iso_code", client.ip);
	...
	
}
```

## Example 3: Pass the ISO code to the backend servers using first X-FORWARDED-FOR IP if supplied

```
import geoip2;

sub vcl_init {
        # You need to supply the full path to the mmdb file, for example /etc/maxmind/GeoLite2-Country.mmdb
	new country = geoip2.geoip2("GeoLite2-Country.mmdb");
}

sub vcl_recv {
	
	...
	set req.http.X-Country-Code = country.lookup("country/iso_code", std.ip(regsub(req.http.x-forwarded-for, "^(\d+\.\d+\.\d+\.\d+).*", "\1"), client.ip));
	...
	
}
```


## License

This VMOD is licensed under BSD license. See LICENSE for details.

### Note

1. Using Homebrew, https://github.com/Homebrew/brew/.
