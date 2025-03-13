# Intro

So what're we doing here?  I'm setting up a home lab kubernetes cluster.  Why?  It's something to fiddle with...

We're going to use [talos linux](https://www.talos.dev/), a virtual machine, and [cluster-template](https://github.com/onedr0p/cluster-template).  It's pretty easy to do, should take you about an hour or less (you can speedrun it in less than 30 minutes, ask me how I know), and it doesn't cost you any money.  Sometimes when folks build home lab clusters they string together a bunch of boxes they can scrounge, but I have enough wires and plugs and whatnot. I'm just using a vm, and a single one at that, you could of course join multiple vm's or bring your own hardware, whatever floats your boat.  Remember, we're just having fun and you can do what makes you happy.

# Background

I'd had cluster-template setup before on a single node talos VM and had a hard drive die... So, I'm doing it again and documenting my steps.  It's been about 7 months since I did this before and the project is a bit different now, so let's see how it goes.

For my starting point I am running windows 11 with vscode, hyper-v, docker desktop for windows, and wsl2 installed, typical consumer cable modem, and an old router.  Despite being just a lowly windows user I am comfortable enough with linux and kubernetes, but it's not exactly my day job.  I'm sure you can follow along with everything on a different platform, swap out hyper-v with whatever toots your horn.

My goal is to host some of my domains, apps, and services that I want, either internal or external, whether written by me or something 3rd party; all from the comfort of a vm running on windows while I play games on steam.  This seems perfectly reasonable and not at all overkill.  While a single node kubernetes cluster running on a vm on windows on a home cable modem isn't at all resilient or available (see above about my nvme dying), it is a reasonably portable and scalable solution starting point for playing along at home.

I have these existing domains on cloudflare that i'm going to use.  To start configuring my cluster I'm going to start with a fictional *somedomain.net*.  I have removed all dns entries, api keys, and tunnels from cloudflare, but the domain is listed in cloudflare.

# Step 1: Git setup

- go to the [cluster-template](https://github.com/onedr0p/cluster-template)
- slap the green 'use the template' button and create a new repository
- give it a name, I creatively chose 'cluster' so I end up with a repo of *github.com/me/cluster*
- decide if you want your cluster to be public or private, i've chosen private
- clone *github.com/me/cluster* locally

# Step 2: Devcontainer install

- open cloned repo dir in vscode
- dev containers: reopen in container, add configuration to workspace
- choose Kubernetes local configuration container template, no other devcontainers extensions
- vscode relaunches now in the dev container
- install vscode extensions when prompted

# Step 3: Devcontainer setup (to be run in your devcontainer shell)

### trust yourself

```bash
git config --global --add safe.directory /workspaces/cluster
```

### install updates and install nmap and dnsutils (for dig):

```bash
sudo apt-get update && sudo apt-get upgrade

sudo apt install nmap

sudo apt install dnsutils
```

### install mise:

```bash
sudo apt update -y && sudo apt install -y gpg sudo wget curl

sudo install -dm 755 /etc/apt/keyrings

wget -qO - https://mise.jdx.dev/gpg-key.pub | gpg --dearmor | sudo tee /etc/apt/keyrings/mise-archive-keyring.gpg 1> /dev/null

echo "deb [signed-by=/etc/apt/keyrings/mise-archive-keyring.gpg arch=amd64] https://mise.jdx.dev/deb stable main" | sudo tee /etc/apt/sources.list.d/mise.list

sudo apt update

sudo apt install -y mise

echo 'eval "$(/usr/bin/mise activate bash)"' >> ~/.bashrc
# on above line I changed the path from the mise docs from ~/.local/bin/mise to /usr/bin/mise as that's where which mise tells me it is ¯\_(ツ)_/¯

source ~/.bashrc
```

### install required tools via mise:

i did this step in a weird order to try to resolve some sort of dependency issue, ymmv

```bash
mise trust
mise install
pip install pipx 
mise install
```

# Step 4: Cloudflare setup

- Add api key

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):
![create token](img/cf-create-token.jpg)

- I did not follow the cluster-template directions here which suggest using the Edit zone DNS template

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):
![Edit zone DNS template](img/cf-token-template.jpg)

if you do this one you can't name it or somesuch...

- Instead used custom template

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):

![Custom template](img/cf-custom-token.jpg)

- Following values

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):

![Custom token values](img/cf-token-values.jpg)

- Create token and **copy the token value somewhere handy**

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):

![Token!](img/cf-token.jpg)

- Verify I see my token in cloudflare

---

Screenshot from [API Tokens - Cloudflare](https://dash.cloudflare.com/profile/api-tokens):

![Verify](img/cf-verify.jpg)

- Authorize cloudflared

```bash
cloudflared tunnel login
```

- Click the link in the shell, sign in, select the domain you want, in my case it's our fictional *somedomain.net*, hit the authorize button, close tab when it says it's ok

- Create tunnel (this step will output a cloudflare-tunnel.json)

```bash
cloudflared tunnel create --credentials-file cloudflare-tunnel.json kubernetes
```

- Verify tunnel was created, I get there through cloudflare.com's search

---

Screenshot from [Cloudflare Dashboard](https://dash.cloudflare.com/):

![Search](img/cf-search-tunnels.jpg)

- It should be there and listed as inactive

---

Screenshot from [Cloudflare Dashboard](https://dash.cloudflare.com/):

![Search](img/cf-tunnels.jpg)

# Step 5: Talos ISO download

- This is the talos ISO I endeded up with: [Talos ISO](https://factory.talos.dev/?arch=amd64&cmdline-set=true&extensions=-&platform=metal&target=metal&version=1.9.4)

# Step 6: VM setup

- Create a new virtual machine in Hyper-V

---

![Hyper-V](img/hv.jpg)


- Named the VM Talos

---

![Hyper-V name](img/hv-name.jpg)

- Select Gen-2

---

![Hyper-V gen2](img/hv-gen2.jpg)

- Memory settings, 4GB + dynamic

---

![Hyper-V memory](img/hv-mem.jpg)

- Network settings, I use my pre-existing external switch

---

![Hyper-V network](img/hv-net.jpg)

- Disk settings, I used the defaults

---

![Hyper-V disk](img/hv-disk.jpg)

- Mount Talos ISO

---

![Hyper-V iso](img/hv-iso.jpg)

- Hit finish to make the Talos VM

- In the VM's settings disable secure boot

---

![Hyper-V secureboot](img/hv-sb.jpg)

- In the VM's settings, I cut down on the number of cores allocated

---

![Hyper-V cpu](img/hv-cpu.jpg)

# Step 7: Start VM

- Just start the VM
- Connect to the VM

I note talos reports it's running on 192.168.0.167, which is my expected network 192.168.0.0/24, it correctly lists that it uses my gateway and dns that I expect from the virtual external switch.

# Step 8: Cluster configuration

- Generate cluster config files

```bash
task init
```

- Fill out cluster.yaml

For cluster.yaml I changed these lines, I picked a block of IP's that should be available, we'll go with the roaring 40's:

```yaml
node_cidr: "192.168.0.0/24"
cluster_api_addr: "192.168.0.40"
cluster_dns_gateway_addr: "192.168.0.41"
cluster_ingress_addr: "192.168.0.42"
repository_name: "me/cluster" # remember, I mentioned this way back in step 1
repository_visibility: "private" # this was marked as optional and I missed it, if you're doing a private repo do this, otherwise set it to public
cloudflare_domain: "somedomain.net" # our fictional domain
cloudflare_token: "abunchofstuffgoeshere" # remember, we saved this somewhere handy way back in step 4
cloudflare_ingress_addr: "192.168.0.43"
```

- Fill out nodes.yaml


```yaml
---
nodes:
  - name: "talos01"
    address: "192.168.0.167" # as noted from step 7
    controller: true
    disk: "" # I can't find a value that works for this so I leave it blank
    mac_addr: "DE:AD:BE:EF:OH:MG"  # get the ether mac address from: talosctl get links -n 192.168.0.167 --insecure (i do this from my host machine as i have talosctl handy)
    schematic_id: "376567988ad370138ad8b2698212367b8edcb69b5fd68c80be1f2ec7d603b4ba" # from my talos iso link in step 5
    mtu: 1500
    secureboot: false
    encrypt_disk: false
```

- Write cluster config

```bash
task configure
```

- Push to git

```bash
git cm "initial" # i use fancy aliases, you do you
git push
```

# Step 9: Add deployment key to github

- Let's add our deploy key to our repo settings, in my case: [https://github.com/me/cluster/settings/keys](https://github.com/me/cluster/settings/keys)

---

Screenshot from [https://github.com/me/cluster/settings/keys](https://dash.cloudflare.com/):

![Add deploy key](img/gh-add.jpg)

- Slap that green Add deploy key button

- Give it a name and paste in the content of `github-deploy.key.pub` and I gave it write access:

---

Screenshot from [https://github.com/me/cluster/settings/keys](https://dash.cloudflare.com/):

![Paste deploy key](img/gh-key.jpg)

- Give it a name and paste in the content of `github-deploy.key.pub` and I gave it write access:

Screenshot from [https://github.com/me/cluster/settings/keys](https://dash.cloudflare.com/):

![Paste deploy key](img/gh-confirm.jpg)

# Step 10: Bootstrap Talos

- Install talos

```bash
task bootstrap:talos
```

Things will happen, you should see the Talos VM go into Installing state, you'll see some connection attempts refused in your shell, it's fine, just hang out.  The VM will reboot.  The VM should boot from iso again and mention to remove the disk and reboot

- Stop the vm

- Unmount the ISO

---

![Hyper-V iso none](img/hv-iso-none.jpg)

- I also change the boot order

---

![Hyper-V boot order](img/hv-boot-order.jpg)

- Start the VM and your shell should complete and return control

# Step 11: Talos setup

```bash
task bootstrap:apps
```

You should see activity in your shell after a bit... Or you can kind of follow along in another terminal with:

```bash
kubectl get pods --all-namespaces --watch
```

Eventually you should get a congrats msg:

"2025-03-12T21:13:24Z INFO Congrats! The cluster is bootstrapped and Flux is syncing the Git repository "

At this point though, we still don't have our network namespace running, so we carry on!

# Step 12: Flux

```bash
task reconcile # make sure flux is all connected to github
flux check
flux get sources git flux-system
```

# Step 13: Verify

```bash
cilium status # make sure cilium says it's ok
nmap -Pn -n -p 443 ${cluster_ingress_addr} ${cloudflare_ingress_addr} -vv # grab these values from cluster.yaml, you should see open 443 ports
dig @${cluster_dns_gateway_addr} echo.${cloudflare_domain} # grab these values from cluster.yaml, you should be able to internally resolve 
kubectl -n cert-manager describe certificates # you should see your cert for *somedomain.net*
```

Huzzah!  At this point you should be on the lines!

You should have your DNS entries in cloudflare for *somedomain.net*, you should see a CNAME entry for external.somedomain.net that goes to your cloudflared tunnel.  You should see a CNAME entry for echo.somedomain.net that goes to external.somedomain.net.  These are both publicly available on https, go try them out in your browser.

I'd probably want to remove the echo server at some point. Perhaps replace that 404 on external with something else.  Next up though I want to host my super-awesome personal website, but I want to do that on *somedomain.com* and not *somedomain.net*.

That'll be the next adventure.
