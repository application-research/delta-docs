# Delta Dataset Manager: Getting Started

Interested in setting up Delta Dataset Manager as a Data Preparer/Onboarder? Follow this guide!

This guide will walkthrough the set-up of Delta and DDM in a standalone deployment, then show you how to add a dataset, content, wallets, SPs, and finally create replications.


## Installation

### Prerequisites
- The following should work on macOS, as well as any Linux distro 
- Prerequisites: Go v1.19+, Rust v1.65+ OR Docker/Docker Compose
- Minimal CPU/RAM/Disk - this is a very lightweight application and should be able to run on a small VM

### Install (Baremetal)
Follow the instructions in the quick-start section in [Delta Standalone Repository](https://github.com/application-research/delta-standalone#quick-start)

Set-up can be done in a few minutes with just a single command. 

Follow the instructions as indicated and save your Delta API key.


### Install (Docker Compose)
Coming soon!


## Set-up

### 0. Verify Installation
Verify that the set-up was successful by running the following command:

```bash
ps -aux | grep delta
```

You should see **three** Delta processes running: `delta`, `delta-dm` and `node` (for NextJS/React Frontend) 

```shell
root     1980744  0.0  0.0   6244  2408 pts/11   S+   22:57   0:00 grep delta
root     3860935  9.9  0.1 11331296 2095992 ?    Sl   Apr07 605:54 ./delta/delta daemon --mode=standalone
root     3861546  0.0  0.0 2869124 30644 ?       Sl   Apr07   0:39 ./delta-dm/delta-dm daemon
root     3861647  0.0  0.0 11156624 56728 ?      Sl   Apr07   0:06 node ./delta-standalone/delta-nextjs-client/node_modules/.bin/next
```

Verify that you can access to the DDM UI by visiting `http://localhost:3000` on the server where you installed Delta.

Paste in the Delta API key, and specify the URL where Delta-DM is running (same server, default port is 1314).

Source `delta.env` to set the environment variables if you'd like to use the CLI.


### 1. Create a Dataset

**By UI**

Create a dataset by clicking on the `+ New Dataset` button on the top left corner of the DDM UI.
Fill in the details of the dataset and click `Create Dataset`.

<img src="./assets/ddm-add-dataset.png" width=500/>

**By CLI**
```bash
./delta-dm dataset add --name radiant-earth-ml --replication-quota 6 --duration 540 --unsealed true --indexed true
```


### 2. Add wallet 
**By CLI**
Wallets must be added via the DDM CLI for security reasons.

```bash
# (On a machine where a wallet is accessible in Lotus)
> ./delta-dm wallet import --hex $(lotus wallet export f1mmb3lx7lnzkwsvhridvpugnuzo4mq2xjmawvnfi)
```

### 2a. Associate wallet with dataset
**By UI**
Find the wallet in the DDM UI, and click the **Associate With Dataset** button below it.

Select the dataset you added previously, and click **Apply**

<img src="./assets/ddm-associate-wallet.png" width=400/>

**By CLI**
```bash
./delta-dm wallet associate --address f1mmb3lx7lnzkwsvhridvpugnuzo4mq2xjmawvnfi --dataset radiant-earth-ml 
```

### 3. Add Content to Dataset
There are a number of ways to associate content with a dataset.

Note: Piece CIDs / CommP's must be globally unique in DDM - thus, you don't have to worry about collisions when importing content from other CAR generation tools - the duplicates will simply not be added. This allows you to progressively import content from those tools as they are generated, without needing to "diff" the CID list.

#### Importing from other CAR generation tooling
**Singularity Content**
Follow the instructions in the [Singularity Import Guide](https://github.com/application-research/delta-dm/blob/main/docs/singularity-import.md))

**Ptolemy Content**
Follow the instructions in [Ptolemy Import Guide](https://github.com/application-research/delta-dm/blob/main/docs/ptolemy-import.md)

#### Importing from CSV/JSON file
**By UI**
Click on the `Datasets` button on the top left corner of the DDM UI. Click the `Attach Content` button below the dataset you created previously. 

<img src="./assets/ddm-attach-content-button.png" width=300/>

After that, you can either click Attach Content or drag and drop the file into the UI.

JSON Format: Consult the [DDM Format](https://github.com/application-research/delta-dm/blob/main/docs/api.md#delta-dm-format) spec for a description of the JSON schema you need to follow. 

**By CLI**
```bash
# json file
./delta-dm content import --dataset radiant-earth-ml --json ./content.json

# OR csv file
./delta-dm content import --dataset radiant-earth-ml --csv ./content.csv
```


### 4. Add Provider(s)
**By UI**
Click on the `+ Add Provider` button on the top left corner of the DDM UI. Enter the SP's address (f0######), and an optional friendly name.

<img src="./assets/ddm-add-provider.png" width=400/>

**By CLI**
```bash
./delta-dm provider add --address f011111 --name "My Provider"
```

### 4a. Associate Provider with datasett
TODO - UI

**By CLI**
> optionally, enable self-service
```bash
./delta-dm provider modify --id f011111 --allowed-datasets radiant-earth-ml --allow-self-service on
```

### 5. Create Replication

**By UI**
