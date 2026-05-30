---
tags:
  - raspberry-pi
  - guide
  - docker
  - install
---
Install docker using the cli.

```bash
curl -sSL https://get.docker.com | sh
```

This will install docker. After it finished, then run.

```bash
sudo systemctl enable docker 
sudo systemctl start docker
```

This will add docker as service.

Then add the current user to the docker group by running:

```bash
# Add your user to the docker group (so you don't need sudo every time)
sudo usermod -aG docker $USER 
# Apply the group change without logging out 
newgrp docker
```

