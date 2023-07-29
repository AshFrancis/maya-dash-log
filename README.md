# Maya Dash Log (Ash Francis)

[Trello](https://trello.com/c/QDnbCI5h/192-maya-protocol-integration)

[Maya Github](https://github.com/mayachain)

[Maya Gitlab](https://gitlab.com/mayachain)

[Maya Whitepaper](https://assets.website-files.com/62a14669b65c6aeed054f32e/62b3648681be198a7fd33120_MAYA%20PROTOCOL%20WHITE%20PAPER_.pdf)

[Batch One](#batch-one)

[Batch Two](#batch-two)

[AWS Bill July](#aws-bill-july)

## Batch One

```
18.04.2023 Tuesday      4h
19.04.2023 Wednesday    6h
20.04.2023 Thursday     1h
22.04.2023 Saturday     2h
23.04.2023 Sunday       6h
08.05.2023 Monday       1h
11.05.2023 Thursday     1h
17.05.2023 Wednesday    1h
18.05.2023 Thursday     1h
19.05.2023 Friday       2h
20.05.2023 Saturday     3h
21.05.2023 Sunday       1h
25.05.2023 Thursday     1h
26.05.2023 Friday       1h
Total                   31h
```

### 18.04.2023 Tuesday 4h

Alex is away from 19th April - 30th May, for both this reason and redundancy I need to ensure full access to the Genesis node. Our first approach is to have a tunnel into his machine and then run the commands as him. To do this we set up a ZeroTier network, a new key pair and some other bits. 

Naturally, come the evening Alex has gone to the airport and Maya dumps me at the deep end:

```
the first migration of fund was actually signed, but not observed we have some solvency problems with the asgard vaults. We are going to reset the bifrost cache and set a start a block height for each of the external chains before those migrations took place.
```

Backup everything, apply some patches, remove the blockchain data for BTC, ETH & THOR, update. Needs to be done 1-by-1 to ensure the network doesn't halt due to not enough signers. 

SSH fell over... I can't access Alex's laptop so can't patch. Letting others run ahead me whilst attempting to resolve. 

Alex managed to remote desktop via his phone and get the tunnel open, we had a call and worked through it to ensure everything went swimmingly. 

### 19.04.2023 Wednesday 6h

Access worked today, though it took several attempts to respond to pings; perhaps just ZeroTier going into an idle state?

I had to run some commands (thanks Maya team!) to get myself properly setup for kubectl... and then it's off to the races.

Pretty major happenings on the Maya side, we started the day with a (mostly) smooth update to 103.1, where we were all naturally jailed. Next up... we had an issue, THOR had a configuration error in Mayanode which needed fixing, we also all need to co-ordinate all updates, with just 6 genesis nodes, if 2-3 of them are not running (during update, for example) then everything will grind to a halt (and cannot be easily fixed)... some slight terrible foreshadowing here!

So, update plan:

```
Self-coordination update
Make sure no one is updating
Notify in the group you are updating
make backup-> mayanode and make backup -> bifrost
make update (Make sure the diff only shows the version change)
Wait for all services to be available again make wait-ready
make set-version if version wasn't updated automatically
Send make status update to this group
Apply patch: git apply debug-bifrost.patch
make debug
# rm -rf /root/data/observer/THOR
# exit
Wait for bifrost to be available again make wait-ready
Send make logs -> bifrost  to the group
Notify the group you have finished and someone else can proceed
```

Pretty uneventful, I went 4th, success:

```
MAYANode version updated
   __  ________  _____   _  __        __   
  /  |/  /   \ \/ /   | / |/ /__  ___/ /__ 
 / /|_/ / /| |\  / /| |/    / __\/ __ / -_)
/_/  /_/_/ |_|/_/_/ |_/_/|_/\___/\_,_/\__/ 

ADDRESS     maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg
IP          18.221.183.211
VERSION     1.103.1
STATUS      Active
BOND        20,002.23
REWARDS     0.00
SLASH       779
PREFLIGHT   {
  "status": "Ready",
  "reason": "OK",
  "code": 0
}

API         http://18.221.183.211:1317/mayachain/doc/
RPC         http://18.221.183.211:27147
MIDGARD     http://18.221.183.211:8080/v2/doc

CHAIN      SYNC       BEHIND         TIP       
MAYA       100.000%   0              571,706   
THOR       100.000%   0              10,485,743
ETH        100.000%   0              17,083,115
ETH (beacon slot)  100.000%   0              6,259,445 
BTC        100.000%   0              786,156   

=> Detected active validator MAYANode on mainnet named mayanode-stagenet
```

Annnd then it broke:

```
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x563702]
```

Maya team created a patch for me, after a bit of back and forth bug resolution, we got there.

Then it broke, again. However, this was nothing to do with us or even Maya... THORChain went down, which meant RUNE was dead in the water... this is a problem for tomorrow.


### 20.04.2023 Thursday 1h

Update time! Access worked and running the update was uneventful. THOR reported an error, but this eventually resolved itself (related to sync) took a few hours to catch up to the tip:  

```
   __  ________  _____   _  __        __   
  /  |/  /   \ \/ /   | / |/ /__  ___/ /__ 
 / /|_/ / /| |\  / /| |/    / __\/ __ / -_)
/_/  /_/_/ |_|/_/_/ |_/_/|_/\___/\_,_/\__/ 

ADDRESS     maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg
IP          18.221.183.211
VERSION     1.103.2
STATUS      Active
BOND        20,057.51
REWARDS     0.00
SLASH       1,083
PREFLIGHT   {
  "status": "Ready",
  "reason": "OK",
  "code": 0
}

API         http://18.221.183.211:1317/mayachain/doc/
RPC         http://18.221.183.211:27147
MIDGARD     http://18.221.183.211:8080/v2/doc

CHAIN      SYNC       BEHIND         TIP       
MAYA       100.000%   0              585,780   
THOR       99.613%    -40,654        10,497,054
ETH        100.000%   0              17,089,377
ETH (beacon slot)  100.000%   -1             6,265,856 
BTC        100.000%   0              786,276   
```

Okay got there, ~5-8hours for syncing isn't terrible. Once all the nodes are synced then THOR trading & signing will resume.

### 22.04.2023 Saturday 2h

THOR seems to be throwing some issues, the Maya team have asked us for status & logs. 

We all seem to be throwing this kind of error:
```  
"message": "failed to get block: rpc error: code = ResourceExhausted desc = grpc: received message larger than max (4516436 vs. 4194304)"
```

Maya quickly identified the issue (TC sent a larger message size than expected and exceeded a grpc config limit) and quickly patched (minor config change to increase the limit).

Time for a coordinated 1-by-1 patch. Everything is going well until my turn... 

No access... can't ping his pc, can't ssh in... this is not going to work long term. Alex is unavailable, he is attempting to resolve remotely but the reality is I need direct access... tbc.


### 23.04.2023 Sunday 6h 

Today was mostly spent on AWS cli, to get permissions to the kubernetes cluster. My access issues seemed to stem from this fun issue:

```
When an Amazon EKS cluster is created, the IAM principal that creates the cluster is added to the Kubernetes RBAC authorization table as the administrator (with system:masters permissions). Initially, only that principal can make calls to the Kubernetes API server using kubectl . For more information, see Enabling IAM principal access to your cluster. If you use the console to create the cluster, make sure that the same IAM credentials are in the AWS SDK credential chain when you are running kubectl commands on your cluster.
```

My next line of enquiry was whether it was configured via a VPC, so I began looking into that. I ran into a ton of access denied errors, despite attempting with either my admin account or our root account. I was unable to assume roles and despite looking at the security groups couldn't see quite where I could gain access. 

All told I spent several hours just banging my head against a wall, better understanding how roles work within AWS in a kubernetes context and so on... here's a dump of my problems:

```
ash@Ashs-MacBook-Pro ~ % aws eks update-kubeconfig --region us-east-2 --name dash-maya
Updated context arn:aws:eks:us-east-2::cluster/dash-maya in /Users/ash/.kube/config
ash@Ashs-MacBook-Pro ~ % kubectl get pods
error: You must be logged in to the server (Unauthorized)
```

Tried giving policy permissions, tried assuming role... no dice.

```
An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam:::user/Ash is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam:::role/dash-maya-cluster
```

Found this in amongst the stackoverflow threads and such I was browsing:

```
To add access to other aws users, first you must edit ConfigMap to add an IAM user or role to an Amazon EKS cluster.

You can edit the ConfigMap file by executing: kubectl edit -n kube-system configmap/aws-auth, after which you will be granted with editor with which you map new users.
```

So I tried editing my aws cli profiles file to try and resolve it:

```
[profile maya]
role_arn = arn:aws:iam:::role/mayanode-eks-node-group
source_profile = default
```

No dice, still cant assume that role, even when I am a root user or an admin... 

I managed to gain access via editing some security groups:

```
/code/node-launcher get pods
no resources found in default namespace
```

Ah.... kubectl get pods --all-namespaces returned something. 

This is  impossible.

Okay turns out, impossible is right before solving it. 

I managed to use the hosted AWS console to act as Alex and add myself as an authorized user in the ConfigMap file... 

We have access & status!

```  
   __  ________  _____   _  __        __   
  /  |/  /   \ \/ /   | / |/ /__  ___/ /__ 
 / /|_/ / /| |\  / /| |/    / __\/ __ / -_)
/_/  /_/_/ |_|/_/_/ |_/_/|_/\___/\_,_/\__/ 

ADDRESS     maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg
IP          18.221.183.211
VERSION     1.103.2
STATUS      Active
BOND        20,270.67
REWARDS     0.00
SLASH       1,639
PREFLIGHT   {
  "status": "Standby",
  "reason": "node account does not meet min version requirement: 1.103.2 vs 1.103.3",
  "code": 1
}

API         http://18.221.183.211:1317/mayachain/doc/
RPC         http://18.221.183.211:27147
MIDGARD     http://18.221.183.211:8080/v2/doc

CHAIN      SYNC       BEHIND         TIP       
MAYA       100.000%   0              633,630   
THOR       100.000%   0              10,540,661
ETH        100.000%   0              17,110,878
ETH (beacon slot)  100.000%   0              6,287,625 
BTC        100.000%   0              786,711   
```

Okay, update time!

Update went without a hitch, logs all look normal, received confirmation from the Maya team, we are now successfully running 103.3! 

### 08.05.2023 Monday 1h

Everything was so quiet. However, it seems something has fallen over, so we're all sharing our log files. 

Update time, update, restart thornode-daemon and provide logs. I did this right away and was quickly signing again [here](https://mayanode.mayachain.info/mayachain/tx/942FC7D59C7EF14F11E438E91D3D2CD1E453CA5DAFAC9E20C79E83B703382820/signers):

```
		],
		"block_height": 10765347,
		"signers": [
			"maya1v7gqc98d7d2sugsw5p4pshv0mm24mfmzgmj64n",
			"maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg", <--- THATS US
			"maya1vm43yk3jq0evzn2u6a97mh2k9x4xf5mzp62g23",
			"maya12amvthg5jqv99j0w4pnmqwqnysgvgljxmazgnq"
		],
		"observed_pub_key": "mayapub1addwnpepqwuwsax7p3raecsn2k9uvqyykanlvhw47asz836se2h0nyg6knug6n9hklq",
		"finalise_height": 10765347
```

This actually went very well! 


### 11.05.2023 Thursday 1h

Another day another update, nothing remarkable here, just the usual process as well as monitoring, sharing logs, at first something looked off but the node sync resolved whatever that was. All good! 

### 17.05.2023 Wednesday 1h 

Small update (Midgard), got this:

```
warning: couldn't attach to pod/reset-midgard, falling back to streaming logs: unable to upgrade connection: container reset-midgard not found in pod reset-midgard_mayanode-stagene
```

Seems okay though?

```
   __  ________  _____   _  __        __   
  /  |/  /   \ \/ /   | / |/ /__  ___/ /__ 
 / /|_/ / /| |\  / /| |/    / __\/ __ / -_)
/_/  /_/_/ |_|/_/_/ |_/_/|_/\___/\_,_/\__/ 

ADDRESS     maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg
IP          18.221.183.211
VERSION     1.103.3
STATUS      Active
BOND        20,806.21
REWARDS     0.00
SLASH       2,565
PREFLIGHT   {
  "status": "Standby",
  "reason": "node account is jailed until block 1018330: failed to perform keysign",
  "code": 1
}

API         http://18.221.183.211:1317/mayachain/doc/
RPC         http://18.221.183.211:27147
MIDGARD     http://18.221.183.211:8080/v2/doc

CHAIN      SYNC       BEHIND         TIP       
MAYA       100.000%   0              1,018,296 
THOR       100.000%   2              10,893,144
ETH        100.000%   0              17,283,307
ETH (beacon slot)  100.000%   -1             6,462,574 
BTC        100.000%   0              790,245   
```

and my node has stopped signing... tbc.

### 18.05.2023 Thursday 1h 

So the Maya theory is that some blocks were skipped an unobserved, which is causing longer term issues. So we're putting our nodes in recovery... 

Update time, more syncing time and general monitoring, giving logs and so on. All looks good so far...


### 19.05.2023 Friday 2h 

Update time, coordinated (1 by 1). 

Everything went wrong. Updates are required to be done 1 by 1, once the network is much larger this isn't a problem because its very unlikely 34 of 100 nodes would all update at exactly the same time... however when you only have 6 genesis nodes, 2-3 going offline (for updates or otherwise) means the network halts. This means... it's rollback time. 

Everyone halts, provides logs, status, creates backups... Maya team working hard to a resolution... 

```
kubectl scale deploy/bifrost --replicas=0 --timeout=5m
Error from server (NotFound): deployments.apps "bifrost" not found
```

Maya team help resolve this error. 

It's called a day, tomorrow should be interesting. 

### 20.05.2023 Saturday 3h 

Lots of work today on getting the network running again, we had to do multiple updates and restarts and so on, provide logs, help explore the issues, all the fun stuff!

We get everything back running again... we're all going to be a lot more careful on the coordinated update front from now on. 

The running version is not the updated version that we were updating to when everything died. Tomorrow we will be updating to v104, one by one, VERY coordinated...


### 21.05.2023 Sunday 1h 

Went to IKEA, brought the laptop with me just in case... and yep, its my update time. Pretty painless. Our node is good, I'm regularly monitoring. The other Node Operators also update and we're back running, everyone can relax!

### 25.05.2023 Thursday 1h

Multiple nodes having an issue with BTC... Maya team confirm with me that my error matches theirs.. tbc

### 26.05.2023 Friday 1h

Update to fix the above issue. Smooth & all sorted, monitored but uneventful! 


## Batch Two

```
13.06.2023 Tuesday    1.5h
27.06.2023 Tuesday    1h
08.07.2023 Saturday   1h 
10.07.2023 Monday     0.5h 
18.07.2023 Tuesday    1h
19.07.2023 Wednesday  0.5h 
23.07.2023 Sunday     0.5h
24.07.2023 Monday     0.5h
25.07.2023 Tuesday    0.5h
26.07.2023 Wednesday  1h
28.07.2023 Friday     2h 
Total                 10h
```


### 13.06.2023 Tuesday 1.5h 

Resolving issues around inability to push dashd-go updates without DCG approval, THORChain originally wanted it in DCGs repo for security reasons but this means a long update loop and hoops to jump. Eventually we just had Maya host the repo as they trust us (for good reason!) and Alex is a maintainer. 


### 27.06.2023 Tuesday 1h

Stagenet issues with chainlock and ignored blocks and reconsiderations, we approached this with understanding that the scanner was skipping blocks that didnt have a chainlock and then processing child blocks with a chainlock without reconsidering the ancestor. We had to review the various dips and responses from nodes to understand how a chainlock is kept in memory, responses of ancestors and so on without quite being able to replicate and test. The end implementation was some kind of replay of prior chain... this seemed okay at the time but I suggested a simpler approach (see next log) that ended up being adopted before launch.

### 08.07.2023 Saturday 1h

Reviewed the chainclient ahead of Dash launch and investigated and subsequently made a suggestion to use getbestchainlock instead of chainheight as a simple approach for handling blockheight with regards to orphan chains and ancestor chainlock application instead of recursive checks with exponential backoff.

### 10.07.2023 Monday 0.5h 

Thornode-daemon update and monitoring

### 18.07.2023 Tuesday 1h

Investigating issue with Dash on stagenet, turns out it was due to chainclient expecting an individual chainlock per tx on a getblock request instead of the top-level chainlock: true applying to all txs in the block. 


### 19.07.2023 Wednesday 0.5h 

Thornode-daemon update and monitoring


### 23.07.2023 Sunday 0.5h

Another update of thornode-daemon image... there was an issue and a halt at THORChain (and also Maya RUNE) so this is to fix that...


### 24.07.2023 Monday 0.5h

Did a bit of a cost investigation, found a couple of inactive reserved volumes to delete (should save ~$100/month) 

### 25.07.2023 Tuesday 0.5h

Addition of Dash chainclient ready for launch, some issues to resolve, debugging, monitoring and so on.


### 26.07.2023 Wednesday 1h

Dash launch, fixing some node issues, resets, config changes, monitoring.


### 28.07.2023 Friday 2h 

Investigating ignored transaction, initial theories are:
- op_return malformed or on wrong vouts
- block was ignored due to how we handle chainlocks
- unexpected response from node on block (ancestor chainlock)

Turned out it was that the op_return was on the wrong vout due to a manually crafted transaction. Found another couple of small transactions like this, Maya team is informing Dash community on manual txs or to wait for more front-ends to add support.

Looked into current hosting bill, forecasted to $2300,  I will to continue to investigate cost savings via migration, greater efficiency or otherwise here... 

Also updated all this log... how meta.

### AWS Bill July
![aws bill](https://i.imgur.com/QsauHMx.png)


