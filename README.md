[![Build Status](https://travis-ci.org/clearcontainers/agent.svg?branch=master)](https://travis-ci.org/clearcontainers/agent)
[![Go Report Card](https://goreportcard.com/badge/github.com/clearcontainers/agent)](https://goreportcard.com/report/github.com/clearcontainers/agent)
[![Coverage Status](https://coveralls.io/repos/github/clearcontainers/agent/badge.svg?branch=master)](https://coveralls.io/github/clearcontainers/agent?branch=master)

# container-vm-agent
Virtual Machine agent for hardware virtualized containers

## Role
[`cc-agent`](https://github.com/clearcontainers/agent) is a daemon running in the
guest as a supervisor for managing containers and processes running within
those containers.

The `cc-agent` execution unit is the pod. A `cc-agent` pod is a container sandbox
defined by a set of namespaces (NS, UTS, IPC and PID). `cc-runtime` can run several
containers per pod to support container engines that require multiple containers
running inside a single VM. In the case of docker, `cc-runtime` creates a single
container per pod.

`cc-agent` uses a communication protocol defined by the [hyperstart](https://github.com/hyperhq/hyperstart)
project. This was chosen to maintain backward compatibility with the `hyperstart`
agent used in the 2.x `Clear Containers` architecture.

The `cc-agent` interface consists of:
- A control serial channel over which the `cc-agent` sends and receives specific
  commands for controlling and managing pods and containers. Detailed information
  about the commands can be found at [`cc-agent` API](https://github.com/clearcontainers/agent/tree/master/api).
- An I/O serial channel for passing the container processes output streams (`stdout`,
  `stderr`) back to `cc-proxy` and receiving the input stream (`stdin`) for them. As
  all streams for all containers are going through one single serial channel, the
  `cc-agent` prepends them with container specific sequence numbers. There are
  at most two sequence numbers per container process: one for `stdout` and `stdin`,
  and another one for `stderr`.

`cc-agent` supports the following commands:
- `StartPodCmd`: Sets up a pod in a newly created VM. 
- `NewContainerCmd`: Creates a new container within a pod. This needs to be sent
  after the `StartPodCmd` has been issued for starting a pod. This command also
  starts the container process.
- `ExecCmd`: Executes a new process within an already running container.
- `KillContainerCmd`: `cc-shim` uses this to send signals to a container process.
- `WinsizeCmd`: `cc-shim` uses this to change the terminal size of the terminal
  associated with a container.
- `RemoveContainerCmd`: Removes a container from the pod. This command will fail
  if the container is in a running state.
- `Destroypod`: Removes all containers within a pod . All containers need to be
  in a stopped state for this command to succeed. The command also frees resources
  associated with the pod.

Each control message is composed of a command code and a payload required for
the command:

```
  ┌────────────────┬────────────────┬──────────────────────────────┐
  │  Command Code  │     Length     │ Payload(request or response) │
  │   (32 bits)    │    (32 bits)   │     (data length bytes)      │
  └────────────────┴────────────────┴──────────────────────────────┘
```
- `Command Code` is the predefined id associated with a command.
- `Length` is the size of the entire message in bytes and encoded in network order.
- `Payload` is the JSON-encoded command request or response data.

Each stream message is composed of a stream sequence code and a payload containing
the stream data.

```
  ┌────────────────┬────────────────┬──────────────────────────────┐
  │  Sequence Code │     Length     │          Payload             │
  │   (64 bits)    │    (64 bits)   │     (data length bytes)      │
  └────────────────┴────────────────┴──────────────────────────────┘
```
- `Sequence code` is the 64 bit sequence number assigned by `cc-agent` for a stream.
- `Length` is the size of the entire stream message in bytes and encoded in network
  order.
- `Payload` is the stream data.

The `cc-agent` makes use of [`libcontainer`](https://github.com/opencontainers/runc/tree/master/libcontainer)
to manage the lifecycle of the container. This way the `cc-agent` reuses most
of the code used by [`runc`](https://github.com/opencontainers/runc).
