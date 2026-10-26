---
title: "Docker Setup on an AWS EC2 Instance"
domain: "DevOps"
tool: "Docker"
source:
  - "../../../raw/DevOps-Tool/Docker/docker.md"
  - "../../../raw/DevOps-Tool/Docker/docker.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["docker", "aws", "ec2", "ssm", "iam"]
blog: ""
---

# Docker Setup on an AWS EC2 Instance

## TL;DR

- Use an IAM **role** with `AmazonSSMManagedInstanceCore`, attached to the instance as an **instance profile**.
- Connect with **Session Manager** instead of opening SSH to the internet.
- Install Docker from Docker's official apt repository, then verify with `hello-world`.
- Add your user to the `docker` group only if you accept that it is root-equivalent.
- Stop or terminate the instance when the lab is done; running instances cost money.

## Why it matters

This is the cheapest reliable lab for learning Docker: a real Linux kernel, real networking and a real cloud IAM path. It also demonstrates the correct AWS pattern - roles and SSM instead of long-lived keys and open SSH ports.

## Core concepts

- **IAM user** - a long-lived identity for humans or CI. Do not use the root account for daily work.
- **IAM role** - temporary credentials assumed by a trusted entity. Here, the trusted entity is the EC2 service.
- **Instance profile** - the container that attaches a role to an EC2 instance.
- **SSM Session Manager** - a shell over the SSM control plane. No inbound SSH port is required.
- **Security group** - a stateful firewall. For SSM-only access, no inbound rules are needed.

## Commands and examples

### 1. Prepare the AWS account

1. Create the AWS account root user, then enable MFA.
2. Create an admin IAM user for yourself and use that for day-to-day work. Do not use root.
3. Never create access keys unless you need them; prefer roles and SSO.

### 2. Create the IAM role

In the IAM console:

1. **Roles -> Create role**.
2. Trusted entity: **AWS service**, use case: **EC2**.
3. Attach `AmazonSSMManagedInstanceCore`.
4. Name it, for example, `ec2-ssm-docker`.
5. Create the role and note the instance profile name (usually the same).

Reference trust policy (`<!-- reference policy, not from the raw notes -->`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 3. Launch the instance

1. Choose an Ubuntu or Amazon Linux AMI.
2. Attach the `ec2-ssm-docker` instance profile under **Advanced details**.
3. Use a security group with **no inbound rules** (SSM does not need them).
4. Launch and wait until the instance is running and registered with SSM.

### 4. Connect

In the console: **EC2 -> Instance -> Connect -> Session Manager -> Connect**.

Or with the AWS CLI and the Session Manager plugin:

```bash
# Replace with your own instance ID; this is a placeholder
aws ssm start-session --target i-0123456789abcdef0
```

### 5. Install Docker (Ubuntu)

The official steps are at <https://docs.docker.com/engine/install/ubuntu/>. In short:

```bash
# Prerequisites
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install the engine and plugins
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### 6. Verify

```bash
sudo docker version
sudo docker info
sudo docker run hello-world
```

### 7. Optional: run Docker without sudo

```bash
# The docker group grants root-equivalent access. Understand the trade-off first.
sudo usermod -aG docker "$USER"

# Group membership only applies to a new login session
exit
# reconnect, then:
docker ps
```

## Gotchas

- **Role vs instance profile.** Creating a role is not enough; the instance must have the instance profile attached. Attaching it after launch usually works, but the SSM agent may need a restart.
- **No SSM registration.** Check that the instance profile has `AmazonSSMManagedInstanceCore`, that the instance has outbound HTTPS access, and that the SSM agent is running.
- **No inbound SSH is required** for Session Manager. Opening port 22 is an unnecessary exposure.
- **`docker` group is root-equivalent.** A member can mount host paths and escape the container. Prefer `sudo docker` on a shared host.
- **Install from Docker's repository, not the distro package**, when you need current versions. `docker.io` from Ubuntu is usually older.
- **This is a lab.** Stop or terminate the instance, and remove unused volumes and elastic IPs, to avoid ongoing cost.
- **Never commit credentials.** No access keys, account IDs or `.pem` files belong in this repo.

## Cheat sheet

| Task | Command / action |
| --- | --- |
| Trusted entity for EC2 role | AWS service -> EC2 |
| Managed policy for SSM | `AmazonSSMManagedInstanceCore` |
| Attach role to instance | Instance profile |
| Connect without SSH | Session Manager |
| Verify Docker | `docker run hello-world` |
| Use Docker without sudo | `sudo usermod -aG docker $USER`, then re-login |
| Check install docs | <https://docs.docker.com/engine/install/ubuntu/> |

## Practice

1. Launch an instance with no inbound rules, connect through SSM, and install Docker.
2. Create a second IAM role without the SSM policy and observe what fails.
3. Run `hello-world`, then stop the instance. Confirm the state in the console.

## Further reading

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [Docker security](https://docs.docker.com/engine/security/)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Docker/docker.md`, `raw/DevOps-Tool/Docker/docker.txt`
- Video referenced in the raw notes: <https://www.youtube.com/watch?v=rjjES5IsPdg>
- Install guide referenced in the raw notes: <https://docs.docker.com/engine/install/ubuntu/>
