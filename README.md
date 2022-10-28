# Delegated Routing Practice

You can run an indexer locally or use a managed service like https://cid.contact, which is available for anyone to use. For the latter, your IPFS node *must* be reachable externally at the advertised addresses as the indexer verifies providers' reachability.  

## Prerequisites

Have GO v1.18+ installed.

Make sure that `GOPATH/bin` is on your `PATH` 

```
$ export GOPATH=$HOME/go
$ export PATH=$PATH:$GOPATH/bin
```

## Set up local indexer (Optional)

An indexer can be setup as per [these instructions](https://github.com/filecoin-project/storetheindex/#install). 

_TLDR;_

Install storetheindex:

```
$ go install github.com/filecoin-project/storetheindex@v0.4.28
```

Initialize the storetheindex repository and configuration:

```
$ storetheindex init
```

Start the indexer

```
$ storetheindex daemon
```


## Set up Kubo

Install Kubo v0.16.0 as per instructions [here](https://github.com/ipfs/kubo#install), or use already configired node. _It's important to make sure that you are on v0.16.0+ version of Kubo or otherwise Reframe won't work._

Initialise Kubo configuration

```
$ ipfs init
```

Setup a custom routing by adding the following configuration block into `~/.ipfs/config`

```
ipfs config Routing --json  '{
  "Type": "custom",
  "Methods": {
    "find-peers": {
      "RouterName": "WanDHT"
    },
    "find-providers": {
      "RouterName": "WanDHT"
    },
    "get-ipns": {
      "RouterName": "WanDHT"
    },
    "provide": {
      "RouterName": "ParallelHelper"
    },
    "put-ipns": {
      "RouterName": "WanDHT"
    }
  },
  "Routers": {
    "CidContact": {
      "Parameters": {
        "Endpoint": "http://127.0.0.1:50617"
      },
      "Type": "reframe"
    },
    "ParallelHelper": {
      "Parameters": {
        "Routers": [
          {
            "IgnoreErrors": true,
            "RouterName": "CidContact",
            "Timeout": "30m"
          },
          {
            "ExecuteAfter": "2s",
            "IgnoreErrors": true,
            "RouterName": "WanDHT",
            "Timeout": "30m"
          }
        ]
      },
      "Type": "parallel"
    },
    "WanDHT": {
      "Parameters": {
        "AcceleratedDHTClient": true,
        "Mode": "dhtserver",
        "PublicIPNetwork": true
      },
      "Type": "dht"
    }
  }
}'
```

Start the IPFS node:

```
$ ipfs daemon
```

Take a note of the `Identity.PeerID` from the Kubo config.


### Set up index-provider

Index-provider can be set up as per [these instructions](https://github.com/filecoin-project/index-provider#install).

_TLDR;_

Install index-provider:

```
$ go install github.com/filecoin-project/index-provider/cmd/provider@v0.9.0
```

Initialise the index-provider repository and configuration:

```
$ provider init
```

Add a direct http announcement (optional) and Reframe configuration:

```
$ vim ~/.index-provider/config
```

Add the following configuration block replacing the ProviderID with the one grabbed at the previous step: 

```
 "DirectAnnounce": {
    "URLs": [
      "http://127.0.0.1:3001" 
    ]
  },
  "Reframe": {
    "ListenMultiaddr": "/ip4/127.0.0.1/tcp/50617",
    "ChunkSize": 1,
    "SnapshotSize": 100,
    "ProviderID": "12D3KooWKZA9t5VoXwRydUKNkmmYoqB2dQTt6B4qVxfFuiwiGbtj", 
    "Addrs": [         
      "/ip4/0.0.0.0/tcp/4001",
      "/ip6/::/tcp/4001",
      "/ip4/0.0.0.0/udp/4001/quic",
      "/ip6/::/udp/4001/quic"
    ]
  }
```

Start the index-provider:

```
$ provider daemon
```

### See all in action

Open up a WebUI at http://127.0.0.1:5001/webui and upload a file.

Take a not of its CID.

Once upoloaded you should see logs start rolling in `storetheindex` and `indexprovider` command lines.

Check whether your cid got indexed by executing

```
$ storetheindex find --cid
```

 



