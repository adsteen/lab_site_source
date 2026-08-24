# Building the Site

This site is an older Hugo Academic site that has been updated to build with
modern Hugo.

## Required Software

- Hugo Extended. The current verified version is:

  ```sh
  hugo v0.148.0+extended+withdeploy
  ```

On macOS with Homebrew, install or update it with:

```sh
brew install hugo
brew upgrade hugo
```

Check that the installed version includes `extended`:

```sh
hugo version
```

## Build

From the repository root:

```sh
hugo --gc --minify
```

If Hugo cannot write to the default user cache directory, point the cache at a
writable location:

```sh
HUGO_CACHEDIR=/tmp/lab_site_source_hugo_cache hugo --gc --minify
```

## Deploy Caveat

`deploy.sh` copies `static/files/cv.pdf` from a local Dropbox path before it
builds. If that source file is missing on a machine, the deploy script will fail
before Hugo runs. Either update that path or copy the CV into
`static/files/cv.pdf` manually before deploying.
