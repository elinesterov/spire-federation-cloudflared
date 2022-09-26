# Expose SPIRE server federation endpoint publicly using cloudflared tunnels
Expose your SPIRE server private federation endpoint using Cloudflare Tunnel (Argo tunnel).

You often want to expose your SPIRE server's federation endpoint so others can access it (e.g. you are federating with AWS through OIDC). However, you don't want to expose the SPIRE server to the Internet or assign a public IP address. SPIRE federation endpoint also uses a server certificate signed by SPIRE so it is not trusted by the other side (you usually use the federation endpoint to get a trust bundle). So federation endpoint should have a server certificate signed by a public CA.

To solve the issues above we'll use [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/). It allows us to solve both issues: expose only a federation endpoint that also will have a certificate signed by a public CA (Cloudlfare in this case).
You can also use [ngrok](https://ngrok.com) or any [other alternatives](https://github.com/anderspitman/awesome-tunneling).

# Why do I need Envoy?
Cloudflared needs to connect to the SPIRE server's federation endpoint and because it is signed by the SPIRE server it won't trust that connection. To workaround it we add an envoy proxy that will expose the HTTP endpoint instead.

Here is a diagram how it looks:

[client] --https-> [cloudflare] <-https-- [cloudlfared] --http-> [envoy] --https-> [SPIRE server federation endpoint]

## Quick start with docker-compose

1. Clone repo
2. Start containers: `docker-compose up -d`
3. Find public address of cloudflared tunnel: `docker logs cloudflared` it is usually a subdomain of `trycloudflare.com`

## Use with your domain
Cloudflare will provide a random subdomain under `trycloudflare.com` every time you restart cloudflared container. To get a stable domain for federation you should configure cloudflare tunnels with your domain.

1. Read docs about Cloudflare tunnels. Create an account and obtain access token for tunnel.
2. edit `docker-compose.yaml` to start cloudflared container with command `tunnel --no-autoupdate run --token your_token_here`

## k8s
[TBD]

## Read more about cloudflared tunnels:
  * https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/remote/
  * https://github.com/cloudflare/cloudflared
