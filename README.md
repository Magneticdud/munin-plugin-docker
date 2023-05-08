# munin-plugin-docker

The *docker_cpu* and *docker_memory* munin plugins monitor container CPU and memory usage over time, respectively.

I just edited the code slightly to let read the performance data from a munin docker container, because i don't want to pass the docker socket for security reasons. I might be wrong so feel free to comment on this.

These plugins read the metrics exposed by control groups (cgroups) on a pseudo-filesystem. We can find the memory metrics for a Docker container under the */sys/fs/cgroup/memory/docker/container_id/* directory and the cpu metrics under the */sys/fs/cgroup/cpuacct/docker/container_id/* directory.

## **docker_memory plugin**

This plugin uses the pseudo-file *memory.usage_in_bytes* to get the memory metrics. The munin data type must be defined as "gauge".

## **docker_cpu plugin**

This plugin uses the pseudo-file *cpuacct.usage* and *cpuacct.usage_percpu* to get the CPU metrics. The pseudo-file *cpuacct.usage* reports the total CPU time (in nanoseconds) consumed by a container and the pseudo-file *cpuacct.usage_percpu* reports the CPU time (in nanoseconds) consumed on each CPU. This plugin gets the total number of cores available to scale the munin graph.

The munin data type must be defined as "derive" because the CPU usage is a cumulative metric. A derivative shows the difference between data points, and allows the monitoring of CPU usage during each period. The munin sample interval is fixed at 300 seconds.

# Configuration

1. Install the munin-node in the docker server. For more information check the following URL: [munin guide](http://guide.munin-monitoring.org/en/latest/installation/install.html).
2. Copy the *docker_cpu* and *docker_memory* plugins to the munin plugin folder: /usr/share/munin/plugins.
3. Set plugin permissions: `chmod 755 docker_*`
4. Create symbolic links.
  
  ```
  cd /etc/munin/plugins/
  ln -s /usr/share/munin/plugins/docker_memory docker_memory
  ln -s /usr/share/munin/plugins/docker_cpu docker_cpu
  ```
5. Give root privilege to the docker plugin. Root privilege is required for munin to execute the docker command.
  1. Create a new file named "docker" inside the folder /etc/munin/plugin-conf.d/
  2. Insert the following content in the new docker file:
  ```
  [docker_*]
  user root
  ```
6. Restart the munin-node service.
