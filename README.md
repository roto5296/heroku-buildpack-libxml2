Heroku buildpack: libxml2
=======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) of libxml2.

Unfortunately libxml2-2.7.6 has a [nasty bug][1] when using Nokogiri, this
buildpack includes version 2.7.7 which fixes the bug.

Unfortunately after much effort, attempting to set an environment
variable to force nokogiri to build against the injected library rather
than the system library doesn't work. However jamming in the injected
version at runtime seems to fix my bug (via `LD_PRELOAD`).

* `LD_LIBRARY_PATH` - doesn't seem to have an effect.
* Which requires that we use `LD_PRELOAD` to fix the issue.
* `LIBRARY_PATH` would seeming do what we want but it comes after paths
  specified by `-L` to gcc so it won't override the system library.
  Unfortunately mkmf in nokogiri insists on including `/usr/lib/` in the
  Makefile which I believe is due to the fact those flags are used when
  building ruby.
* Writing to `/opt/*` `/usr/*` is restricted, so we can't slip in our
  versions that way.
* Writing libraries to `vendor/ruby*/lib` don't seem to be picked up.
* `bundle config build.nokogiri --with-xml2-lib=dir` doesn't inject it's
  `-L` soon enough.

Usage
-----

Example usage:

```shell
$ heroku create --stack cedar --buildpack https://github.com/mcolyer/heroku-buildpack-libxml2.git

# or if your app is already created:
$ heroku config:add BUILDPACK_URL=https://github.com/mcolyer/heroku-buildpack-libxml2.git

$ git push heroku master
```

Instructions for building the binary
------------------------------------

```
heroku run bash

cd ~/
mkdir -p vendor/
cd vendor
curl -O "http://xmlsoft.org/sources/libxml2-2.7.7.tar.gz"
tar -zxvf libxml2-2.7.7.tar.gz
cd libxml2-2.7.7
./configure --prefix /app/vendor/libxml2
make
make install
cd ..
tar -cvjf libxml2-2.7.7-x86_64.tar.bz2 libxml2

curl  -o s3 https://raw.github.com/nzoschke/s3/master/s3.py
chmod +x s3

S3_SECRET_ACCESS_KEY= S3_ACCESS_KEY_ID= ./s3 put s3://some-bucket/libxml2-2.7.7-x86_64.tar.bz2
curl --request PUT --upload-file libxml2-2.7.7-x86_64.tar.bz2 "URL"
```

Note
-----

Use my fork of
[heroku-buildpack-multi](https://github.com/mcolyer/heroku-buildpack-multi)
to allow for proper export of variables to subsequent buildpacks, which
currently doesn't do anything useful.

[1]: https://github.com/sparklemotion/nokogiri/issues/458
