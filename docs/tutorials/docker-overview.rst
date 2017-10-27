Overview
========

This tutorial will show you how to use Docker Compose to run a demo RackHD environment.

The RackHD services will all be run in separate Docker containers.  Docker Compose will be used to coordinate running the container and to set up the required networks.  A separate Docker Compose file will be used to run virtual compute nodes.

After RackHD is set up successfully, RackHD’s discovery, catalog and poller functionality will be introduced by using the simulated nodes that are run in a separate Docker Compose session. In addition, you have the opportunity to experiment with some RackHD APIs. In this module, you will learn about two different RESTful endpoints in RackHD and experiment with them.  Additionally, the RackHD set up is used to perform an unattended OS install onto a virtual node.

Finally, if you want to learn more about relevant knowledge, you can go to following links.

- RackHD: https://github.com/RackHD
- Docker: https://www.docker.com/
- Docker Compose: https://docs.docker.com/compose/
- Infrasim: https://infrasim.readthedocs.io
