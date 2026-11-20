# Step 1: Foundation (PC 2)

Goal: both PCs are on the same private network, have Docker and Git, and can reach each other. No honeypots yet.

> The same steps apply to PC 1. Only the hostname changes (`pc1` instead of `pc2`).

## 1. Create the GitHub repository (once, one person)

1. On GitHub: **New repository**, name `honeydeception-cti`, set it to **Private**, and do not add a README (we already have one).
2. **Settings > Collaborators > Add people**, then add your friend.
3. Push this folder:

```bash
cd honeydeception-cti
git init -b main
git add .
git commit -m "Initial commit: architecture docs and repo layout"
git remote add origin git@github.com:<your-username>/honeydeception-cti.git
git push -u origin main
```

(If you prefer HTTPS, use `https://github.com/<your-username>/honeydeception-cti.git` and a personal access token.)

4. Your friend clones it:

```bash
git clone git@github.com:<your-username>/honeydeception-cti.git
```

### SSH key for GitHub (if you do not have one)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
```

Paste the output in **GitHub > Settings > SSH and GPG keys**.

### Git identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Protect `main` (recommended)

**Settings > Branches > Add rule** for `main`, then enable "Require a pull request before merging".

## 2. Install Docker (Ubuntu 22.04 / 24.04)

```bash
sudo apt update
sudo apt install -y ca-certificates curl git
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker
docker run --rm hello-world
docker compose version
```

On Windows: install **WSL2 (Ubuntu)** and **Docker Desktop** with the WSL2 backend, then run the commands above inside the Ubuntu terminal.

## 3. Install Tailscale and join the mesh

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --hostname=pc2
tailscale ip -4
```

Log in through the link it prints.

**Connecting your friend:** use one shared Tailscale network. In the [admin console](https://login.tailscale.com/admin/users), invite your friend by email (the free plan allows a small number of users), or use **Share node** from the machine menu. Later the sensor VM and the VPS join the same network.

Machine names we will use: `pc1`, `pc2`, `sensor-vm`.

## 4. Verify connectivity

From PC 2:

```bash
tailscale status
tailscale ping pc1
ping -c 3 pc1
```

From PC 1: the same with `pc2`. Both must answer.

## 5. Prepare local config

```bash
cp .env.example .env
# edit .env and set a real POSTGRES_PASSWORD
```

`.env` is ignored by Git. Never commit it.

## 6. Define "done"

- [ ] Repo is on GitHub and both people can push branches
- [ ] `docker run hello-world` works on both PCs
- [ ] `tailscale ping` works in both directions
- [ ] `.env` exists locally on both PCs and is **not** tracked by Git (`git status` does not list it)

Then update the roadmap checkbox in the main README and open a Pull Request.

## Next: Step 2

Create the isolated sensor VM on PC 2 and run Cowrie with Fluent Bit inside it. Before that, decide with your friend which hypervisor to use (VirtualBox or KVM/libvirt) and what the VM network looks like (internal network, no internet egress, only a firewall rule to PC 1 over Tailscale).
