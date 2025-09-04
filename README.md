# Overview
This repo contains the set up files for a public facing Minecraft server. This README will go over how I host the Minecraft server which is served on my domain at: mc.pointtomyserver.xyz (using a whitelist for invited players and filtered network traffic with Cloudflare zero-trust). If the domain is typed into a web browser, a live google earth style map of the server is shown.

When building this server (still a work in progress), I experimented with a few different stacks but landed on a setup that I think is pretty clean.

## Hosting
A couple of months ago I purchased an old HP workstation on ebay for cheap. It has 16 GB RAM, Intel Core i7-7700, and enough storage for this project.

This is what we are going to be hosting the Minecraft server on. I am also running the Minecraft server inside a docker container on this machine for ease of use. I am using the [https://github.com/itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server) docker image for this.

The host machine runs tailscale VPN which lets me or my friends connect to the server from any device I invite to the tailscale network. This will come up later.

Once I had the Minecraft server running, I wanted to put it up on my domain. The only problem with this is that I would have to create a DNS record that points my domain to my home IP. I don't like this approach, and I wanted to find a better way.

## Forwarding Traffic

### Option One: Cloudflare Tunnels
Cloudflare Tunnels allow you to securely route traffic from the internet to your local server without exposing your server’s public IP address. I use Cloudflare to manage my domain and have previously used tunnels as a super easy way to serve a self-hosted app on your domain. This method is very clean because the tunnel sits inside of another Docker container and just passes traffic to the main app's container, all without revealing my home IP or forwarding any ports.

**The problem: Minecraft uses raw TCP**  

After troubleshooting some connection errors, I found out the issue was Minecraft's raw TCP traffic. It couldn't be passed through the tunnel since Tunnels can only send various supported http protocols, not raw TCP.

Oh, well.

### Option Two: Proxy Server
I remembered that I had my student benefits from Github and redeemed my $200 credit on Digital Ocean. I spun up a droplet on the cheapest server, starting at $4 per month. This got me a static IP, 512 MB of RAM, 1 vCPU, and a whopping 10 GB SSD.

This beast is going to be our proxy server. Its only job is to pass all of the Minecraft traffic that it gets onto the Minecraft Server. The setup process was fairly straightforward for this server.

1. Connect to tailscale network.
    - In order for the proxy server to see the minecraft server, we need to add this machine to our tailscale network. This is a 2 or 3 commmand process I won't get into here because tailscale provides a fine guide.
2. Install the proxy service
    - I went through a few options but ended up choosing socat for pure simplicity. We will selectively forward all traffic we get on port 25565, which is Minecraft's chosen port. Install using `apt install socat`. Then create a service to start the proxy at startup (See [mcproxy.service](mcproxy.service)). And that's all that we needed to do on this machine to set it up. Restart the server to start the proxy service, and the Minecraft server should start being available at the proxy server's IP.

Now, all that's needed is the DNS record pointing the mc.pointtomyserver.xyz subdomain to the proxy server IP. This is done in the Cloudflare dashboard (assuming your domain's DNS is managed through Cloudflare). We want to create an A record that points the mc subdomain to the IP of our proxy server droplet.