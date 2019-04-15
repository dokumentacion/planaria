# Contribute to Planaria

Here's how to build an image:

# 1. Build

## Build Planaria

Clone the repository:

```
git clone https://github.com/interplanaria/planaria.git
```

Go to the root folder and build a docker image:

```
docker build -t interplanaria/planaria .
```

## Build Planarium

Clone the repository:

```
git clone https://github.com/interplanaria/planarium.git
```

Go to the root folder and build a docker image:

```
docker build -t interplanaria/planarium .
```

---

# 2. Docker

The deployment configuration depends on three files:

1. `planaria.yml`: The docker-compose file that handles Planaria deployment.
1. `planarium.yml`: The docker-compose file that handles Planarium deployment.
1. `.env`: This is a **hidden file** that stores some environment variables.

Normally you don't need to touch these files manually, you should use the `pc` command. But if you must, you can manually change the files to test.

---

# 3. Planaria Computer

All administration is done with the `pc` command. You can contribute to Planaria Computer by pulling from the repository and sending a PR:

```
git clone https://github.com/interplanaria/pc.git
```
