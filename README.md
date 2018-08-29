# Intent based networking showcase: VPN service
The purpose of this simple demo is to showcase intent based networking by modeling VPN service with declarative Cloudify DSL which is based on TOSCA.
In order to make it easy to read and follow, we're using simplified node & relationship types (phantoms) - which the only purpose is to print on the console fact that given lifecycle action is being triggered on a node.
The intent is described in a blueprint files and as such, we have 3 blueprints describing VPN with one, two or three branches:
- branch-1-blueprint.yaml
- branch-2-blueprint.yaml
- branch-3-blueprint.yaml
What we're going to do, is to create deployment of VPN with one branch and later on, using deployment update lifecycle management function in Cloudify Manager - we're going to "upgrade" existing VPN to two and three branches respectively. Upgrade operation works both ways up and down if needed.

```
demo$ cat ./types/demo-types.yaml
node_types:

  cloudify.nodes.vRouter:
    derived_from: cloudify.nodes.Router
    interfaces:
      cloudify.interfaces.lifecycle:
        start:
          implementation: scripts/router-start.py
          executor: central_deployment_agent
        stop:
          implementation: scripts/router-stop.py
          executor: central_deployment_agent

----CUT---
relationships:

  cloudify.relationships.rtr_connected_to_network:
    derived_from: cloudify.relationships.connected_to
    source_interfaces:
      cloudify.interfaces.relationship_lifecycle:
        preconfigure:
          implementation: scripts/relationships/src-preconfigure.py
          executor: central_deployment_agent
---CUT---
```

Besides of update workflow execution, this example is demonstrating what lifecycle interfaces are triggered in install & uninstall workflow executions.

```
2018-08-14 01:55:00.503  LOG <ACME-vpn> [HQ_v8f67m.start] INFO: ---> VNF HQ is starting...
2018-08-14 01:55:00.503  LOG <ACME-vpn> [branch-1-rtr_zexi4x.start] INFO: ---> VNF branch-1-rtr is starting...
2018-08-14 01:55:00.503  LOG <ACME-vpn> [establish] INFO: ---> src-establish is starting...
2018-08-14 01:55:00.503  LOG <ACME-vpn> [establish] INFO: ---> trgt-establish is starting...
```
# Execution examples

### STEP1: Upload blueprints
```
cfy blueprint upload -b branch-1 ./branch-1-blueprint.yaml
cfy blueprint upload -b branch-2 ./branch-2-blueprint.yaml
cfy blueprint upload -b branch-3 ./branch-3-blueprint.yaml

```

### STEP2: Create VPN with one branch
```
cfy deployment create ACME-vpn -b branch-1
cfy executions start -d ACME-vpn install
```

### STEP3: Update VPN to two or three branches

```
cfy deployments update  ACME-vpn -b branch-2
cfy deployments update  ACME-vpn -b branch-3
```
