# Create Cloudflare tunnel to share your localhost

Requirements:
- You need have a domain
- You must change your domain nameservers to Cloudflare

Follow tutorial guide of Cloudflare: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/

Step by step to setup locally by CLI
1. Install cloudflared cli
```bash
brew install cloudflare/cloudflare/cloudflared
```
2. Authenticated
```bash
cloudflared tunnel login
```
3. Create a tunnel
```bash
cloudflared tunnel create <NAME>
```
4. Route a subdomain to tunnel
```bash
cloudflared tunnel route dns <TUNNEL_NAME> <hostname>
# example: cloudflared tunnel route dns devapp devapp.duysmile.com
```
5. Expose app to tunnel
```bash
cloudflared tunnel --url http://localhost:8080 run <TUNNEL_NAME>
```
6. Setup local to run as a shortcut
- Create a bash script file like `tunnel`
```bash
#!/bin/bash
port=$1
echo "Tunnel is running at port $port with host: <your hostname>"
cloudflared tunnel --url http://localhost:$port run <TUNNEL_NAME>
```
- Add this script to `$PATH`
- Run tunnel everywhere in cli
```bash
# run tunnel in port 8000
tunnel 8000
```

