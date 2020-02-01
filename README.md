# ECS monitoring using Prometheus and Grafana
Setup ECS monitoring using Prometheus, Grafana, cAdvisor and Node Exporter

## Requirements:

1. Modify 'init-container' definition (in prometheus-grafana-compose.yml/prometheus-grafana-definition.json) to download the correct 'prometheus.yml' configuration file.
2. Modify 'prometheus.yml' to make sure it queries and filters correct set of EC2 instance which needs to be monitored.
   Here I used filter (in ec2_sd_configs section) with tag 'aws:autoscaling:groupName' and vlaue 'ECS_ASG-2' as all instances in my ECS clsuter was tagged with it.
3. Use EC2 instance profile for ECS Cluster instances with permission to make describe calls for EC2 instances. This will be used by Prometheus to discover instance in your ECS cluster and add it in Prometheus targets  (http://monitor_ec2_public_ip:9090/targets).
4. Make sure instances in cluster can reach each other on ports 9090, 9100, and 9200 and open Grafana port 3000 for the IP range from where you need to access the Grafana Dashboard.

## Usages Instructions:

1. Create Task Definitions for cAdvisor, Node-Exporter, Prometheus and Grafana:

```
aws ecs register-task-definition --cli-input-json file://./cadvisor-node-exporter-definition.json --region us-west-2
aws ecs register-task-definition --cli-input-json file://./prometheus-grafana-definition.json --region us-west-2
```
  
2. Create a DAEMON Service to run cAdvisor, Node-Exporter on every node in ECS Cluster:

```
aws ecs create-service --cluster MyWorkingCluster --service-name cadvisor-node-exporter --task-definition cadvisor-node-exporter-definition:1 --launch-type EC2 --scheduling-strategy DAEMON --region us-west-2
```

3. Run one ECS Task for Prometheus and Grafana in the clsuter:

```
aws ecs run-task --cluster MyWorkingCluster --task-definition prometheus-grafana-definition:1  --region us-west-2
```

4. Access Grafana Dashboard using URL: http://monitor_ec2_public_ip:3000
   Use user:admin and password:admin to login and then reset the password.

5. After logginig in, add datasource:

   <img src="https://miro.medium.com/max/2732/0*I8R5h2bM0DpgaoVs" title="Adding the Datasource" alt="Adding the Datasource" />

   * Fig 1: Monitoring results for Uptime, Containers, Load(Numerical Representation), DiskSpace, Memory, Filesystem Usage, CPU Usage, Memory Usage, Targets Online, Total Memory Usage, Alerts


6. Import or Create a custom Grafana-Dashboard to monitor Docker containers running on the host:

   ![Fig 2](https://miro.medium.com/max/2732/0*W0yXIT_P-1Gc_sY4)

   * Fig 2: Monitoring results for Uptime, Containers, Load(Numerical Representation), DiskSpace, Memory, Filesystem Usage, CPU Usage, Memory Usage, Targets Online, Total Memory Usage, Alerts.

   ![Fig 3](https://miro.medium.com/max/2732/0*EO1JyVMHPkbFEYdk)

   * Fig 3: Monitoring results for CPU Usage, Network Traffic, Load, Available Memory, Node Network Traffic, Disk I/O, Node Memory, Filesystem Available, Container Network Input, Container Network Output.

   ![Fig 4](https://miro.medium.com/max/2732/0*HeRmOCOHeHeJkkBB)

   * Fig 4: Monitoring results for System Load on Node and Cached Memory per Container(Stacked).

   ![Fig 5](https://miro.medium.com/max/2732/0*Gmqz4PFyP5LjfRNn)

   * Fig 5: Monitoring Results for CPU Usage per container, Sent Traffic per Container, Received Network Traffic per Container, Memory Usage per Container.

   ![Fig 6](https://miro.medium.com/max/2724/0*WjYd0f9n53689GUl)

   * Fig 6: Monitoring results for Network Rx, Network Tx, Tables for Usage Memory, Remaining Memory, Limit Memory.

   ![Fig 7](https://miro.medium.com/max/2728/0*3jj0V42Ph67rZwFN)

   * Fig 7: Monitoring results for Running versions and Metrics.


7. Useful Grafana Dashboards:
   - Docker Host Monitoring: 11074, 10619, 395
   - Docker Monitoring: 193
   - Docker monitoring with Node selection: 8321
