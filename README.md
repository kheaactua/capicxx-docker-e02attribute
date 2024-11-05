# Overview

Small experimental docker setup to investigate various vsomeip 3.5 issues.

# Usage

Clone this repo, and start the services with:
```sh
docker compose up
```

To view the logs in DLT, start DLT with:
```sh
docker compose up dlt-viewer
```
and connect to `a_dlt-daemon` with the DLT viewer.

To view the network traffic, open Wireshark on your host, and connect to an
interface whose name starts with `br-`

To simulate a network issue, use `docker compose execute` to manually modify the link, _i.e._
```sh
docker compose execute a_rm ip set link down eth0
<wait 4-5 seconds>
docker compose execute a_rm ip set link up eth0
```

# Config

## Startup

You'll notice in the `a_rm` service the commands:
```yaml
    command:
      - cmd:routingmanagerd # Primary command (special entry-point argument)
      - amf-init-config
      - amf-config-add-service $${MA_SERVICE_ID} $${MA_SERVICE_INSTANCE_ID}
      - amf-config-add-application $${MA_SERVICE_APP_ID} $${MA_SERVICE_APP_NAME}
      - amf-config-add-service $${E02ATTRIBUTE_SERVICE_ID} $${E02ATTRIBUTE_SERVICE_INSTANCE_ID}
      - amf-config-add-application $${E02ATTRIBUTE_SERVICE_APP_ID} $${E02ATTRIBUTE_SERVICE_APP_NAME}
      - amf-config-set-routing routingmanagerd
      - /usr/local/bin/routingmanagerd \| tee $${LOG_DIR}/rm.log
```

The first line simply tells the entrypoint the name of the command that'll eventually be called.  This exists sort of as a hack so that it can be overwritten with bash when testing.

- `amf-init-config`: Copies the vsomeip config files to the correct location and adjusts their network addresses
- `amf-config-add-application`: Adds the application to the vsomeip config
- `amf-config-add-service`: Adds the service to the vsomeip config
- `amf-config-set-routing`: Sets the routing manager to the correct application
- `/usr/local/bin/routingmanagerd \| tee $${LOG_DIR}/rm.log`: Starts routingmanagerd and pipes it's output to a log

These functions allow easy configuration of each node.

## "A" and "B" nodes

This environment is setup to simulate two systems communicating with each
other.  The `a_rm` service is the "A" node, and the `b_rm` service is the "B"
node.  The `a_rm` service is configured to send a message to the `b_rm`
service, and the `b_rm` service is configured to send a message to the `a_rm`
service.

All the variables used in the commands above are in the respective
`run_env/capinode_<network>.env` file which is loaded into the service
definition.  This is how the same variable can be used on both nodes, but with
different values.

# ManyAttributes Example

This setup includes a second example.  It's interactive so it doesn't work well with `docker compose up`.  Instead, to run it, in one shell start the service:
```sh
docker compose run --rm a_ma_service
```

and in another, start the client
```sh
docker compose run --rm a_ma_client
```

