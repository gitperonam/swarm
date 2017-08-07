# **SWARM**  

- [Installation](#installation)
- [Running](#running)
    - [Running options](#running-options)
- [Usage](#usage)
    - [File upload](#file-upload)- [**SWARM**](#swarm)
- [Installation](#installation)
- [Running](#running)
    - [Running options](#running-options)
- [Usage](#usage)
    - [File upload](#file-upload)
    - [Directory upload](#directory-upload)
    - [File retrieval](#file-retrieval)
    - [Other ways to interact](#other-ways-to-interact)
    - [Attach Geth console](#attach-geth-console)
- [Manifests](#manifests)
    - [Creating custom manifests ("routing table")](#creating-custom-manifests-routing-table)
        - [Example](#example)
- [Considerations](#considerations)
    - [Directory upload](#directory-upload)
    - [File retrieval](#file-retrieval)
    - [Other ways to interact](#other-ways-to-interact)
    - [Attach Geth console](#attach-geth-console)
- [Manifests](#manifests)
- [Considerations](#considerations)

# Installation

 The installation and initial setup is pretty straightforward. A guide detailing this process can be found [here](http://swarm-guide.readthedocs.io/en/latest/installation.html#installation).  
# Running  
The most barebone running command is  
```bash
$SWARM_BIN_PATH/swarm --bzzacount value 
```
Although `--datadir` has a default value, is better practice to specify a directory. This directory should contain the keystore where the account's key is, if not, the path of the keystore will be specified with `--keystore` 
```bash  
$SWARM_BIN_PATH/swarm --bzzacount <value> --datadir <value>
``` 
## Running options

 The most used options are:  
* ```--bzzaccount value```
Sets the Ethereum account to be used. ***mandatory***

* ```--datadir value ```  
Directory which will be used to store all SWARM files. Also serves as default location of keystore (if not specified with ```--keystore```) 
* ```--ens-api value ```  
If conecting to geth (blockChain), value would be geth.ipc path. If using SWARM without connecting to geth, value is ' ' .
* ```--httpaddr value ```  
Exposes the SWARM HTTP API  in the specified address (default is *localhost:8500*).
# Usage
## File upload
To upload a file we use the `up` command
```bash
$SWARM_BIN_PATH/swarm up <fileName>
```
## Directory upload
To upload a whole directory we add the `--recursive` option
```bash
$SWARM_BIN_PATH/swarm --recursive up <directoryName>
```
In both cases the output will yield the hash of the manifest associeted to the file/files  
## File retrieval
In order to retrieve a file we have to access it relatively to the manifest
```
http://<swarm-gateway>/bzz:/<manifestHash>/<path>
```
Where swarm-gateway is *localhost:8500* by default (to use a different address use `--httpaddr <gateway-address>` )  
We can explore the files asociated to the manifest 
```
http://<swarm-gateway>/bzz:/<manifestHash>?list=true
```
If no filename is especified in the path, it wil try to serve the default empty-path file. This empty-path file can be specified by modifying the manifest 
## Other ways to interact
We can also upload files using the [HTTP API](http://swarm-guide.readthedocs.io/en/latest/usage.html#the-http-api) or the [IPC API](http://swarm-guide.readthedocs.io/en/latest/usage.html#swarm-ipc-api).  

The HTTP API requests will be sent to a swarm gateway ( *localhost:8500*, the address specified with `--httpaddr` or to a public swarm-gateway like [http://swarm-gateways.net](http://swarm-gateways.net/) )  
The IPC API is used through a geth console conected to the Swarm ipc.  
## Attach Geth console 
In order to attach a geth console to SWARM we use:  
```bash
geth attach <swarm_datadir>/bzzd.ipc 
```
# Manifests  
When uploading  files, a manifest  which contains the hashes and paths of the files is also uploaded. The hash the console returns is the manifest's hash.  
We could think of manifests as a file that links paths to files **i.e** given a path it serves a specific file. 

Manifests have a default file to serve when a empty route ' ' is requested (*index.html* is the default). This default file can be changed by modyfing the manifest      (either doing it [manually](https://github.com/ethereum/go-ethereum/wiki/URL-Scheme#swarm-webservers)  or through the IPC API upload function, which takes a defaultFile as one of the parameters).

## Creating custom manifests ("routing table")  
Custom manifests allow us to specify path-resource pairs. With this we can create, for example, simple web servers.
In order to create a custom manifest we first have to upload the files which are going to be referenced in the manifest. When uploading the files we use the flag ```--manifest=false``` which tells SWARM to upload the file without creating a manifest, and to display the file's hash (not the manifest asociated to the file)
  
After this we will create a json manifest where we will specify all the custom routing. Finally we upload the manifest using the ``` --manifest=false ``` flag. 

---
### Example

In the following example we will create a simple web server that has two html (index and page1) and an image (logo.png).  
First we upload the files

```bash
$SWARM_BIN_PATH/swarm --manifest=false up index.html
$SWARM_BIN_PATH/swarm --manifest=false up page1.html
$SWARM_BIN_PATH/swarm --manifest=false up logo.png
```
The output will look somenthing like this (the hash depends entirely on the content of the file)

```bash
$SWARM_BIN_PATH/swarm --manifest=false up index.html
ffcfee8c4057fd84e7f42191ade3e83f74c8f1d122943a6030ad2c347e586919

$SWARM_BIN_PATH/swarm--manifest=false up page1.html
2f8ac92ea82565f9f638fa6ee0bc21a81ecf7d13110aa5f35e42235ea7161b5e

$SWARM_BIN_PATH/swarm --manifest=false up logo.png
378a12c60962d7a17c1b65e0044bb74d3e024fee6b3a6ce2ec5c5d054b3dbf0a

```
At this point the files are uploaded to swarm and we have the hashes of those files. Now we need to create a manifest that references the files, and upload it. The most basic `manifest.json` will look somenthing like this:

```
{
    "entries": [
        {
            "hash": "ffcfee8c4057fd84e7f42191ade3e83f74c8f1d122943a6030ad2c347e586919"
        },

        {
            "path": "index",
            "hash": "ffcfee8c4057fd84e7f42191ade3e83f74c8f1d122943a6030ad2c347e586919"
        },
        {
            "path": "page1",
            "hash": "2f8ac92ea82565f9f638fa6ee0bc21a81ecf7d13110aa5f35e42235ea7161b5e"
        },
        {
            "path": "img/logo",
            "hash": "378a12c60962d7a17c1b65e0044bb74d3e024fee6b3a6ce2ec5c5d054b3dbf0a"
        }
    ]


```
To upload it
```bash
$SWARM_BIN_PATH/swarm --manifest=false up manifest.json
4c97d663b572d148f1c0c2ac46e8b40c8be69423ca43543eab7e8fb2fcafd2a4
```

The first entry doesn't have a path field, SWARM will interpretate this entry as the default file to serve (index.html) for the empty path. I we want to have a link in index.html pointing to page1.html the link's href would be `./page1`, if we want to display the image `./img/logo`, and so on.

To access the webpage index.html
```
http://<swarm-gateway>/bzz:/4c97d663b572d148f1c0c2ac46e8b40c8be69423ca43543eab7e8fb2fcafd2a4
```
We can also reference files that are described in other manifests, for example the file `image.jpg` referenced in a manifest with hash `efcb8a7a1f0632c42c55b0c84f3353c9401724bab510df28c71d2ff9be9aef2a` will be referenced as `efcb8a7a1f0632c42c55b0c84f3353c9401724bab510df28c71d2ff9be9aef2a/image.jpg`

An in depth explanation of manifests can be found [here](https://github.com/ethereum/go-ethereum/wiki/URL-Scheme)


# Considerations

* Attaching SWARM to geth (Blockchain) is only necesary if [SWAP](https://github.com/ethersphere/swarm/wiki/Swap) or the [ENS](https://ens.domains/) is going to be used. 
* Right now (03/08/17) the propagation of a file through the network takes quite some time, this is a transitory issue and should disapear once the network settles down
