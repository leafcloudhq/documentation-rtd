# Installing Helm

Helm is a tool that makes it easier to deploy applications on your Kubernetes cluster.
We will be writing some tutorials for helm-based deployment of different pieces of software for your cluster.
So to help you get started, here are the instruction for installing helm on your machine.

The official docs are located here:
https://helm.sh/docs/intro/install/

it's basicall this:

Mac:
```
brew install helm
```

Windows Chocolatey:
```
choco install kubernetes-helm
```

Windows Scoop:
```
scoop install helm
```

Debian/Ubuntu:
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

Fedora:
```
sudo dnf install helm
```

Snap:
```
sudo snap install helm --classic
```

FreeBSD:
```
pkg install helm
```