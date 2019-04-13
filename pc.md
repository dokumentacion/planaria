![pc](term.png)

# Planaria Computer

Planaria Computer is a command line interface for

- Scaffolding planaria apps
- Managing Bitcoin keypairs as API keys
- Managing Planaria containers
- Publishing your planaria state machines
- Pulling and deploying existing Planaria state machines from the network

![pc](pc2.gif)

# Install

Planaria Computer is an NPM Package. You can install it globally by running:

```
npm install -g planaria
```

---

# User Commands

## 1. new

Create a new user

```
pc new user
```

## 2. ls

List user accounts

```
pc ls user
```

---

# Machine Commands

## 1. new

### genesis

To try a default Planaria which contains all the available attributes, generate a new machine with:

```
pc new genesis
```

### machine

Create an empty new project with:

```
pc new machine
```

You will be presented with a short questionnaire.

Once you finish the questionnaire, the command will auto-generate all the files required to run a planaria node:

Now you're ready to start!


---

## 2. start

Start a node

1. Start Planaria
2. Start Planarium
3. Start All

### Start Planaria

Planaria is the "Write" interface. Here's how to start a Planaria container:

```
pc start write
```

### Start Planarium

Planarium is the "Read" interface. Here's how to start a Planarium container:

```
pc start read
```

### Start all

You can start both Planaria and Planarium with a simple command:

```
pc start
```

This will start Planaria first, and then Planarium.

---


## 3. stop

Stop a node

1. Stop Planaria
2. Stop Planarium
3. Stop All

### Stop Planaria

Planaria is the "Write" interface. Here's how to stop a Planaria container:

```
pc stop write
```

### Stop Planarium

Planarium is the "Read" interface. Here's how to stop a Planarium container:

```
pc stop read
```

### Stop all

You can stop both Planaria and Planarium with a simple command:

```
pc stop
```

This will stop Planarium first, and then Planaria.

---

## 4. restart

Restart a node.

You may want to use `restart` if you already have a running node and restart with a single command

1. Restart Planaria
2. Restart Planarium
3. Restart All

### Restart Planaria

Restart Planaria

```
pc restart write
```

### Restart Planarium

Restart Planarium

```
pc restart read
```

### Restart all

Restart both Planaria and Planarium

```
pc restart
```

---

## 5. logs

Display logs. A convenience method for `docker logs` for Planaria/Planarium containers.

### Planaria Logs

Start watching Planaria (write) logs. Displays last N lines of Planaria container logs, and starts watching.

```
pc logs write [LINES]
```

You can also just enter the following to start watching from the last 1000 lines (default)

```
pc logs write
```


### Planarium Logs

Start watching Planarium (read) logs. Displays last N lines of Planarium container logs and starts watching.

```
pc logs read [LINES]
```

You can also just enter the following to start watching from the last 1000 lines (default)

```
pc logs read
```

> Internally this is just a convenience method for `docker logs -f [CONTAINER_ID] --tail [LINES]`. After all, everything is a docker container so you can just use the docker commands

---

## 6. rewind

Rewind time to right before the specified height

```
pc rewind [height]
```

Above command will:

1. Stop all containers (First `Planarium`, and then `Planaria`)
2. Rewind Planaria's clock to `height`
3. Restart Planaria, which will execute [onrestart](api?id=onrestart)

> **Of course, you will have to implement the `onrestart` method to handle the rewind, such as cleaning up all the relevant items after the `height`, and resetting the clock.**


---


## 7. push

Push the current machine

```
pc push
```

You must run the command inside the `genes/[ADDRESS]` folder, since you are pushing each "gene" to the registry, and not the entire node.

> **NOTE: One node can run multiple "genes" (Planaria machines)**


---

## 8. pull

Run the command from the root folder:

```
pc pull [ADDRESS]
```

> **NOTE: You can pull (and run) as many machines as you want from a single node, simultaneously.**

![pull](pull.gif)

---

## 9. update

Update the Planaria and Planarium images

1. Update Planaria
2. Update Planarium
3. Update All

Recommended: run update after stopping the containers.

### Update Planaria

Update Planaria to the latest stable image

```
pc update write
```

### Update Planarium

Update Planarium to the latest stable image

```
pc update read
```

### Update all

Update both Planaria and Planarium to the latest stable image

```
pc update
```


---

# Reset the Entire Node

Sometimes you may want to reset the entire node and start over. In this case you can try the following:

First, stop the containers:

```
pc stop
```

Second, prune all Docker related stuff:

```
docker system prune --volumes
```

Third, delete the volume. If you were using `./db` to store the Planaria state, you can run:

```
rm -rf ./db
```

---
