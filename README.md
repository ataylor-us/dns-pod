# Adguard Home Kube Play

This is a guide to setting up Adguard home on a fresh EL10-like system from scratch, in podman userland.

## Step 1: Ports

Since port 53 is a sub-1024 protected port (tl;dr: important), we need to give special permission to bind to it as a user, run:

```bash
# Add a sysctl file
echo 'net.ipv4.ip_unprivileged_port_start=53' | sudo tee /etc/sysctl.d/99-adguard.conf
# Apply settings to runtime
sudo sysctl --system
```

Alternatively we can bind it to another port like `10053`, but this makes it easier to work with clients.


## Step 2: Running the pod

We're going to run this pod with "quadlet", a way for systemd to manage the pod. This allows the pod to start on boot, and reboot on failure.

Create the following directory, and move the pod and quadlet file to them.
```bash
mkdir -p ~/.config/containers/systemd
cp adguardhome-pod.yaml adguardhome.kube ~/.config/containers/systemd/
# Reload userland systemd
systemctl --user daemon-reload
```

Also, let's open up the ports on the firewall so we can view this remotely:

```bash
# DNS Service (53 tcp/udp)
sudo firewall-cmd --permanent --add-service=dns
# Web UI
sudo firewall-cmd --permanent --add-port=3000/tcp
# Apply Rules
sudo firewall-cmd --reload
```

## Step 3: Persistence

Normally podman containers are "daemonless", meaning that there's nothing to have it reboot when your computer starts up again.

With a few changes, we can have our user session "linger" on boot, allowing it to start on boot.

```bash
# Enable lingering
loginctl enable-linger $USER
# Start the service 
systemctl --user start adguardhome.service
```

## Step 4: Updating

Updating the container is easy, just run:
```bash
podman auto-update
```

For automatic updates, enable the systemd timer:
```bash
systemctl --user enable --now podman-auto-update.timer
```
