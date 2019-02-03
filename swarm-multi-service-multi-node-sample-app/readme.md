# Steps to create multi-service multi-node sample app

## Public sample voting app for cats and dogs

    5 services, 2 networks, 1 volume

## 1. Create two networks

driver `overlay` available for swarms

### frontend network

    docker network create -d overlay n_frontend

### backend network

    docker network create -d overlay n_backend

### list results

    docker network ls

## 2. Create services

### voter service (application python frontend) attached to frontend network

    docker service create --name voter -p 80:80 --network n_frontend --replicas 2 dockersamples/examplevotingapp_vote:before

### redis service (in memory key-value database) attached to frontend network

Note: If there is no `replicas` parameted in the command, 1 replica is assumed

    docker service create --name redis --network n_frontend redis:3.2

### worker service (.Net) attached to frontend and backend networks

Note: If there is no `replicas` parameted in the command, 1 replica is assumed

    docker service create --name worker --network n_frontend --network n_backend dockersamples/examplevotingapp_worker

### postgres db with volume

`-v` command is not compatible with services
Using `--mount` command with supplied parameters:
`type`, `source`, `target` (this three are the minimum required)
Note that we are making a volume not a bind-mount.
Name it `db` otherwise worker doesn't work blaahh.

    docker service create --name db --network n_backend --mount type=volume,source=db-data,target=/var/postgresql/data postgres:9.4

### results app (nodejs)

there is a problem with websocket (nodejs app)
that's why only 1 replica
there should be a proxy in front of the nodejs app to handle more than 1 replicas using websocket communication. (check for updates)
Note: If there is no `replicas` parameted in the command, 1 replica is assumed

    docker service create --name results --network n_backend -p 5001:80 dockersamples/examplevotingapp_result:before