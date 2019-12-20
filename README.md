<h1 align="center">
  <a href="https://ipfs.io"><img width="650px" src="https://ipfs.io/ipfs/QmQJ68PFMDdAsgCZvA1UVzzn18asVcf7HVvCDgpjiSCAse" alt="BTFS http client lib logo" /></a>
</h1>

<h3 align="center">The JavaScript HTTP client library for BTFS implementations.</h3>
<p align="center">
  <a href="https://app.fossa.io/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fipfs%2Fjs-ipfs-http-client?ref=badge_small" alt="FOSSA Status"><img src="https://app.fossa.io/api/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fipfs%2Fjs-ipfs-http-client.svg?type=small"/></a>
  <br>
  <a href="https://github.com/feross/standard"><img src="https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square"></a>
  <a href="https://github.com/RichardLitt/standard-readme"><img src="https://img.shields.io/badge/standard--readme-OK-green.svg?style=flat-square" /></a>
  <a href=""><img src="https://img.shields.io/badge/npm-%3E%3D3.0.0-orange.svg?style=flat-square" /></a>
  <a href=""><img src="https://img.shields.io/badge/Node.js-%3E%3D10.0.0-orange.svg?style=flat-square" /></a>
  <br>
</p>

> A client library for the BTFS HTTP API, implemented in JavaScript. This client library implements the [interface-ipfs-core](https://github.com/TRON-US/js-btfs-http-client) enabling applications to change between an embedded js-ipfs node and any remote BTFS node without having to change the code. In addition, this client library implements a set of utility functions.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Install](#install)
  - [Running the daemon with the right port](#running-the-daemon-with-the-right-port)
  - [Importing the module and usage](#importing-the-module-and-usage)
  - [Importing a sub-module and usage](#importing-a-sub-module-and-usage)
  - [In a web browser](#in-a-web-browser)
  - [CORS](#cors)
  - [Custom Headers](#custom-headers)
  - [Global Timeouts](#global-timeouts)
- [Usage](#usage)
  - [API](#api)
    - [Files](#files)
    - [Graph](#graph)
    - [Network](#network)
    - [Node Management](#node-management)
    - [Additional Options](#additional-options)
    - [Instance Utils](#instance-utils)
    - [Static Types and Utils](#static-types-and-utils)
- [Development](#development)
  - [Testing](#testing)
- [Contribute](#contribute)
- [Historical context](#historical-context)
- [License](#license)

## Install

This module uses node.js, and can be installed through npm:

```bash
npm install --save ipfs-http-client
```

We support both the Current and Active LTS versions of Node.js. Please see [nodejs.org](https://nodejs.org/) for what these currently are.

### Running the daemon with the right port

To interact with the API, you need to have a local daemon running. It needs to be open on the right port. `5001` is the default, and is used in the examples below, but it can be set to whatever you need.

```sh
# Show the btfs config API port to check it is correct
> btfs config Addresses.API
/ip4/127.0.0.1/tcp/5001
# Set it if it does not match the above output
> btfs config Addresses.API /ip4/127.0.0.1/tcp/5001
# Restart the daemon after changing the config

# Run the daemon
> btfs daemon
```

### Importing the module and usage

```javascript
const ipfsClient = require('ipfs-http-client')

// connect to ipfs daemon API server
const ipfs = ipfsClient('http://localhost:5001') // (the default in Node.js)

// or connect with multiaddr
const ipfs = ipfsClient('/ip4/127.0.0.1/tcp/5001')

// or using options
const ipfs = ipfsClient({ host: 'localhost', port: '5001', protocol: 'http' })

// or specifying a specific API path
const ipfs = ipfsClient({ host: '1.1.1.1', port: '80', apiPath: '/ipfs/api/v0' })
```

### Importing a sub-module and usage

```javascript
const bitswap = require('ipfs-http-client/src/bitswap')('/ip4/127.0.0.1/tcp/5001')

const list = await bitswap.wantlist(key)
// ...
```

### In a web browser

**through Browserify**

Same as in Node.js, you just have to [browserify](http://browserify.org) the code before serving it. See the browserify repo for how to do that.

See the example in the [examples folder](/examples/bundle-browserify) to get a boilerplate.

**through webpack**

See the example in the [examples folder](/examples/bundle-webpack) to get an idea on how to use `js-ipfs-http-client` with webpack.

**from CDN**

Instead of a local installation (and browserification) you may request a remote copy of BTFS API from [unpkg CDN](https://unpkg.com/).

To always request the latest version, use the following:

```html
<script src="https://unpkg.com/ipfs-http-client/dist/index.min.js"></script>
```

Note: remove the `.min` from the URL to get the human-readable (not minified) version.

For maximum security you may also decide to:

* reference a specific version of BTFS API (to prevent unexpected breaking changes when a newer latest version is published)
* [generate a SRI hash](https://www.srihash.org/) of that version and use it to ensure integrity
* set the [CORS settings attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes) to make anonymous requests to CDN

Example:

```html
<script src="https://unpkg.com/btfs-http-client@9.0.0/dist/index.js"
integrity="sha384-5bXRcW9kyxxnSMbOoHzraqa7Z0PQWIao+cgeg327zit1hz5LZCEbIMx/LWKPReuB"
crossorigin="anonymous"></script>
```

CDN-based BTFS API provides the `IpfsHttpClient` constructor as a method of the global `window` object. Example:

```js
const ipfs = window.IpfsHttpClient({ host: 'localhost', port: 5001 })
```

If you omit the host and port, the client will parse `window.host`, and use this information. This also works, and can be useful if you want to write apps that can be run from multiple different gateways:

```js
const ipfs = window.IpfsHttpClient()
```

### CORS

In a web browser BTFS HTTP client (either browserified or CDN-based) might encounter an error saying that the origin is not allowed. This would be a CORS ("Cross Origin Resource Sharing") failure: BTFS servers are designed to reject requests from unknown domains by default. You can whitelist the domain that you are calling from by changing your ipfs config like this:

```console
$ ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin  '["http://example.com"]'
$ ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "POST", "GET"]'
```

### Custom Headers

If you wish to send custom headers with each request made by this library, for example, the Authorization header. You can use the config to do so:

```js
const ipfs = ipfsClient({
  host: 'localhost',
  port: 5001,
  protocol: 'http',
  headers: {
    authorization: 'Bearer ' + TOKEN
  }
})
```

### Global Timeouts

To set a global timeout for _all_ requests pass a value for the `timeout` option:

```js
// Timeout after 10 seconds
const ipfs = ipfsClient({ timeout: 10000 })
// Timeout after 2 minutes
const ipfs = ipfsClient({ timeout: '2m' })
// see https://www.npmjs.com/package/parse-duration for valid string values
```

## Usage

### API

> `js-btfs-http-client` follows the spec defined by [`interface-ipfs-core`](https://github.com/ipfs/interface-js-ipfs-core), which concerns the interface to expect from BTFS implementations. This interface is a currently active endeavor. You can use it today to consult the methods available.

#### Files

- [Regular Files API](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md)
  - [`ipfs.add(data, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#add)
  - [`ipfs.addPullStream([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#addpullstream)
  - [`ipfs.addReadableStream([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#addreadablestream)
  - [`ipfs.addFromStream(stream)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#addfromstream)
  - [`ipfs.addFromFs(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#addfromfs)
  - [`ipfs.addFromURL(url, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#addfromurl)
  - [`ipfs.cat(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#cat)
  - [`ipfs.catPullStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#catpullstream)
  - [`ipfs.catReadableStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#catreadablestream)
  - [`ipfs.get(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#get)
  - [`ipfs.getPullStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#getpullstream)
  - [`ipfs.getReadableStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#getreadablestream)
  - [`ipfs.ls(ipfsPath)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#ls)
  - [`ipfs.lsPullStream(ipfsPath)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#lspullstream)
  - [`ipfs.lsReadableStream(ipfsPath)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#lsreadablestream)
- [MFS (mutable file system) specific](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#mutable-file-system)

  _Explore the Mutable File System through interactive coding challenges in our [ProtoSchool tutorial](https://proto.school/#/mutable-file-system/)._
  - [`ipfs.files.cp([from, to])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filescp)
  - [`ipfs.files.flush([path])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesflush)
  - [`ipfs.files.ls([path], [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesls)
  - [`ipfs.files.mkdir(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesmkdir)
  - [`ipfs.files.mv([from, to])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesmv)
  - [`ipfs.files.read(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesread)
  - [`ipfs.files.readPullStream(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesreadpullstream)
  - [`ipfs.files.readReadableStream(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesreadreadablestream)
  - [`ipfs.files.rm(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesrm)
  - [`ipfs.files.stat(path, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#filesstat)
  - [`ipfs.files.write(path, content, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/FILES.md#fileswrite)

- [block](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BLOCK.md)
  - [`ipfs.block.get(cid, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BLOCK.md#blockget)
  - [`ipfs.block.put(block, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BLOCK.md#blockput)
  - [`ipfs.block.stat(cid)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BLOCK.md#blockstat)

- [refs](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md)
  - [`ipfs.refs(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refs)
  - [`ipfs.refsReadableStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refsreadablestream)
  - [`ipfs.refsPullStream(ipfsPath, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refspullstream)
  - [`ipfs.refs.local()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refslocal)
  - [`ipfs.refs.localReadableStream()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refslocalreadablestream)
  - [`ipfs.refs.localPullStream()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REFS.md#refslocalpullstream)

#### Graph

- [dag](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DAG.md)

  _Explore the DAG API through interactive coding challenges in our [ProtoSchool tutorial](https://proto.school/#/basics)._
  - [`ipfs.dag.get(cid, [path], [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DAG.md#dagget)
  - [`ipfs.dag.put(dagNode, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DAG.md#dagput)
  - [`ipfs.dag.tree(cid, [path], [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DAG.md#dagtree)

- [object](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md)
  - [`ipfs.object.data(multihash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectdata)
  - [`ipfs.object.get(multihash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectget)
  - [`ipfs.object.links(multihash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectlinks)
  - [`ipfs.object.new([template])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectnew)
  - [`ipfs.object.patch.addLink(multihash, DAGLink, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectpatchaddlink)
  - [`ipfs.object.patch.appendData(multihash, data, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectpatchappenddata)
  - [`ipfs.object.patch.rmLink(multihash, DAGLink, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectpatchrmlink)
  - [`ipfs.object.patch.setData(multihash, data, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectpatchsetdata)
  - [`ipfs.object.put(obj, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectput)
  - [`ipfs.object.stat(multihash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/OBJECT.md#objectstat)

- [pin](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PIN.md)
  - [`ipfs.pin.add(hash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PIN.md#pinadd)
  - [`ipfs.pin.ls([hash], [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PIN.md#pinls)
  - [`ipfs.pin.rm(hash, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PIN.md#pinrm)

- refs
  - `ipfs.refs.local()`

#### Network

- [bootstrap](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BOOTSTRAP.md)
  - [`ipfs.bootstrap.add(addr, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BOOTSTRAP.md#bootstrapadd)
  - [`ipfs.bootstrap.list()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BOOTSTRAP.md#bootstraplist)
  - [`ipfs.bootstrap.rm(addr, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BOOTSTRAP.md#bootstraprm)

- [bitswap](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BITSWAP.md)
  - [`ipfs.bitswap.stat()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BITSWAP.md#bitswapstat)
  - [`ipfs.bitswap.wantlist([peerId])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BITSWAP.md#bitswapwantlist)
  - [`ipfs.bitswap.unwant(cid)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/BITSWAP.md#bitswapunwant)

- [dht](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md)
  - [`ipfs.dht.findPeer(peerId)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md#dhtfindpeer)
  - [`ipfs.dht.findProvs(hash)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md#dhtfindprovs)
  - [`ipfs.dht.get(key)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md#dhtget)
  - [`ipfs.dht.provide(cid)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md#dhtprovide)
  - [`ipfs.dht.put(key, value)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/DHT.md#dhtput)

- [pubsub](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md)
  - [`ipfs.pubsub.ls(topic)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md#pubsubls)
  - [`ipfs.pubsub.peers(topic)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md#pubsubpeers)
  - [`ipfs.pubsub.publish(topic, data)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md#pubsubpublish)
  - [`ipfs.pubsub.subscribe(topic, handler, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md#pubsubsubscribe)
  - [`ipfs.pubsub.unsubscribe(topic, handler)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/PUBSUB.md#pubsubunsubscribe)

- [swarm](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/SWARM.md)
  - [`ipfs.swarm.addrs()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/SWARM.md#swarmaddrs)
  - [`ipfs.swarm.connect(addr)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/SWARM.md#swarmconnect)
  - [`ipfs.swarm.disconnect(addr)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/SWARM.md#swarmdisconnect)
  - [`ipfs.swarm.peers([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/SWARM.md#swarmpeers)

- [name](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md)
  - [`ipfs.name.publish(addr, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md#namepublish)
  - [`ipfs.name.pubsub.cancel(arg)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md#namepubsubcancel)
  - [`ipfs.name.pubsub.state()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md#namepubsubstate)
  - [`ipfs.name.pubsub.subs()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md#namepubsubsubs)
  - [`ipfs.name.resolve(addr, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/NAME.md#nameresolve)

#### Node Management

- [miscellaneous operations](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md)
  - [`ipfs.dns(domain)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#dns)
  - [`ipfs.id()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#id)
  - [`ipfs.ping(id, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#ping)
  - [`ipfs.pingPullStream(id, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#pingpullstream)
  - [`ipfs.pingReadableStream(id, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#pingreadablestream)
  - [`ipfs.stop()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#stop). Alias to `ipfs.shutdown`.
  - [`ipfs.version()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/MISCELLANEOUS.md#version)

- [config](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md)
  - [`ipfs.config.get([key])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md#configget)
  - [`ipfs.config.replace(config)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md#configreplace)
  - [`ipfs.config.set(key, value)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md#configset)
  - [`ipfs.config.profiles.list()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md#configprofileslist)
  - [`ipfs.config.profiles.apply(name, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/CONFIG.md#configprofilesapply)

- [stats](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md)
  - [`ipfs.stats.bitswap()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md#statsbitswap)
  - [`ipfs.stats.bw([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md#statsbw)
  - [`ipfs.stats.bwPullStream([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md#statsbwpullstream)
  - [`ipfs.stats.bwReadableStream([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md#statsbwreadablestream)
  - [`ipfs.stats.repo([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/STATS.md#statsrepo)

- log
  - `ipfs.log.level(subsystem, level, [options])`
  - `ipfs.log.ls()`
  - `ipfs.log.tail()`

- [repo](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REPO.md)
  - [`ipfs.repo.gc([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REPO.md#repogc)
  - [`ipfs.repo.stat([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REPO.md#repostat)
  - [`ipfs.repo.version()`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/REPO.md#repoversion)

- [key](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md)
  - [`ipfs.key.export(name, password)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keyexport)
  - [`ipfs.key.gen(name, [options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keygen)
  - [`ipfs.key.import(name, pem, password)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keyimport)
  - [`ipfs.key.list([options])`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keylist)
  - [`ipfs.key.rename(oldName, newName)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keyrename)
  - [`ipfs.key.rm(name)`](https://github.com/ipfs/interface-ipfs-core/blob/master/SPEC/KEY.md#keyrm)

#### Additional Options

All core API methods take _additional_ `options` specific to the HTTP API:

* `headers` - An object or [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers) instance that can be used to set custom HTTP headers. Note that this option can also be [configured globally](#custom-headers) via the constructor options.
* `signal` - An [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) that can be used to abort the request on demand.
* `timeout` - A number or string specifying a timeout for the request. If the timeout is reached before data is received a [`TimeoutError`](https://github.com/sindresorhus/ky/blob/2f37c3f999efb36db9108893b8b3d4b3a7f5ec45/index.js#L127-L132) is thrown. If a number is specified it is interpreted as milliseconds, if a string is passed, it is intepreted according to [`parse-duration`](https://www.npmjs.com/package/parse-duration). Note that this option can also be [configured globally](#global-timeouts) via the constructor options.
* `searchParams` - An object or [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) instance that can be used to add additional query parameters to the query string sent with each request.

#### Instance Utils

- `ipfs.getEndpointConfig()`

Call this on your client instance to return an object containing the `host`, `port`, `protocol` and `api-path`.

#### Static Types and Utils

Aside from the default export, `btfs-http-client` exports various types and utilities that are included in the bundle:

- [`isIPFS`](https://www.npmjs.com/package/is-ipfs)
- [`Buffer`](https://www.npmjs.com/package/buffer)
- [`PeerId`](https://www.npmjs.com/package/peer-id)
- [`PeerInfo`](https://www.npmjs.com/package/peer-info)
- [`multiaddr`](https://www.npmjs.com/package/multiaddr)
- [`multibase`](https://www.npmjs.com/package/multibase)
- [`multicodec`](https://www.npmjs.com/package/multicodec)
- [`multihash`](https://www.npmjs.com/package/multihash)
- [`CID`](https://www.npmjs.com/package/cids)

These can be accessed like this, for example:

```js
const { CID } = require('ipfs-http-client')
// ...or from an es-module:
import { CID } from 'ipfs-http-client'
```

## Development

### Testing

We run tests by executing `npm test` in a terminal window. This will run both Node.js and Browser tests, both in Chrome and PhantomJS. To ensure that the module conforms with the [`interface-ipfs-core`](https://github.com/ipfs/interface-ipfs-core) spec, we run the batch of tests provided by the interface module, which can be found [here](https://github.com/ipfs/interface-ipfs-core/tree/master/js/src).

## Contribute

The js-btfs-http-client is a work in progress. As such, there's a few things you can do right now to help out:

- **[Check out the existing issues](https://github.com/ipfs/js-btfs-http-client/issues)**!
- **Perform code reviews**. More eyes will help a) speed the project along b) ensure quality and c) reduce possible future bugs.
- **Add tests**. There can never be enough tests. Note that interface tests exist inside [`interface-ipfs-core`](https://github.com/ipfs/interface-ipfs-core/tree/master/js/src).
- **Contribute to the [FAQ repository](https://github.com/ipfs/faq/issues)** with any questions you have about BTFS or any of the relevant technology. A good example would be asking, 'What is a merkledag tree?'. If you don't know a term, odds are, someone else doesn't either. Eventually, we should have a good understanding of where we need to improve communications and teaching together to make BTFS and IPN better.

## License

[MIT](LICENSE)

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fipfs%2Fjs-ipfs-http-client.svg?type=large)](https://app.fossa.io/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fipfs%2Fjs-ipfs-http-client?ref=badge_large)
