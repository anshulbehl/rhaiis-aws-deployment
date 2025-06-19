# Red Hat AI Inference Server (RHAIIS) AWS Deployment

This repository provides a **one-command deployment** solution for Red Hat AI Inference Server (RHAIIS) on AWS. Deploy from zero to a fully functional AI inference server in minutes!

## What You Get

- **One-Command Deployment**: Full infrastructure + software deployment with `ansible-playbook deploy.yml -e @vars.yml`
- **Complete AWS Infrastructure**: VPC, subnet, security groups, EC2 instance with GPU support
- **Production-Ready Setup**: NVIDIA CUDA drivers, container runtime, and systemd service configuration
- **AI Model Deployment**: Automatically deploys your chosen foundation model (default: Llama-3.1-8B-Instruct)
- **Persistent Access**: Generated inventory files for ongoing management and SSH access
- **Easy Cleanup**: One command to tear down all resources

## Quick Start

### Prerequisites

- Ansible Core 2.15 or higher
- AWS account with appropriate permissions
- AWS CLI configured with access credentials
- Python 3.9 or higher

### Installation

1. **Clone this repository:**
   ```bash
   git clone https://github.com/yourusername/rhaiis-aws-deployment.git
   cd rhaiis-aws-deployment
   ```

2. **Install required Ansible collections:**
   ```bash
   ansible-galaxy collection install -r collections/requirements.yml
   ```

3. **Configure your deployment:**
   ```bash
   cp sample_vars.yml vars.yml
   # Edit vars.yml with your AWS settings, HuggingFace token, etc.
   ```

### Deploy Everything

**Deploy your complete RHAIIS infrastructure and software stack:**

```bash
ansible-playbook deploy.yml -e @vars.yml
```

That's it! This single command will:

1. **Build AWS Infrastructure** (VPC, subnet, security groups, EC2 instance)
2. **Install NVIDIA CUDA** (drivers, toolkit, container runtime)
3. **Deploy RHAIIS** (container service, model deployment, API configuration)
4. **Verify Deployment** (health checks, connection testing)
5. **Provide Access Details** (SSH commands, API endpoints, testing examples)

## Configuration

Edit `vars.yml` to customize your deployment:

### Core Settings
```yaml
instance_name: "my-rhaiis-instance"
region: "us-east-1"
instance_type: "g5.2xlarge"  # GPU instance for AI workloads
volume_size: 500             # Disk space in GB

# Authentication
setup_rhaiis_on_rhel_hf_token: "your-huggingface-token"
setup_rhaiis_on_rhel_registry_username: "your-registry-user"
setup_rhaiis_on_rhel_registry_password: "your-registry-password"

# Model Configuration
setup_rhaiis_on_rhel_vllm_model: "RedHatAI/Llama-3.1-8B-Instruct"
setup_rhaiis_on_rhel_api_token: "your-api-token"
```

### Instance Tagging
```yaml
instance_tags:
  owner: "your-username"
  Environment: "Development"
  Project: "RHAIIS"
  CostCenter: "Engineering"
```

## Testing Your Deployment

After deployment completes, test your RHAIIS instance:

### Check Available Models
```bash
curl -s http://<INSTANCE_IP>:8000/v1/models \
  -H "Authorization: Bearer your-api-token" | jq
```

### Generate Text
```bash
curl -X POST http://<INSTANCE_IP>:8000/v1/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-token" \
  -d '{
    "model": "RedHatAI/Llama-3.1-8B-Instruct",
    "prompt": "Explain the benefits of using Red Hat Enterprise Linux",
    "max_tokens": 200,
    "temperature": 0.7
  }'
```

### Chat Completion
```bash
curl -X POST http://<INSTANCE_IP>:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-token" \
  -d '{
    "model": "RedHatAI/Llama-3.1-8B-Instruct",
    "messages": [
      {"role": "system", "content": "You are a helpful AI assistant."},
      {"role": "user", "content": "What are the advantages of GPU-accelerated AI inference?"}
    ],
    "temperature": 0.7,
    "max_tokens": 250
  }'
```

## Ongoing Management

### SSH Access
```bash
# Direct SSH (command provided in deployment output)
ssh -i ./your-key.pem ec2-user@<instance-ip>

# Or check the generated inventory file for details
cat inventory.ini
```

### Ansible Management
```bash
# Check service status
ansible gpu_servers -i inventory.ini -m shell -a "systemctl status rhaiis"

# Monitor GPU usage
ansible gpu_servers -i inventory.ini -m shell -a "nvidia-smi"

# Check system resources
ansible gpu_servers -i inventory.ini -m shell -a "free -h && df -h"

# View RHAIIS logs
ansible gpu_servers -i inventory.ini -m shell -a "tail -f /tmp/rhaiis.log"
```

### Run Additional Playbooks
```bash
# Deploy updates or configuration changes
ansible-playbook maintenance.yml -i inventory.ini

# Scale or modify the deployment
ansible-playbook update-config.yml -i inventory.ini
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Account                          │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                Custom VPC                           │   │
│  │                                                     │   │
│  │  ┌───────────────────────────────────────────────┐ │   │
│  │  │              Public Subnet                    │ │   │
│  │  │                                               │ │   │
│  │  │  ┌─────────────────────────────────────────┐ │ │   │
│  │  │  │         EC2 Instance                    │ │ │   │
│  │  │  │         (g5.2xlarge)                    │ │ │   │
│  │  │  │                                         │ │ │   │
│  │  │  │  ┌─────────────────────────────────┐   │ │ │   │
│  │  │  │  │        RHAIIS Stack             │   │ │ │   │
│  │  │  │  │                                 │   │ │ │   │
│  │  │  │  │  • RHEL 9.5                     │   │ │ │   │
│  │  │  │  │  • NVIDIA CUDA Drivers          │   │ │ │   │
│  │  │  │  │  • Container Runtime            │   │ │ │   │
│  │  │  │  │  • vLLM Service                 │   │ │ │   │
│  │  │  │  │  • Foundation Model             │   │ │ │   │
│  │  │  │  │  • API Server (Port 8000)       │   │ │ │   │
│  │  │  │  └─────────────────────────────────┘   │ │ │   │
│  │  │  └─────────────────────────────────────────┘ │ │   │
│  │  └───────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Repository Structure

```
rhaiis-aws-deployment/
├── deploy.yml                    # Main unified deployment playbook
├── sample_vars.yml              # Configuration template
├── inventory.ini.j2             # Inventory template
├── teardown_aws.yml            # Infrastructure cleanup
├── roles/                       # Ansible roles
│   ├── setup_nvidia_cuda/       #   GPU driver setup
│   └── setup_rhaiis_on_rhel/    #   RHAIIS installation
├── collections/requirements.yml # Required Ansible collections
└── instances/                   # Generated inventory files
```

## Cleanup

Remove all AWS resources when you're done:

```bash
ansible-playbook teardown_aws.yml -e @vars.yml
```

This will:
- Terminate the EC2 instance
- Delete VPC and associated networking resources
- Remove security groups and key pairs
- Clean up local files and inventory

## Advanced Usage

### Multi-Instance Management

Deploy multiple RHAIIS instances by changing the `instance_name` in your vars.yml:

```bash
# Deploy first instance
ansible-playbook deploy.yml -e @vars.yml -e instance_name=rhaiis-dev

# Deploy second instance  
ansible-playbook deploy.yml -e @vars.yml -e instance_name=rhaiis-staging

# Switch between instances
ln -sf ./instances/rhaiis-dev/inventory.ini inventory.ini
ln -sf ./instances/rhaiis-staging/inventory.ini inventory.ini
```

### Step-by-Step Deployment (Advanced)

For debugging or customization, you can run the deployment in steps:

```bash
# 1. Provision infrastructure only
ansible-playbook provision_aws.yml -e @vars.yml

# 2. Deploy software stack
ansible-playbook rhaiis.yml -i inventory.ini -e @vars.yml
```

## Troubleshooting

### Common Issues

**SSH Connection Failed:**
- Check security group rules allow SSH (port 22)
- Verify your SSH key permissions: `chmod 600 your-key.pem`
- Ensure your IP is allowed in the security group

**Model Download Stuck:**
- **Symptoms**: Download gets stuck at high percentage (e.g., 97%) with no progress
- **Example**: `model-00002-of-00004.safetensors: 97% 4.87G/5.00G [stuck]`
- **Solution**: Restart the RHAIIS service to resume the download:
  ```bash
  ansible gpu_servers -i inventory.ini -m shell -a "sudo systemctl restart rhaiis"
  ansible gpu_servers -i inventory.ini -m shell -a "tail -f /tmp/rhaiis.log"
  ```
- **Root Cause**: Network timeouts, partial file corruption, or resource locks during large file downloads

**Model Loading Failed:**
- Verify your HuggingFace token has access to the model
- Check instance has sufficient memory for the model size
- Review logs: `ansible gpu_servers -i inventory.ini -m shell -a "tail -n 100 /tmp/rhaiis.log"`

**GPU Not Detected:**
- Ensure you're using a GPU instance type (g5.*, p3.*, p4.*)
- Check NVIDIA drivers: `ansible gpu_servers -i inventory.ini -m shell -a "nvidia-smi"`

### Getting Help

- Check the deployment output for specific error messages
- Review logs on the instance: `tail -f /tmp/rhaiis.log`
- Verify AWS permissions and quotas
- Ensure all required variables are set in `vars.yml`

## Key Variables Reference

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `instance_name` | Unique name for your EC2 instance | `your-rhaiis-instance` | Yes |
| `region` | AWS region for deployment | `us-east-1` | Yes |
| `instance_type` | AWS instance type (use GPU instances) | `g5.2xlarge` | Yes |
| `setup_rhaiis_on_rhel_hf_token` | HuggingFace access token | - | Yes |
| `setup_rhaiis_on_rhel_registry_username` | Registry username | - | Yes |
| `setup_rhaiis_on_rhel_registry_password` | Registry password | - | Yes |
| `setup_rhaiis_on_rhel_vllm_model` | Model to deploy | `RedHatAI/Llama-3.1-8B-Instruct` | No |
| `volume_size` | Root volume size in GB | `500` | No |
| `key_name` | SSH key name (auto-generated if empty) | `""` | No |

## Cost Considerations

- **g5.2xlarge**: ~$1.01/hour (24GB GPU memory)
- **Storage**: ~$0.10/GB/month for EBS volumes
- **Network**: Minimal for API usage
- **Estimated monthly cost**: ~$750/month for 24/7 operation

Remember to use `teardown_aws.yml` when you're done to avoid ongoing charges!

## TODO / Future Enhancements

### HTTPS Support with Nginx Proxy

Add nginx reverse proxy configuration to enable secure HTTPS access to the RHAIIS API:

- **SSL/TLS Termination**: Configure nginx to handle SSL certificates and terminate HTTPS connections
- **Reverse Proxy Setup**: Forward requests from nginx (port 443) to RHAIIS backend (port 8000)
- **Certificate Management**: Integrate Let's Encrypt or AWS Certificate Manager for automatic SSL certificate provisioning
- **Security Headers**: Add appropriate security headers (HSTS, CSP, etc.) for production deployment
- **Rate Limiting**: Implement request rate limiting and DDoS protection at the proxy level
- **Load Balancing**: Support for multiple RHAIIS backend instances when scaling horizontally

**Implementation Considerations:**
- Create new Ansible role: `setup_nginx_proxy`
- Update security groups to allow HTTPS traffic (port 443)
- Add domain name and DNS configuration variables
- Include nginx configuration templates for SSL and proxy settings
- Add health checks and monitoring for the proxy layer

---

## Summary

You now have a production-ready AI inference server running on AWS. The deployment provides you with a powerful GPU-accelerated instance running the latest Red Hat AI Inference Server with your chosen foundation model, ready to handle AI workloads at scale.
