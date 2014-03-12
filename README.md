Heroku buildpack: libxml2
=======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) of libxml2.

Unfortunately libxml2-2.7.6 has a [nasty bug][1] when using Nokogiri, this
builpack includes version 2.7.7 which fixes the bug.

Usage
-----

Example usage:

```shell
$ heroku create --stack cedar --buildpack https://github.com/mcolyer/heroku-buildpack-libxml2.git

# or if your app is already created:
$ heroku config:add BUILDPACK_URL=https://github.com/mcolyer/heroku-buildpack-libxml2.git

$ git push heroku master
```

Note
-----

Use my fork of
[heroku-buildpack-multi](https://github.com/mcolyer/heroku-buildpack-multi)
to allow for proper export of variables to subsequent buildpacks.

[1]: https://github.com/sparklemotion/nokogiri/issues/458
