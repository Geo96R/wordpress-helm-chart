<<<<<<< HEAD
# wordpress-helm-chart
Wordpress set up, clone read the readme.md and deploy(scalable)
=======
# WordPress Helm Chart with Cloudflare Tunnel by George Tatevosov

This Helm chart deploys WordPress with MySQL database and a Cloudflare tunnel for secure access using Bitnami Nginx-WordPress image for CMS
Feel free to follow my linkedin https://www.linkedin.com/in/george-tatevosov-1a585a2b4/
## Prerequisites

- Kubernetes cluster (K3s recommended and tested on)
- Helm installed
- Cloudflare account with a domain
- `kubectl` configured to connect to your cluster

## Step-by-Step Installation

### 1. Install Helm Dependencies
Create the directory in where you will place this i assume you already cloned into it this repo
so if you cloned it into /home/wp/{repo here} from this dir run the following
```bash
helm dependency update 
```

### 2. Set Up Cloudflare Tunnel

Install the cloudflared CLI:

```bash
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
```

Login to Cloudflare:

```bash
cloudflared tunnel login
```

You'll receive a link to authorize the tunnel in your Cloudflare account.
Create a new tunnel:

```bash
cloudflared tunnel create my-k8s-tunnel
```

This will generate credentials in `~/.cloudflared/`.

List your tunnels to confirm creation:

```bash
cloudflared tunnel list
```

Point your domain to the tunnel:

```bash
cloudflared tunnel route dns my-k8s-tunnel yourdomain.com
```

Copy the credentials file to the Helm chart's cloudflare directory:

```bash
# Create directory if it doesn't exist
mkdir -p cloudflare

# Replace TUNNEL_ID with your tunnel ID from the list command REMEMBER this part for values.yaml also!
cp ~/.cloudflared/TUNNEL_ID.json cloudflare/credentials.json
```

### 3. Update values.yaml

Update the following values in `values.yaml`:

- `domainName`: Set to your domain
- `cloudflare.tunnelId`: Set to your tunnel ID
- Update any other settings for WordPress, MySQL, etc.

### 4. Deploy the Helm Chart

```bash
helm install wordpress-site . -n production --create-namespace #Note the name space will be sent across the configuration in the values
```

### 5. Verify Deployment

```bash
kubectl get pods -n production
```

You should see pods for:
- Cloudflare tunnel
- WordPress
- MySQL
- Nginx

## Configuration

The main configuration options in `values.yaml` are:

```yaml
domainName: "yourdomain.com"  # Your domain name

nginx:
  replicaCount: 1  # Number of nginx pods

wordpress:
  image: bitnami/wordpress-nginx:latest  # WordPress image
  replicas: 1  # Number of WordPress pods
  resources: {}  # Resource limits
  persistence:
    enabled: true  # Enable persistent storage
    storageSize: 10Gi  # Storage size

database:
  image: mysql:8.0  # MySQL image
  name: wordpress_db  # Database name
  storageSize: 10Gi  # Storage size

cloudflare:
  tunnelId: "your-tunnel-id"  # Your Cloudflare tunnel ID
  credentialsFile: "credentials.json"  # Name of your credentials file
```

## Troubleshooting

- Check the Cloudflare tunnel logs: `kubectl logs -n production -l app=cloudflare-tunnel`
- Check WordPress logs: `kubectl logs -n production -l app=wordpress-svc`
- Check MySQL logs: `kubectl logs -n production -l app=mysql`

## Upgrading

To upgrade your deployment after making changes:

```bash
helm upgrade wordpress-site . -n production
```

## Uninstalling

To remove the deployment:

```bash
helm uninstall wordpress-site -n production
```
##
## Your storage is at /var/lib/rancher/k3s/storage/pvc-* ##
## both mysql and the wordpress crucial files like wp-content and wp-config.php which matter for the sites functionality ##
##

## Final note, internall nginx of bitnami is used to running userless consider using RunAsUser if you come across any problems
## and the bitnami image listens on 8080 and 8443 take that in consideration.(unless you wanna tinker within the files)

Good luck!!
For redirects and changes to ingress or nginx i suggest backing everything up!
>>>>>>> 9b86ceb (Fully functional Wordpress Helm)
