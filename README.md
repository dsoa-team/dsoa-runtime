dsoa-runtime
============
0. Executing Felix:
Select dsoa-runtime project and "Run as... Java Application" 

1. Installing DSOA Service Simulator:
start file:apps/simulation/commons-math-2.2.jar
start file:apps/simulation/dsoa-qos-simulator.jar

2. Installing and Executing Homebroker Example:
start file:apps/HomebrokerModel.jar
start file:apps/simulation/simulated-service.jar
start file:apps/HomebrokerBB.jar
start file:apps/HomebrokerBBClient.jar

3. Basic Felix Commands:
3.1 ps: lists bundles
3.2 inspect service|package capability|requirement [id]: lists the services provided/required by bundle identified by id
3.3 arch 
	-factories: lists component factories
	-factory factory_name: details component factory
	-instances: lists component instances
	-instance instance_name: details component instance 
