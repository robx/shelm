# shelm
_subverting Elm packaging since 2019_

__status: works for me, might eat your files__

`shelm` is a bash wrapper around the `elm` tool that allows bypassing
the Elm packaging infrastructure.

It allows you to
- depend on Elm packages that have not been published to
  [package.elm-lang.org](https://package.elm-lang.org), such as
  versioned Github releases, local paths or arbitrary git urls.
- work with native Javascript code: Depend on modified versions of core
  Elm modules or write your own kernel modules by placing a package within
  the `elm/` or `elm-explorations/` package namespace.

__Note: Best not talk about this on the offical Elm channels unless you're
trolling.__


## How to use

`shelm` expects to be called from the top level directory of an Elm project,
which contains an `elm.json` file. The basic workflow is: First call
`shelm fetch` to download dependencies into the local package cache. Then
call `shelm make` as you would usually call `elm make`. (Other `elm` subcommands
are passed through, too, but might not work.)

With a regular `elm.json`, there should be no functional difference to normal
`elm` tool use. To make use of `shelm`'s extra features, you might:

- Add dependencies for a packages that have not been released to the Elm
  package repository. Suppose you have an Elm package in a GitHub repository
  `me/elm-experiment` that you haven't published using `elm publish`. Then if
  you tag a commit as "1.0.0", depending on
  ```
  "dependencies": {
      "direct": {
          ...
          "me/elm-experiment": "1.0.0",
          ...
  ```
  in `elm.json` as usual will make `shelm` download the 1.0.0 release.

- Specify alternative locations for dependencies. E.g., to use a patched
  version of the `elm/time` package that's published at `github.com/me/elm-time`,
  you would change the dependency section `elm.json` to read
  ```
  "dependencies": {
      "direct": {
          ...
          "elm/time": "1.0.0"
      },
      "indirect": {
          ...
      },
      "locations": {
          "elm/time": {
              "method": "github",
              "name": "me/elm-time"
          }
      }
  }
  ```
  See below for full documentation of the `"locations"` field.


## Locations

`shelm` currently supports the following location types:

__github__

Parameters: `name` as "author/project".

Tagged GitHub releases, as used by regular `elm` packages. Downloads the
source archive for the release specified by the dependency version.

See above for an example.

__file__

Parameters: `path` as a local directory.

Copies over the given path as the source.

E.g., if you're developing `my-fancy-gui-toolkit` and want to test
it in `my-flashy-app` before publishing, you might tweak `my-flashy-app`'s
`elm.json` to include
```
"locations": {
    "me/my-fancy-gui-toolkit": {
        "method": "file",
        "path": "../my-fancy-gui-toolkit"
    }
}
```

__git__

Parameters: `url` as a git url, `ref` as a branch/tag/commit (defaults to version).

Checks out the given reference from the given git repository.

E.g., to build your application against the upstream development version
of `elm/time`,
```
"locations": {
    "elm/time": {
        "method": "git",
        "url": "https://https://github.com/elm/time.git",
        "ref": "master"
    }
}
```


## Troubleshooting

The `elm` tool error messages are explicitly unhelpful if something goes wrong
compiling dependencies. If you get some complaints about corruption or version
conflicts, chances are that `shelm` or your `elm.json` setup is at fault. Wiping
`elm-stuff` and calling `shelm fetch` again is a good first try.

It's important that both `elm make` and `elm make --docs=docs.json` works for
each dependency.

It's your responsibility to make sure that the source archive pointed at by
a location actually has the same version as listed in the dependencies. Things
are likely to go wrong if those don't align.


## Dependencies

__Elm compiler__

The `elm` tool should be in the path. Versions 0.19.0 and 0.19.1 are supported.

__jq__

The [jq](https://stedolan.github.io/jq/) JSON processor is used to read `elm.json`.

__curl, git, tar__

`git` is required the "git" location method, `curl` and `tar` for the default
"github" archive method.


## How it works

shelm manages its own local copy of the Elm package database in
`elm-stuff/home/.elm`, by downloading source archives using a variety
of measures, and generating a matching package registry.
It calls `elm` with `$HOME` pointed at this directory and networking
disabled by setting an invalid `$HTTP_PROXY` variable.

The following two articles go into this with a bit more detail:

- https://vllmrt.net/spam/guix-elm-2.html describes a Guix build system
  for Elm applications that uses the same techniques.
- https://vllmrt.net/spam/subverting-elm.html shows how shelm came about
  when figuring out how to build an Elm app with native code.
