![title](pic/title.png)


[slides](https://docs.google.com/presentation/d/1iPXMhChZSoxUF0wdVdqB6OcjMytnGAiv8M9-n3hED6o/edit?usp=sharing), [video](https://www.dropbox.com/s/5pyxaks8ntxaara/Non-overlapping%20algorithm.mp4?dl=0), and [final report](https://www.dropbox.com/s/rwqy49ubcqcq2e6/Cloud_Computing_Final_Report.pdf?dl=0)

## Motivation
The high capacity mentioned in the [Jellyfish paper](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final82.pdf) inspires us to explore its potential to tackle burst flow. We want to maximize the average throughput at the expense of tolerable latency.


## Progress

We put forward and implemented a new routing algorithm named **k-non-overlapping Path**, which guarantees all links on paths from A to B have no overlapping.

This repository implements and tests non-overlapping path algorithm in Jellyfish Network.

![demo](pic/demo.jpg)

### Path Diversity

Jellyfish network with 50 switches (8 for peer switches and 1 for host). Based on Jellyfish Paper Figure 9: Inter-switch link’s path count in ECMP, k-shortest-path and k-non-overlapping routing for random permutation traffic at the server-level on a Jellyfish of 50 servers. For each link, we count the number of distinct paths it is on. Each network cable is considered as two links, one for each direction.

Check our [Google colab link](https://colab.research.google.com/drive/1Gr3CdQoGaxquAKJgxV-6qfBktk2h_r7p?usp=sharing) to reproduce this plot!

![plot](pic/diversity.png)


### Average Throughput per Server

Jellyfish network with 50 switches
(8 ports connecting peer switches and 1 for host). Links between switches are 10 Mbps.

The result is subject to lots of factors and may differ in another machine and/or randomness. 
See next section to reproduce yourself!

| Routing Algorithm | 10 Servers / 10 Clients | 15 Servers / 15 Clients | 20 Servers / 20 Clients | 25 Servers / 25 Clients |
| ----------------- | ----------------------- | ----------------------- | :---------------------: | :---------------------: |
| 8-Non-overlapping | 38.68  Mbps (↑13.2%)    | 32.7  Mbps (↑8.6%)      |   29.75 Mbps (↑19.1%)   |   25.35 Mbps (↑16.0%)   |
| 8-Shortest-Paths  | 34.19  Mbps             | 30.1  Mbps              |       24.98 Mbps        |       21.83 Mbps        |


![plot](pic/comparsion.png)

You can check [experiment_data.xlsx](./experiment_data.xlsx) for concrete data.

## Build


### Creating Environment

The most recommended way to reproduce it is using [google computer engine](https://cloud.google.com/compute); We provided a public image for the whole test environment. Simply run in [google cloud shell](https://cloud.google.com/shell):

```
gcloud compute instances create [VM Name] --image non-overlapping --image-project winter-cargo-272015
```

and then you are all set. 

Any questions, please check documents on [Creating an instance with an image shared with you](https://cloud.google.com/compute/docs/instances/create-start-instance#sharedimage). You may want to create a high-performance VM for the following experiments.

Alternatively, if you would like to build it from scratch, you should install [Mininet](https://github.com/mininet/mininet) first, clone the repo and execute `bash setup.sh`. **Python 2** is used for our code -- all lib we use is only in Python 2. During this process, you may encounter some problems like `ModuleNotFoundError: No module named 'networkx'`. Deal with it yourself and good luck to you.

###  Test Prep
After finishing configuration, 
```
ssh mininet@[Your VM IP]
```
the password is also `mininet`.

Then run
```
cd jellyfish
sudo git fetch origin master
sudo git reset --hard origin/master
sudo bash setup.sh
```
to assure you get and install the lastest code. Each time you change [`JellyfishRouting`](https://github.com/Lw-Cui/Non-overlapping-Path-in-Jellyfish/blob/master/ripl/ripl/routing.py#L49) or [`JellyfishTopo`](https://github.com/Lw-Cui/Non-overlapping-Path-in-Jellyfish/blob/master/ripl/ripl/dctopo.py#L221), you have to rerun `sudo bash setup.sh`.

To generate topology and test script, run 
```
python build_topology.py --node=50 --port=8
python tcp_test.py --node=50 --port=8 --test=20 > test.sh
```
The default settings is 50 switches (8 ports connecting peer switches and 1 for host); 20 senders and 20 receivers.

![](pic/experiment.png)
### Run Test

Start controller first:
```
pox/pox.py riplpox.riplpox --topo=jelly,50,8,rrg_8_50 --routing=jelly,unique_rrg_8_50 --mode=reactive
```
Then in a new shell, run
```
sudo mn --custom ripl/ripl/mn.py --topo jelly,50,8,rrg_8_50 --link tc --controller=remote --mac
```
run command 
```
source test.sh
```
in mininet prompt.

Wait until `result/output.txt` exists (about half minute), then run
```
python data_process.py results/output.txt
```
Result is like this:

```
[SUM]  0.0-10.7 sec  41.9 MBytes  32.7 Mbits/sec
[SUM]  0.0-11.3 sec  57.6 MBytes  42.9 Mbits/sec
[SUM]  0.0-11.0 sec  47.4 MBytes  36.0 Mbits/sec
[SUM]  0.0-11.2 sec  39.8 MBytes  29.8 Mbits/sec
[SUM]  0.0-11.3 sec  40.9 MBytes  30.3 Mbits/sec
[SUM]  0.0-11.9 sec  36.0 MBytes  25.4 Mbits/sec
[SUM]  0.0-11.3 sec  46.5 MBytes  34.4 Mbits/sec
[SUM]  0.0-11.1 sec  27.4 MBytes  20.6 Mbits/sec
[SUM]  0.0-11.3 sec  29.0 MBytes  21.5 Mbits/sec
[SUM]  0.0-12.4 sec  41.0 MBytes  27.6 Mbits/sec
[SUM]  0.0-11.3 sec  55.1 MBytes  40.8 Mbits/sec
[SUM]  0.0-11.8 sec  37.1 MBytes  26.5 Mbits/sec
[SUM]  0.0-11.4 sec  34.5 MBytes  25.4 Mbits/sec
[SUM]  0.0-11.2 sec  24.6 MBytes  18.4 Mbits/sec
[SUM]  0.0-11.5 sec  28.9 MBytes  21.1 Mbits/sec
[SUM]  0.0-11.0 sec  38.8 MBytes  29.5 Mbits/sec
[SUM]  0.0-11.0 sec  60.9 MBytes  46.4 Mbits/sec
[SUM]  0.0-11.1 sec  23.1 MBytes  17.4 Mbits/sec
[SUM]  0.0-11.3 sec  54.5 MBytes  40.6 Mbits/sec
[SUM]  0.0-10.6 sec  57.0 MBytes  45.1 Mbits/sec

[REPORT] The average throughput per server is 30.62 Mbits/sec
```

Then the average throughput per server is 30.62 Mbps.
Again, you can check [experiment_data.xlsx](./experiment_data.xlsx) for concrete data.

### Comparsion with ksp

**Stop** pox & mininet and **delete** `result/output.txt` first; old routing table in switches, states in controller may influence result.

This time start controller, change `unique_rrg_8_50` to `ksp_rrg_8_50`:
```
pox/pox.py riplpox.riplpox --topo=jelly,50,8,rrg_8_50 --routing=jelly,ksp_rrg_8_50 --mode=reactive
```
and do things again.

## Contribution

* Liwei Cui: Reproduced Jellyfish network; Implemented and benchmarked routing algorithms; Analyzed and visualized data
* Mou Zhang: Performed experiments under various circumstances; Migrated the environment to the Cloud
* Yifeng Yin: Provided mathematics analysis


## Acknowledge

* [Mininet](https://github.com/mininet/mininet) library for network emulation
* [Pox](https://github.com/noxrepo/pox) library for OpenFlow controller
* [RipL](https://github.com/brandonheller/ripl) library for simplifying data center code
* [RipL-POX](https://github.com/brandonheller/riplpox) library for controller built on RipL
* Austin Poore and Tommy Fan’s [open-source code](https://github.com/lechengfan/cs244-assignment2) for inspiration to reproduce Jellyfish and k-shortest-paths routing

## Reference
* Singla A, Hong C, Popa L, Godfrey PB. Jellyfish: Networking data centers randomly. *2012:225-238.*
* Singh A, Ong J, Agarwal A, et al. Jupiter rising: A decade of Clos topologies and centralized control in google's datacenter network. *ACM SIGCOMM computer communication review​. 2015;45(4):183-197.*
* Bollobás, B., & De La Vega, W. F. (1982). The diameter of random regular graphs. *Combinatorica, 2(2), 125-134.*
