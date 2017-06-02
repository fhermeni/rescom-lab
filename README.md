# Vicc project: homemade VM Schedulers

This project aims at developing different VM schedulers for a given IaaS cloud. Each scheduler will have meaningful properties for either the cloud customers or the cloud provider.

The implementation and the evaluation will be made over the IaaS cloud simulator [CloudSim](http://www.cloudbus.org/cloudsim/). The simulator will replay a workload extracted from Emulab, on a datacenter having realistic characteristics (when it was created). 

The simulated datacentre is made of:
- 400 HP Proliant Ml111G4 nodes. 2 CPUs (aka. PES) @ 1860 Mips, 4 GB RAM
- 400 HP Proliant Ml111G5 nodes. 2 CPUs (aka. PES) @ 2660 Mips, 4 GB RAM

The workload consists of 10 random days of Planetlab.
- 1052 VMs
- 4 VM templates:
  - 1x 500 MIPS, 613 MB RAM
  - ...
  - 1x 2500 MIPS, 870 MB RAM
- The Mips requirements for the VMs change over the time
- All the VMs are launched at the same moment
 
#### Some usefull resources:

- CloudSim [FAQ](https://code.google.com/p/cloudsim/wiki/FAQ#Policies_and_algorithms)
- CloudSim [API](http://www.cloudbus.org/cloudsim/doc/api/index.html)
- CloudSim [source code](cloudsim-3.0.3-src.tar.gz)
- CloudSim [mailing-list](https://groups.google.com/forum/#!forum/cloudsim)

#### Setting up the environment

You must have a working Java 7 + [maven](http://maven.apache.org) environment to develop and Git to manage the sources. No IDE is required but feel free to use it.

1. clone this repository. The project directory is organized as follow:
```sh
$ tree
 |- src # the source code
 |- repository # external dependencies
 |- planetlab # the workload to process
 |-cloudsim-3.0.3-src.tar.gz # simulator sources
 \- pom.xml # maven project descriptor
```
2. check everything is working by typing `mvn install` in the root directory
3. Integrate the project with your IDE if needed

#### How to test

`fr.unice.vicc.Main` is the entry point. It can be launch from an IDE or using the command `mvn compile exec:java`.

```sh
Usage: Main scheduler [day]
```

- `scheduler` is the identifier of the scheduler to test, prefixed by `--`.
- `day` is optional, it is one of the workload day (see folders in `planetlab`). When `all` is indicated all the days are replayed sequentially.

By default, the output is written in a log file in the `logs` folder.

If you execute the program through `mvn exec:java`, then the arguments are provided using the 'sched' and the 'day' properties.

- To execute the simulator using the `naive` scheduler and all the days:
`mvn compile exec:java -Dsched=naive -Dday=all`
- to replay only day `20110303`: `mvn compile exec:java -Dsched=naive -Dday=20110303`

For this project, you have to develop various VM schedulers.
To integrate your schedulers within the codebase, you will have to declare your schedulers inside the class `VmAllocationPolicyFactory`.

## A first fit scheduler to warm up

This scheduler aims only at discovering the CloudSim API. This scheduler simply places each `Vm` to the first `Host` having enough free resources (CPU and memory). The skeleton is in the class `NaiveVmAllocationPolicy.java`. It is used from the CLI using the `naive` flag.

The 2 `allocateHostForVm` are the core of the Vm scheduler. They are called by the simulator each time a Vm is submitted.

1. Implementing `allocateHostForVm(Vm, Host)` is straighforward as the host is forced. To allocate the Vm on a host look at the method `Host.vmCreate(Vm)`. It allocates and returns true iff the host has sufficient free resources. The method `getHostList()` from `VmAllocationPolicy` returns the datacenter nodes. Use the `hoster` attribute to map the host associated to each Vm.
1. Implement `allocateHostForVm(Vm)`, the main method of this class. A simple _first fit_ algorithm does the job.
1. Test your simulator on a single day. If the simulation terminates successfully, all the VMs have been scheduled and the provider revenues is displayed.
1. Test the simulator on all the days. For future comparisons, save the daily revenues and the global one. At this stage, it is ok to have penalties due to SLA violations
	
## Alternative schedulers

Below are different request for features that require to develop alternative schedulers.
All the scheduler are independent. They are not sorted by difficulty. Implement those you are interested in. 

### Fault-tolerance for replicated applications
Let consider the VMs run replicated applications. To make them fault-tolerant to node failure, the customer expects to have the replicas running on distinct hosts.

Implement a new scheduler (`antiAffinity` flag) that places the Vms with regards to their affinity. In practice, all Vms with an id between [0-99] must be on distinct nodes, the same with Vms having an id between [100-199], [200-299], ... .
 
### Load balancing

Develop an algorithm based on a _worst fit_ heuristic (`worstFit` flag) that balances with regards to both RAM and mips. There is different solutions to consider the two dimensions. Be pragmatic, test alternative implementations and try to observe the consequences in terms of SLA violations.

### Performance satisfaction

Implement a scheduler that ensures there can be no SLA violation. Remember the nature of the hypervisor in terms of CPU allocation and the VM templates. The scheduler is effective when you can successfully simulate all the days, with the `Revenue` class reporting no re-fundings due to SLA violation.

For a practical understanding of what a SLA violation is in this project, look at the `Revenue` class. Basically, there is a SLA violation when the associated Vm is requiring more MIPS it is possible to get on its host. If the SLA is not met then the provider must pay penalties to the client.

### Energy-efficient schedulers

Develop a scheduler that reduces the overall energy consumption without relying on VM migration. The resulting simulation must consumes less energy than all the previous schedulers.

### Greedy scheduler

Develop a scheduler that maximizes revenues. It is then important to provide a good trade-off between energy savings and penalties for SLA violation. Justify your choices and the theoretical complexity of the algorithm
