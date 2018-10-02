# cgroups

cgroups are a linux kernel feature that allow for placing certain restrictions (memory usage, cpu usage, etc.) on groups of processes

## Controllers

TODO document different and define the `controllers`
- `cpu`
- `memory`
- `blkio`
- ...
-
## How to use
> This instructions are tested on Ubuntu 18.04

First step is to install the `cgroup-tools` package, this package installs the following tools and their documentation (listed using `dpkg-query -L cgroup-tools`)
```
cgclassify
cgcreate
cgdelete
cgexec
cgget
cgset
cgsnapshot
lscgroup
lssubsys
```
1. Create a new cgroup using `cgcreate` and define the `controllers` that are going to be applied. Simple syntax `cgcreate -g <controller>[,controller]*:<cgroup-name>`

  Ex: `sudo cgcreate -g cpu,memory:my-first-container`

This will create a folder named `<cgroup-name>`  in `/sys/fs/cgroup/<controller>/` for each controller specified

2. Configure the controllers, each of the controller have config files that control how much of each resource is given to the group.
  - `/sys/fs/cgroup/memory/<cgroup-name>/memory.limit_in_bytes`: memory limit
  - `/sys/fs/cgroup/cpu/<cgroup-name>/cpu.cfs_period_us`:  specifies a period of time in microseconds (µs, represented here as "us") for how regularly a cgroup's access to CPU resources should be reallocated.
  - `/sys/fs/cgroup/cpu/<cgroup-name>/cpu.cfs_quota_us`: the amount of time in microseconds that ALL tasks in a cgroup can take up during one `cfs_quota_us`.

3. Add process(es) to the cgroup vis `cgexec`. Simple syntax `cgexec -g <controller>[,controller]*:<cgroup-name>`.

  Ex: `sudo cgexec -g cpu,memory:my-first-container sleep 16`
  Ex: `sudo env "PATH=$PATH" cgexec -g cpu,memory:my-first-container echo $PATH` (this preserves path)

## Examples
To allow a given process to only consume X% of the cpu, set `cfs_period_us` to `1000000` and set `cfs_quota_us` to `X00000`

