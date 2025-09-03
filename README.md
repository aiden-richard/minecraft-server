# Overview
This is a short document that will go over how I host a Minecraft server which is served on my domain at: mc.pointtomyserver.xyz.

When building this server (still a work in progress), I experimented with a few different stacks but landed on a setup that I think is pretty clean.

## Hosting
A couple of months ago I purchased an old HP workstation on ebay for cheap. It has 16 GB RAM, Intel Core i7-7700, and enough storage for this project.

This is what we are going to be hosting the minecraft server on. I am also running the server in a docker container on this machine for ease of use. I am using the [https://github.com/itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server) docker image for this.

The server runs tailscale VPN which lets me or my friends connect from any device I invite to the tailscale network. This will come up later.

Once I had the server running, I wanted to put it up on my domain. The only problem with this is that I would have to create a DNS record pointing my domain to my home IP. Some people are okay with this; I don't like it.

## Forwarding Traffic

### Option One: Cloudflare Tunnels
Cloudflare Tunnels allow you to securely route traffic from the internet to your local server without exposing your server’s public IP address. I use Cloudflare to manage my domain and have previously used tunnels as a super easy way to serve a self-hosted app on your domain. This method is very clean because the tunnel sits in another Docker container and just passes traffic to the main app container, all without revealing my home IP.

**The problem: Minecraft uses raw TCP**  

After some trial and error, I found out that the issue was Minecraft's raw TCP traffic. It couldn't be passed through the tunnel since Tunnels can only send various supported http protocols, not raw TCP.

Oh, well.

### Option Two: Proxy Server
I remembered I had my student benefits from Github and redeemed my $200 credit on Digital Ocean. I spun up a droplet on the cheapest server, starting at $4 per month. This got me 512 MB of RAM, 1 vCPU, and a whopping 10 GB SSD. Oh, and a static ip which is nice.

This beast is going to be our proxy server that mc.pointtomyserver.xyz will resolve to. The set up process was fairly simple for this machine.

1. Connect to tailscale network.
    - In order to see the minecraft server, we need to add this machine to our tailscale network. This is a 2 or 3 commmand process I won't go over here because tailscale provides a fine guide.
2. Install the proxy service
    - I went through a few options but ended up choosing socat for pure simplicity. Install using `apt install socat`. Then create a service to start the proxy at startup. (See [mcproxy.service](mcproxy.service))
3. Shutting off the proxy server if the minecraft server is inactive. This is something I toyed with using mcrcon commands and crontab. I might come back and add to it.

That's Really all there is to the proxy server. Now all that's needed is the DNS record pointing the mc subdomain to the proxy server IP