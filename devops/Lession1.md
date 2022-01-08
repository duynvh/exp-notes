# What is Devops?

# TDD
- Goals: 
	- know something break and where
	- know that the whole system is working correctly
	
# CI
- To prevent coding errors, and increase user satisfication
- Setup CI
- LayerCI
- Setup CI for PR:
	- After create a PR, CI will be trigger and run test
	- PR can be automatically rejected if CI run failed
	- If it passed, we can confidently merge this PR.
	
# Code coverage

# Linting best practices
- Should add to CI
- Run automatically linting if not linted and create and new branch for that

# Ephemeral environments

# VMs and containers
- Linux
	- Memory
	- CPU (processors)
	- Disk
	- Devices

- Show number of running command in Linux machine:
```bash
ps aux | wc -l
```
-  When running docker in Linux machine, number of process equal to process running on docker container and in linux machine outside container.

- Containers provide a "fake" version of Linux.
- VMs provide "fake" version of CPU, RAM, Disk and devices.

-> VMs fake on level deeper

- VM equivalent of Docker is called a hypervisor.

- VMs vs Containers Perf
	- CPU in VMs is 10-20% slower than containers
	- VMs also usually use 50-100% more storage
	- VMs use ~200MB of memory each for the 'inner operating system' whereas containers have essentially no memory overhead.
	
- When VMs are better choice
	- If you are running untrusted code
	- If you are running a Windows or MacOS guest within a Linux Server
	- You can emulate hardware devices with a VM
	
# Rolling deployments
- Remove downtime
- Must ensure backwards compitable in API

# Blue/Green deployments
- Acceptance tests

- Rainbow deployments
- Canary deployments
	- Just route around 5% user to new cluster and check if they don't have any negative feedbacks
	
# Auto scaling
- Autoscaling is usually discussed on the timeline of ~1hour chunks of work, used for services that are slower to start or require state.
- If you used autoscaling and took it to its limit, you'd get serverless: define resources that are quickly started, and use them on the timeline of ~100ms.
- Serverless is primarily used for services that are somewhat fast to start, and stateless. You wouldn't run something like a CI run within a serverless framework, but you might run something like a webserver or notification service.

# Service discovery
- Use it when 
	- you need "zero downtime deployment"
	- have more than a couple of microservices
	- you are deploying a serveral environments
- DNS locally

# Log aggregation
- ELK
- Fluentd
- DataDog
- LogDNA
- AWS CloudWatch Logs

# Production metrics
- Log deal with text
- Metric deal with number
- Idea for metric collect:
	- Request fulfillment times
	- Request counts
	- Service resources:
		- Database size and maximum database size
		- Web server memory
		- Network throughput and capacity
		- TLS Cert expire time and life left
- Quartile analysis:
	- How long did the slowest 1% of requests take?
	- How long did the slowest 5% of requests take?
	- How long did the slowest 25% of requests take?
	
- Tools:
	- Grafana/Prometheus
	- DataDog
	- NewRelic
	- Cloudwatch Metrics
	- ...

> Source: https://www.youtube.com/watch?v=j5Zsa_eOXeY&t=15s