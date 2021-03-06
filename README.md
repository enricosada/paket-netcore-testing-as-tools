# Paket .NET Core

Both `paket` and `paket.bootstrapper` as .NET tool, trying to maintain a dev flow similar to current paket v5 (and hopefully minimal maintenance)

# Requirements

- [.NET Core Sdk 2.1.402]

`NOTE` install from zip/binaries (safe way), but you need to set `DOTNET_ROOT` env var to the dir of unzipped sdk (bug in `dotnet/cli` for preview https://github.com/dotnet/cli/issues/9114 )

# Try it

See the scenarios below.

The scenario 1 (`dotnet restore .paket`) maintain current flow with `paket.bootstrapper`

As .net tools so each one can be installed separately to specific dir, so can be run as normal native binaries. as

- `dotnet tool install paket.bootstrapper --version "[5.179.403]" --tool-path "mydir" --add-source https://www.myget.org/F/paket-netcore-as-tool/api/v3/index.json`
- `dotnet tool install paket --version "[5.179.403]" --tool-path "mydir2" --add-source https://www.myget.org/F/paket-netcore-as-tool/api/v3/index.json`

`NOTE` to both command you need to add the `--add-source` because is in myget:

PRO:

- maintan the `paket.bootstrapper` (download from github, etc) but is not mandatory
- the `paket.bootstrapper` download the nupkg (from github or nuget feed) and .net core sdk install it from a local dir (so not using nuget client)
- you can install just `paket` as .net tool, no need for `paket.bootstrapper`
- native exe in `.paket`, as usual

CONS:

- some dirty in `.paket` dir (the `.store`).
  - we can restore in `paket-files\paket-bin` and symlink/shellscript the exe?
- no atm `dotnet paket`. this will be another PR for an helper global tool (`dotnet tool install -g`)
- need .net core sdk. But we can use paket repotool when ready for `paket` .net full too

Behaviour:

- install the bootstrapper as .NET Tool
- run `paket.bootstrapper` but download the nupkg and install `paket` as .NET tool

# Scenarios

Before each scenario do a `git clean -xdf`

# scenario 1, download bootstrapper and paket

Install the boostrapper and restore `paket` with

`dotnet restore .paket`

after that, as usual

`.paket\paket --help`

or

`.paket\paket restore`

# scenario 2, integration with sdk

normal command like the following should work, without explicit bootstrapping

```
dotnet run -p src/c1
```

or for a suave app

```
dotnet run -p src/c1 -- --port 8083
```

# scenario 3, docker

In `Dockerfile`, with multi steps to reuse layers to build a smaller `alpine`+`.net core runtime`

build the image with

```
docker build . -t paket-netcore-app
```

Run just the console app

```
docker run paket-netcore-app
```

Run suave webapp (after that is avaiable at http://localhost:8083/ )

```
docker run -p 8083:8083 paket-netcore-app --port 8083
```

# scenario 4, just paket

To just download the `paket` in `.paket` dir, use

`dotnet tool install paket --version "[5.179.403]" --tool-path ".paket" --add-source https://www.myget.org/F/paket-netcore-as-tool/api/v3/index.json`

after that, as usual

`.paket\paket --help`

# EXPECTED TO WORK

- the `dotnet restore .paket` should work docker/win/osx/unix
- integration with sdk wihout explicit bootstrap first (just `dotnet build`) on win. fails on unix (just do `dotnet restore .paket` first)
- `.paket/paket` commands
- docker scenario

# KNOWN BUGS

- if there is a system proxy, there is an access error. the code for proxy management is temporary disabled on .net core version (thx @vaskir).

# EXPECTED TO NOT WORK (WIP)

- `dotnet paket`. It's not installed as global command. will do a workaround later. For now, use as before `.paket/paket --version`
- download from github, need a real version of paket deployed. atm just myget feed are usable (forcenuget or prefernuget)
- the cache is disabled.

# HOW TO MIGRATE FROM PAKET .NET

- require .net sdk >= 2.1.401 (best practice is to add a [global.json](https://raw.githubusercontent.com/enricosada/paket-netcore-testing-as-tool/master/global.json) in root)
- delete `.paket/paket.exe`
- delete `.paket/paket.bootstrapper.exe`
- append to .gitignore some rules

    ````
    /.paket/*
    !/.paket/*.proj
    !/.paket/*.exe.config
    !/.paket/*.props
    !/.paket/*.targets
    ````
- add `.paket/paket.bootstrapper.proj` from  [this repo .paket/paket.bootstrapper.proj](https://raw.githubusercontent.com/enricosada/paket-netcore-testing-as-tool/master/.paket/paket.bootstrapper.proj)
- because currently it's a prerelease, we need to add these to config the version and feeds
  - add `.paket/paket.bootstrapper.props` from  [this repo .paket/paket.bootstrapper.props](https://raw.githubusercontent.com/enricosada/paket-netcore-testing-as-tool/master/.paket/paket.bootstrapper.props)
  - add `.paket/paket.bootstrapper.exe.config` from  [this repo .paket/paket.bootstrapper.exe.config](https://raw.githubusercontent.com/enricosada/paket-netcore-testing-as-tool/master/.paket/paket.bootstrapper.exe.config)
- run `dotnet restore .paket` to bootstrap paket
- run `.paket/paket restore`. this will update `.paket/Paket.Restore.targets` to latest version

from now **on a clean repo**, like previous scenarios:

- `dotnet build` of project should work
- you can do `dotnet run .paket` to  explicit bootstrap paket
