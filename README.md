[![Build Status](https://travis-ci.org/clearcontainers/agent.svg?branch=master)](https://travis-ci.org/clearcontainers/agent)
[![Go Report Card](https://goreportcard.com/badge/github.com/clearcontainers/agent)](https://goreportcard.com/report/github.com/clearcontainers/agent)
[![Coverage Status](https://coveralls.io/repos/github/clearcontainers/agent/badge.svg?branch=master)](https://coveralls.io/github/clearcontainers/agent?branch=master)

# container-vm-agent
Virtual Machine agent for hardware virtualized containers

## Role
This project holds the code related to a virtual machine agent relying on the communication protocol defined by hyperstart. That way, it allows to spawn some processes on behalf of pod/container(s) running inside the virtual machine.
The code relies heavily on [libcontainer](https://github.com/opencontainers/runc/tree/master/libcontainer) so that we can reuse as much as possible the code used by `runc` (standard to run containers on bare metal).
