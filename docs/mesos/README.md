<!--- master only -->
> ![warning](../images/warning.png) This document applies to the HEAD of the calico-docker source tree.
>
> View the calico-docker documentation for the latest release [here](https://github.com/projectcalico/calico-docker/blob/v0.13.0/README.md).
<!--- else
> You are viewing the calico-docker documentation for release **release**.
<!--- end of master only -->

# Mesos with Calico Networking
**Calico provides IP-Per-Container networking for your Mesos cluster.** The following collection of tutorials will walk you through the steps necessary to get up and running.

Note: IP-Per-Container Networking with Calico is an opt-in feature for Mesos Frameworks that launch tasks with [Networkinfo](https://github.com/apache/mesos/blob/0.26.0-rc3/include/mesos/mesos.proto#L1383). This means that your favorite Mesos Frameworks will not work with Calico until they have opted to include Networkinfo when launching tasks. Marathon is the first Framework planned to use this new feature, but its Mesos-networking support is not yet complete. 

Calico support is under development. Use the following information to ensure you choose the right version:
- Calico fully supports and recommends Mesos 0.26
- Calico supports Mesos 0.25, but we recommend against using it as there aren't any Frameworks (including Marathon) which support the Networkinfo specs from 0.25 (which were modified for 0.26)
- Calico works on Mesos 0.24, but only as a proof of concept, and is no longer supported.

Have any questions on these tutorials? Is there any material you want to see covered? Let us know on our [Slack channel](https://calicousers-slackin.herokuapp.com/).

## 1. Prepare Master and Agent Nodes
The [Mesos Host Preparation tutorial](PrepareHosts.md) will walk you through hostname and firewall configuration for compatability between Calico and Mesos.

## 2. Prepare Core Services
Zookeeper and etcd serve as the backend datastores for Mesos and Calico, respectively. The [Core Services Preparation tutorial](PrepareCoreServices.md) will walk you through setting up both services using Docker.

## 3. Calico
The following tutorials will help you install the Calico services required for each agent in your Mesos cluster.

### Calico With Docker (Recommended)
Calico is primarily distributed as a Docker container. Follow one of the tutorials below to set that up.

a.) [Install the Calico Networking in Mesos from RPM](RpmInstallCalico.md)

b.) [Manually Install Calico Networking in Mesos](ManualInstallCalico.md)

### Calico Without Docker
Don't want to use Docker in your Mesos Cluster? Calico can run directly on your Agent. Choose one of the following tutorials to install Calico without Docker.

c.) Create and Install the Dockerless Calico-Mesos RPM (coming soon)

d.) Manually Install Dockerless Calico (coming soon)

[calico]: http://projectcalico.org
[mesos]: https://mesos.apache.org/
[net-modules]: https://github.com/mesosphere/net-modules
[docker]: https://www.docker.com/

## 4. Mesos + Netmodules
Calico works in conjunction with [netmodules][net-modules], a Mesos Networking Module. Choose one of the following tutorials to install Mesos + Netmodules: 

a.)  We've bundled Mesos with netmodules into a convenient RPM - follow the [Mesos + Netmodules RPM Installation tutorial](RpmInstallMesos.md) to install it.

b.) Already have mesos installed? Netmodules can be compiled onto an existing mesos deployment so long as all the mesos source files are present (therefore, unfortunately at this time, netmodules can not be added to mesos when installed via the mesosphere rpm releases). Follow the [Manually Install Mesos + Netmodules tutorial](ManualInstallNetmodules.md) to install it.

## 5. Launching Tasks
Calico is compatible with all frameworks which use the new NetworkInfo protobuf when launching tasks. Marathon has introduced limited support for this. For an early peek, use `djosborne/marathon:ip_per_task`:
```
$ docker run \
-e MARATHON_MASTER=zk://<ZOOKEEPER-IP>:2181/mesos \
-e MARATHON_ZK=zk://<ZOOKEEPER-IP>:2181/marathon \
-p 8080:8080 \
djosborne/marathon:ip_per_task
```
This version of Marathon supports two new fields in an application's JSON file:

- `ipAddress`: Specifiying this field grants the application an IP Address networked by Calico.
- `group`: Groups are roughly equivalent to Calico Profiles. The default implementation isolates applications so they can only communicate with other applications in the same group. Assign a task the static `public` group to allow it to communicate with any other application.

The Marathon UI has does not yet include a field for specifiying NetworkInfo, so we'll use the command line to launch an app with Marathon's REST API. Below is a sample `app.json` file that is configured to receive an address from Calico:
```
{
    "id":"/calico-apps",
    "apps": [
        {
            "id": "hello-world-1",
            "cmd": "ifconfig && sleep 30",
            "cpus": 0.1,
            "mem": 64.0,
            "ipAddress": {
                "groups": ["my-group-1"]
            }
        }
    ]
}
```

Send the `app.json` to marathon to launch it:
```
$ curl -X PUT -H "Content-Type: application/json" http://localhost:8080/v2/groups/calico-apps  -d @app.json
```

[![Analytics](https://ga-beacon.appspot.com/UA-52125893-3/calico-docker/docs/mesos/README.md?pixel)](https://github.com/igrigorik/ga-beacon)