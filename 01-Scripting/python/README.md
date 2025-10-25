# 🐍 Python for DevOps Cheatsheet

[← Back to Main ToC](../../README.md)

## Core Concepts

### Virtual Environments
```bash
# Create virtual environment
python3 -m venv venv
python3 -m venv .venv  # Hidden directory

# Activate
source venv/bin/activate          # Linux/Mac
venv\Scripts\activate             # Windows

# Deactivate
deactivate

# Requirements management
pip freeze > requirements.txt
pip install -r requirements.txt
pip install package==1.2.3       # Specific version
```

### File Operations
```python
import os
import shutil
from pathlib import Path

# Path operations
Path('/path/to/file').exists()
Path('/path').is_dir()
Path('/path/to/file').is_file()

# Read file
with open('file.txt', 'r') as f:
    content = f.read()
    lines = f.readlines()

# Write file
with open('file.txt', 'w') as f:
    f.write('content')
    f.writelines(['line1\n', 'line2\n'])

# Directory operations
os.listdir('/path')              # List directory
os.makedirs('/path', exist_ok=True)  # Create directory
shutil.copytree('src', 'dst')    # Copy directory
shutil.rmtree('/path')           # Remove directory
```

## Command Execution

### subprocess Module
```python
import subprocess

# Run command (Python 3.5+)
result = subprocess.run(
    ['ls', '-la'],
    capture_output=True,
    text=True,
    check=True  # Raise exception on non-zero exit
)
print(result.stdout)
print(result.stderr)
print(result.returncode)

# Run with shell (use cautiously)
result = subprocess.run(
    'echo $HOME',
    shell=True,
    capture_output=True,
    text=True
)

# Pipe commands
ps = subprocess.Popen(['ps', 'aux'], stdout=subprocess.PIPE)
grep = subprocess.Popen(
    ['grep', 'python'],
    stdin=ps.stdout,
    stdout=subprocess.PIPE
)
output = grep.communicate()[0]
```

## Working with APIs

### requests Library
```python
import requests
import json

# GET request
response = requests.get('https://api.example.com/data')
if response.status_code == 200:
    data = response.json()

# POST request
response = requests.post(
    'https://api.example.com/data',
    json={'key': 'value'},
    headers={'Authorization': 'Bearer token'}
)

# With error handling
try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()  # Raise exception for 4xx/5xx
    data = response.json()
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

## YAML & JSON Processing

```python
import json
import yaml

# JSON
with open('config.json', 'r') as f:
    config = json.load(f)

with open('config.json', 'w') as f:
    json.dump(config, f, indent=2)

# YAML (requires pyyaml)
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

with open('config.yaml', 'w') as f:
    yaml.safe_dump(config, f, default_flow_style=False)
```

## Environment Variables

```python
import os
from dotenv import load_dotenv  # Requires python-dotenv

# Get environment variables
api_key = os.environ.get('API_KEY')
api_key = os.getenv('API_KEY', 'default_value')

# Set environment variable
os.environ['API_KEY'] = 'value'

# Load from .env file
load_dotenv()
api_key = os.getenv('API_KEY')
```

## Logging

```python
import logging

# Basic configuration
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

# Logging levels
logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message')
logger.critical('Critical message')

# Exception logging
try:
    risky_operation()
except Exception as e:
    logger.exception('An error occurred')  # Includes traceback
```

## Error Handling

```python
# Try-except-else-finally
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
else:
    print("Success - no exceptions")
finally:
    cleanup()  # Always executed

# Context managers
class ManagedResource:
    def __enter__(self):
        print("Acquiring resource")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        return False  # Propagate exceptions

with ManagedResource() as resource:
    resource.use()
```

## Common DevOps Libraries

| Library | Purpose | Installation |
|---------|---------|-------------|
| `requests` | HTTP requests | `pip install requests` |
| `paramiko` | SSH operations | `pip install paramiko` |
| `boto3` | AWS SDK | `pip install boto3` |
| `kubernetes` | K8s Python client | `pip install kubernetes` |
| `docker` | Docker SDK | `pip install docker` |
| `ansible` | Automation | `pip install ansible` |
| `pyyaml` | YAML parsing | `pip install pyyaml` |
| `jinja2` | Templating | `pip install jinja2` |
| `click` | CLI creation | `pip install click` |
| `python-dotenv` | Environment variables | `pip install python-dotenv` |

## Docker SDK Example

```python
import docker

client = docker.from_env()

# List containers
for container in client.containers.list():
    print(f"{container.name}: {container.status}")

# Run container
container = client.containers.run(
    'nginx:latest',
    detach=True,
    ports={'80/tcp': 8080},
    name='my-nginx'
)

# Container operations
container.stop()
container.start()
container.remove(force=True)

# Build image
image, logs = client.images.build(
    path='.',
    tag='myapp:latest',
    rm=True
)
```

## Kubernetes Python Client

```python
from kubernetes import client, config

# Load kubeconfig
config.load_kube_config()

# Create API client
v1 = client.CoreV1Api()

# List pods
pods = v1.list_namespaced_pod(namespace='default')
for pod in pods.items:
    print(f"{pod.metadata.name}: {pod.status.phase}")

# Get pod logs
logs = v1.read_namespaced_pod_log(
    name='pod-name',
    namespace='default'
)
print(logs)

# Create deployment
apps_v1 = client.AppsV1Api()
deployment = apps_v1.create_namespaced_deployment(
    namespace='default',
    body=deployment_manifest
)
```

## AWS Boto3 Example

```python
import boto3

# S3 operations
s3 = boto3.client('s3')

# List buckets
response = s3.list_buckets()
for bucket in response['Buckets']:
    print(bucket['Name'])

# Upload file
s3.upload_file('local.txt', 'bucket-name', 'remote.txt')

# Download file
s3.download_file('bucket-name', 'remote.txt', 'local.txt')

# EC2 operations
ec2 = boto3.resource('ec2')

# List instances
for instance in ec2.instances.all():
    print(f"{instance.id}: {instance.state['Name']}")

# Start/stop instances
instance = ec2.Instance('i-1234567890abcdef0')
instance.start()
instance.stop()
```

## CLI with Click

```python
import click

@click.group()
def cli():
    """DevOps CLI tool"""
    pass

@cli.command()
@click.option('--env', default='dev', help='Environment')
@click.option('--verbose', is_flag=True, help='Verbose output')
@click.argument('service')
def deploy(env, verbose, service):
    """Deploy a service"""
    if verbose:
        click.echo(f'Deploying {service} to {env}')
    # Deployment logic

@cli.command()
@click.argument('filename', type=click.File('r'))
def validate(filename):
    """Validate configuration file"""
    content = filename.read()
    click.echo(f'Validating {filename.name}')

if __name__ == '__main__':
    cli()
```

## Best Practices & Gotchas

### ✅ Do's
- Use virtual environments for isolation
- Use `pathlib.Path` instead of string paths
- Use `with` statements for file operations
- Use `subprocess.run()` with `check=True` for commands
- Use type hints (Python 3.5+) for better code clarity
- Use `logging` instead of `print()` for applications
- Handle exceptions explicitly
- Use `.env` files for configuration

### ❌ Don'ts
- Don't use `shell=True` with user input (security risk)
- Don't use bare `except:` clauses
- Don't hardcode credentials
- Don't ignore return codes from subprocess
- Don't use mutable default arguments
- Don't use `os.system()` or `os.popen()` (deprecated)

### 🔧 Common Gotchas
- **Mutable defaults**: `def func(lst=[]):` creates shared list
- **Late binding closures**: Loop variables in lambdas
- **String encoding**: Always specify encoding in Python 2/3
- **Subprocess shell=True**: Security and portability issues
- **Context manager errors**: `with` doesn't suppress all exceptions

### 🛡️ Security Best Practices
```python
# Don't hardcode secrets
API_KEY = os.getenv('API_KEY')  # ✅ Good
# API_KEY = 'hardcoded-key'     # ❌ Bad

# Validate user input
import shlex
command = shlex.quote(user_input)  # ✅ Escape shell input

# Use subprocess safely
subprocess.run(['ls', user_input])  # ✅ Good (list)
# subprocess.run(f'ls {user_input}', shell=True)  # ❌ Bad

# Restrict file permissions
os.chmod('secrets.txt', 0o600)  # Owner read/write only
```

### 📝 Script Template
```python
#!/usr/bin/env python3
"""
Script description and usage
"""

import sys
import logging
import argparse
from pathlib import Path

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def main():
    """Main function"""
    parser = argparse.ArgumentParser(
        description='Script description'
    )
    parser.add_argument(
        'input',
        help='Input file or value'
    )
    parser.add_argument(
        '-v', '--verbose',
        action='store_true',
        help='Enable verbose output'
    )
    
    args = parser.parse_args()
    
    if args.verbose:
        logging.getLogger().setLevel(logging.DEBUG)
    
    try:
        # Script logic here
        logger.info(f"Processing {args.input}")
        # ...
    except Exception as e:
        logger.error(f"Error: {e}")
        sys.exit(1)

if __name__ == '__main__':
    main()
```

This cheatsheet covers essential Python patterns and libraries for DevOps automation tasks.
