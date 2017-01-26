
#1. Introduction

The contrail BGP implementation was designed from scratch to run on modern server environments. The main goals were to be able to take advantage of multicore CPUs, large (>4G) memory footprints and modern software development techniques.

BGP can be divided in the following components:

1. **Input processing**: decoding and validating messages received from each peer.
2. **Routing table operations**: modifying the routing table and determining the set of updates to generate.
3. **Update encoding**: draining the update queue and encoding messages to a set of peers.
4. **Management operations**: configuration processing and diagnostics.

This blueprint provides a detailed description on defining a new Route Distinguisher field by:

1. Making changes in Contrail configuration files.
2. Making changes in Contrail GUI.

All of these steps are to be performed for the new functionality to work successfully.

#2. Problem statement
###Normalize route origin when learning routes from a VM/VNF.
This feature request is related to BGP as a Service (i.e. the vRouter peering with a VNF running in a VM). It concerns the route origin field in BGP. On PNF routers it is possible to override the route origin field of incoming routes that are learned from peers. This same capability is needed when Contrail learns routes from VNFs. In particular, this is related to the vLB VNF because VNF does NOT want route origin to be part of the tiebreaker protocol when choosing between routes. It is necessary to be able to set the route origin field of routes learned from vLB VNFs to a single value across entire network so that differences in route origin value between different vLB instances won't contribute to route selection.

#3. Proposed solution
Contrail by default exposes certain configurable options to the admin in management console which are eventually used by underlying service when making certain decisions or creating packets. In order to make origin field configurable, following set of changes are needed:

+ Expose a Route Distinguisher in Network admin UI

##3.1 Alternatives considered
Describe pros and cons of alternatives considered.

##3.2 API schema changes
 +
 +**Configuration Changes:**
 +
 ++ When used in the control node process, network derives its internal configuration from the configuration distributed by the IFMAP server. This functionality is handled by the BgpConfigManager class.
 +
 ++ The **first step** towards defining a new knob is to add it to the schema. OpenContrail auto-generates the **REST API** that stores the configuration and makes it available through the IF-MAP server. It also generates the **API client library** that is can be used to set the configuration parameters. The network related schema is present in **controller/src/schema/vnc_cfg.xsd**.
 +
 +**Changes in vnc_cfg.xsd:**
 +
 ++ Add a new XSD element called “override-rd”. Where network elements defined.
 +
 ++ Execute the command **scons controller/src/api-lib**. This command builds the Python client api library that we will use later on to set the new configuration parameter. You can poke around at the generated code: **grep override-rd build/debug/api-lib/vnc_api/gen/**
 
##3.3 User workflow impact

Contrail GUI allows the user define a new route distinguisher. Route Distinguisher field define in network creation wizard, where user can provide desired  Route Distinguisher value.

##3.4 UI changes

Details in section 4.1 below.

##3.5 Notification impact

There were no changes made in logs, UVEs or alarms.

#4. Implementation
##4.1  Work items

It has 4 modules. The GUI changes are mentioned below.

****The rest are defined in contrail-controller repo README.md****

###4.1.1 UI changes

These steps are to be followed to make changes in contrail GUI to reflect the impact of modifications in schema:

+ In vnCfgModel, add override_rd in defaultConfig which is present in this file: **webroot/config/networking/networks/ui/js/models/vnCfgModel.js**.

+ To add a new field on the GUI, a text field is defined in **webroot/config/networking/networks/ui/js/views/vnCfgEditView.js**

By making the above mentioned changes, the Route Distinguisher Field will become configurable in the UI.
On frontend, we get field of **Route Distinguisher IP** in the tabs **Create** and **Edit**. Route Distinguisher field is also visible in the tab “Networks”.

![alt text](images/sec_4.1.1_a.png "Img 10")

![alt text](images/sec_4.1.1_b.png "Img 10")


An object is passed from frontend to API Server when we create network.

Details in contrail-controller repo README.md

#5. Performance and scaling impact
##5.1 API and control plane

There are no changes in scalability of API and Control Plane.
##5.2 Forwarding performance
We do not expect any change to the forwarding performance.

#6. Upgrade
The BGP origin field is a new field and hence does not have any upgrade impact.

#7. Deprecations
There are no deprecations when this change is made.

#8. Dependencies
There are no dependencies for this feature.

#9. Testing
##9.1 Unit test

GUI unit test: Check if values are visible on frontend and are passed to the backend.

##9.2 Dev test

Flow Test Steps:

+ Check if value of Route Distinguisher is received from frontend.

These tests were completed successfully.

#10. Documentation Impact
Route Distinguisher field details have to be added in user documentation.

#11. References
[bgp_design](http://juniper.github.io/contrail-vnc/bgp_design.html)

[adding-bgp-knob-to-opencontrail](http://www.opencontrail.org/adding-bgp-knob-to-opencontrail/)

[contrail-controller (source-code)](https://github.com/Juniper/contrail-controller/tree/master/src/vnsw/agent)
