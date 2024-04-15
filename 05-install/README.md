# Containerlab Installation

## Docker

Install docker using the handy scrip:

```bash
curl -L http://containerlab.dev/setup-debian \
| sudo bash -s "install-docker"
```

Add yourself to the `docker` group to enable sudo-less docker commands:

```bash
sudo usermod -aG docker $USER
```

Now log out and log in for the group change to take effect.

Check that docker is installed and running:

```bash
docker run --rm hello-world
# Expected output: Hello from Docker!
```

Alternative installation options can be found [here](https://docs.docker.com/engine/install/).

## Containerlab

Install containerlab using the installation script:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Alternative installation options are available [here](https://containerlab.dev/install/).
