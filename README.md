This repository contains an example setup that can be used to bootstrap a K8s
single node cluster via Rancher's RKE installer on a DigitalOcean VM in a few
commands. In addition, it includes deployment configuration that will bring up
a very simple default nginx web server, running HTTP natively, but utilizing
the default RKE ingress controller to perform TLS offloading at the domain
example.com (with a bundled self-signed certificate matching that domain name
to boot).

Here is the commands that can be used to bring this all up:

```fish
set ssh_keys (doctl compute ssh-key list --format ID --no-header | string join ',')
doctl compute droplet create --enable-monitoring --size s-2vcpu-2gb\
 --ssh-keys $ssh_keys --region tor1 --image ubuntu-18-04-x64\
 --format ID,PublicIPv4 --no-header --wait rke-node-01 |\
 string replace -r " +" " " | read node ip_address
sed -i '' "s/address:.*/address: $ip_address/" rke-deploy.yaml
sleep 20
ssh root@$ip_address\
 "curl https://releases.rancher.com/install-docker/18.09.2.sh | sh"
sleep 10
rke up --config rke-deploy.yaml
sleep 10
kubectl --kubeconfig kube_config_rke-deploy.yaml apply -f k8s-deploy.yaml
```

Here are the commands that will allow you to quickly clean up when done:

```fish
rke remove --config rke-deploy.yaml
doctl compute droplet delete $node
```
