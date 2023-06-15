# Rönd website

This repo olds the Rönd official website and documentation

## Contribution

Contribution are welcome, if you want to add or correct anything feel free to open a PR!

### 1. Get the code

You can clone the repository right to where you want to host the docs:

```bash
git clone git@github.com:rond-authz/rond-authz.github.io.git docs
cd docs
```

### 2. Run locally


The preferred way for serving the site locally is to use `docker compose`:

```bash
docker compose up
```

or, to run it as a daemon:

```bash
docker compose up -d
```

You can then open your browser to [http://localhost:4000](http://localhost:4000)
to see the server running.

> Note : changes `baseurl: ""` in _config.yml  when you are running in local and prod according to the requirement.

If you already have a working installation of jekyll you could instead run it directly with:

```bash
jekyll serve
# or
bundle exec jekyll serve
```

**NOTE:** If the above serve command throws an error saying `require': cannot load such file -- webrick (LoadError)` try to run `bundle add webrick` to automatically add the webrick gem to your Gemfile, or manually add `gem "webrick"` line to the Gemfile and then run the serve command again.
