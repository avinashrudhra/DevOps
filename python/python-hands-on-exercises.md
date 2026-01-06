# Python Hands-On Exercises

40+ practical Python exercises for DevOps automation and scripting.

---

## Beginner Level Exercises

### Exercise 1: System Information Script
**Objective**: Get system information using Python

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
System Information Script
"""
import platform
import psutil
from datetime import datetime

def get_system_info():
    """Get comprehensive system information"""
    
    print("=" * 50)
    print("SYSTEM INFORMATION")
    print("=" * 50)
    
    # OS Information
    print(f"\nOperating System: {platform.system()}")
    print(f"OS Release: {platform.release()}")
    print(f"OS Version: {platform.version()}")
    print(f"Architecture: {platform.machine()}")
    print(f"Hostname: {platform.node()}")
    print(f"Processor: {platform.processor()}")
    
    # CPU Information
    print(f"\nCPU Cores: {psutil.cpu_count()}")
    print(f"CPU Usage: {psutil.cpu_percent(interval=1)}%")
    
    # Memory Information
    memory = psutil.virtual_memory()
    print(f"\nTotal Memory: {memory.total / (1024**3):.2f} GB")
    print(f"Available Memory: {memory.available / (1024**3):.2f} GB")
    print(f"Memory Usage: {memory.percent}%")
    
    # Disk Information
    disk = psutil.disk_usage('/')
    print(f"\nTotal Disk Space: {disk.total / (1024**3):.2f} GB")
    print(f"Used Disk Space: {disk.used / (1024**3):.2f} GB")
    print(f"Disk Usage: {disk.percent}%")
    
    # Boot Time
    boot_time = datetime.fromtimestamp(psutil.boot_time())
    print(f"\nSystem Boot Time: {boot_time.strftime('%Y-%m-%d %H:%M:%S')}")

if __name__ == '__main__':
    get_system_info()
```
</details>

---

### Exercise 2: Log File Parser
**Objective**: Parse and analyze log files

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
Log File Parser for Nginx/Apache logs
"""
import re
from collections import Counter
from datetime import datetime

def parse_log_file(log_file):
    """Parse access log and generate statistics"""
    
    ips = []
    status_codes = []
    urls = []
    
    # Regex pattern for common log format
    pattern = r'(\d+\.\d+\.\d+\.\d+).*\[([^\]]+)\] "(\w+) ([^ ]+).*" (\d{3})'
    
    try:
        with open(log_file, 'r') as f:
            for line in f:
                match = re.search(pattern, line)
                if match:
                    ip = match.group(1)
                    method = match.group(3)
                    url = match.group(4)
                    status = match.group(5)
                    
                    ips.append(ip)
                    status_codes.append(status)
                    urls.append(url)
        
        # Generate statistics
        print("=" * 60)
        print("LOG ANALYSIS REPORT")
        print("=" * 60)
        
        print(f"\nTotal Requests: {len(ips)}")
        
        print("\nTop 10 IP Addresses:")
        for ip, count in Counter(ips).most_common(10):
            print(f"  {ip:20} - {count:5} requests")
        
        print("\nStatus Code Distribution:")
        for code, count in sorted(Counter(status_codes).items()):
            print(f"  {code}: {count:5} requests")
        
        print("\nTop 10 Requested URLs:")
        for url, count in Counter(urls).most_common(10):
            print(f"  {url[:50]:50} - {count:5} requests")
        
        # Find 4xx and 5xx errors
        errors_4xx = [code for code in status_codes if code.startswith('4')]
        errors_5xx = [code for code in status_codes if code.startswith('5')]
        
        print(f"\n4xx Client Errors: {len(errors_4xx)}")
        print(f"5xx Server Errors: {len(errors_5xx)}")
        
    except FileNotFoundError:
        print(f"Error: File {log_file} not found")
    except Exception as e:
        print(f"Error: {e}")

if __name__ == '__main__':
    parse_log_file('/var/log/nginx/access.log')
```
</details>

---

## Intermediate Level Exercises

### Exercise 3: Health Check Script
**Objective**: Monitor services and send alerts

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
Service Health Check Monitor
"""
import requests
import subprocess
import smtplib
from email.mime.text import MIMEText
from datetime import datetime

class HealthChecker:
    def __init__(self, config):
        self.config = config
        self.failed_checks = []
    
    def check_http_endpoint(self, url, expected_status=200):
        """Check if HTTP endpoint is responding"""
        try:
            response = requests.get(url, timeout=5)
            if response.status_code == expected_status:
                return True, f"OK - Status {response.status_code}"
            else:
                return False, f"FAIL - Status {response.status_code}"
        except requests.exceptions.RequestException as e:
            return False, f"FAIL - {str(e)}"
    
    def check_service_status(self, service_name):
        """Check if systemd service is running"""
        try:
            result = subprocess.run(
                ['systemctl', 'is-active', service_name],
                capture_output=True,
                text=True
            )
            if result.stdout.strip() == 'active':
                return True, "OK - Service is running"
            else:
                return False, f"FAIL - Service is {result.stdout.strip()}"
        except Exception as e:
            return False, f"FAIL - {str(e)}"
    
    def check_port_listening(self, port):
        """Check if port is listening"""
        try:
            result = subprocess.run(
                ['ss', '-tuln'],
                capture_output=True,
                text=True
            )
            if f':{port}' in result.stdout:
                return True, f"OK - Port {port} is listening"
            else:
                return False, f"FAIL - Port {port} not listening"
        except Exception as e:
            return False, f"FAIL - {str(e)}"
    
    def send_alert(self, subject, message):
        """Send email alert"""
        try:
            msg = MIMEText(message)
            msg['Subject'] = subject
            msg['From'] = self.config['from_email']
            msg['To'] = self.config['to_email']
            
            with smtplib.SMTP(self.config['smtp_server']) as server:
                server.send_message(msg)
            print("Alert sent successfully")
        except Exception as e:
            print(f"Failed to send alert: {e}")
    
    def run_checks(self):
        """Run all health checks"""
        print(f"\n{'='*60}")
        print(f"Health Check Report - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"{'='*60}\n")
        
        # Check HTTP endpoints
        print("HTTP Endpoint Checks:")
        for endpoint in self.config.get('http_endpoints', []):
            success, message = self.check_http_endpoint(endpoint['url'])
            status = "✓" if success else "✗"
            print(f"  {status} {endpoint['name']}: {message}")
            if not success:
                self.failed_checks.append(f"{endpoint['name']}: {message}")
        
        # Check services
        print("\nService Status Checks:")
        for service in self.config.get('services', []):
            success, message = self.check_service_status(service)
            status = "✓" if success else "✗"
            print(f"  {status} {service}: {message}")
            if not success:
                self.failed_checks.append(f"{service}: {message}")
        
        # Check ports
        print("\nPort Listening Checks:")
        for port in self.config.get('ports', []):
            success, message = self.check_port_listening(port)
            status = "✓" if success else "✗"
            print(f"  {status} Port {port}: {message}")
            if not success:
                self.failed_checks.append(message)
        
        # Send alert if any check failed
        if self.failed_checks:
            alert_message = "\n".join(self.failed_checks)
            print(f"\n⚠️  {len(self.failed_checks)} check(s) failed!")
            self.send_alert("Health Check Alert", alert_message)
        else:
            print("\n✓ All checks passed!")

if __name__ == '__main__':
    config = {
        'http_endpoints': [
            {'name': 'Main Website', 'url': 'https://example.com'},
            {'name': 'API Health', 'url': 'https://api.example.com/health'},
        ],
        'services': ['nginx', 'postgresql', 'redis'],
        'ports': [80, 443, 5432, 6379],
        'smtp_server': 'smtp.example.com',
        'from_email': 'monitoring@example.com',
        'to_email': 'admin@example.com'
    }
    
    checker = HealthChecker(config)
    checker.run_checks()
```
</details>

---

### Exercise 4: Docker Container Manager
**Objective**: Manage Docker containers using Python

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
Docker Container Manager
"""
import docker
from datetime import datetime
import sys

class DockerManager:
    def __init__(self):
        try:
            self.client = docker.from_env()
        except docker.errors.DockerException as e:
            print(f"Error connecting to Docker: {e}")
            sys.exit(1)
    
    def list_containers(self, all_containers=False):
        """List all containers"""
        containers = self.client.containers.list(all=all_containers)
        
        print(f"\n{'='*80}")
        print(f"{'NAME':<20} {'IMAGE':<25} {'STATUS':<15} {'PORTS'}")
        print(f"{'='*80}")
        
        for container in containers:
            name = container.name
            image = container.image.tags[0] if container.image.tags else container.image.short_id
            status = container.status
            ports = ', '.join([f"{k}→{v[0]['HostPort']}" for k, v in container.ports.items() if v])
            
            print(f"{name:<20} {image:<25} {status:<15} {ports}")
    
    def start_container(self, container_name):
        """Start a container"""
        try:
            container = self.client.containers.get(container_name)
            container.start()
            print(f"✓ Started container: {container_name}")
        except docker.errors.NotFound:
            print(f"✗ Container not found: {container_name}")
        except Exception as e:
            print(f"✗ Error starting container: {e}")
    
    def stop_container(self, container_name):
        """Stop a container"""
        try:
            container = self.client.containers.get(container_name)
            container.stop()
            print(f"✓ Stopped container: {container_name}")
        except docker.errors.NotFound:
            print(f"✗ Container not found: {container_name}")
        except Exception as e:
            print(f"✗ Error stopping container: {e}")
    
    def remove_container(self, container_name, force=False):
        """Remove a container"""
        try:
            container = self.client.containers.get(container_name)
            container.remove(force=force)
            print(f"✓ Removed container: {container_name}")
        except docker.errors.NotFound:
            print(f"✗ Container not found: {container_name}")
        except Exception as e:
            print(f"✗ Error removing container: {e}")
    
    def run_container(self, image, name=None, **kwargs):
        """Run a new container"""
        try:
            container = self.client.containers.run(
                image,
                name=name,
                detach=True,
                **kwargs
            )
            print(f"✓ Started new container: {container.name}")
            return container
        except Exception as e:
            print(f"✗ Error running container: {e}")
            return None
    
    def container_logs(self, container_name, lines=50):
        """Get container logs"""
        try:
            container = self.client.containers.get(container_name)
            logs = container.logs(tail=lines).decode('utf-8')
            print(f"\nLogs for {container_name} (last {lines} lines):")
            print("=" * 80)
            print(logs)
        except docker.errors.NotFound:
            print(f"✗ Container not found: {container_name}")
        except Exception as e:
            print(f"✗ Error getting logs: {e}")
    
    def container_stats(self, container_name):
        """Get container resource usage"""
        try:
            container = self.client.containers.get(container_name)
            stats = container.stats(stream=False)
            
            # Calculate CPU percentage
            cpu_delta = stats['cpu_stats']['cpu_usage']['total_usage'] - \
                       stats['precpu_stats']['cpu_usage']['total_usage']
            system_delta = stats['cpu_stats']['system_cpu_usage'] - \
                          stats['precpu_stats']['system_cpu_usage']
            cpu_percent = (cpu_delta / system_delta) * 100.0
            
            # Memory usage
            memory_usage = stats['memory_stats']['usage'] / (1024**2)  # MB
            memory_limit = stats['memory_stats']['limit'] / (1024**2)  # MB
            memory_percent = (memory_usage / memory_limit) * 100
            
            print(f"\nStats for {container_name}:")
            print(f"  CPU Usage: {cpu_percent:.2f}%")
            print(f"  Memory Usage: {memory_usage:.2f}MB / {memory_limit:.2f}MB ({memory_percent:.2f}%)")
            
        except docker.errors.NotFound:
            print(f"✗ Container not found: {container_name}")
        except Exception as e:
            print(f"✗ Error getting stats: {e}")
    
    def cleanup_unused(self):
        """Remove unused containers, images, and volumes"""
        try:
            print("\nCleaning up Docker resources...")
            
            # Remove stopped containers
            self.client.containers.prune()
            print("✓ Removed stopped containers")
            
            # Remove unused images
            self.client.images.prune()
            print("✓ Removed unused images")
            
            # Remove unused volumes
            self.client.volumes.prune()
            print("✓ Removed unused volumes")
            
            # Remove unused networks
            self.client.networks.prune()
            print("✓ Removed unused networks")
            
        except Exception as e:
            print(f"✗ Error during cleanup: {e}")

def main():
    manager = DockerManager()
    
    # Example usage
    print("Docker Container Manager")
    print("=" * 80)
    
    # List all containers
    manager.list_containers(all_containers=True)
    
    # Run a test container
    # manager.run_container('nginx', name='test-nginx', ports={'80/tcp': 8080})
    
    # Get logs
    # manager.container_logs('test-nginx', lines=20)
    
    # Get stats
    # manager.container_stats('test-nginx')
    
    # Cleanup
    # manager.cleanup_unused()

if __name__ == '__main__':
    main()
```
</details>

---

## Advanced Level Exercises

### Exercise 5: Kubernetes Deployment Manager
**Objective**: Deploy and manage applications in Kubernetes

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
Kubernetes Deployment Manager
"""
from kubernetes import client, config
from kubernetes.client.rest import ApiException
import yaml
import sys

class K8sManager:
    def __init__(self):
        try:
            config.load_kube_config()
            self.v1 = client.CoreV1Api()
            self.apps_v1 = client.AppsV1Api()
        except Exception as e:
            print(f"Error loading Kubernetes config: {e}")
            sys.exit(1)
    
    def list_namespaces(self):
        """List all namespaces"""
        try:
            namespaces = self.v1.list_namespace()
            print("\nNamespaces:")
            for ns in namespaces.items:
                print(f"  - {ns.metadata.name}")
        except ApiException as e:
            print(f"Error listing namespaces: {e}")
    
    def list_pods(self, namespace='default'):
        """List pods in namespace"""
        try:
            pods = self.v1.list_namespaced_pod(namespace)
            
            print(f"\nPods in namespace '{namespace}':")
            print(f"{'NAME':<40} {'STATUS':<15} {'RESTARTS'}")
            print("=" * 60)
            
            for pod in pods.items:
                name = pod.metadata.name
                status = pod.status.phase
                restarts = sum([c.restart_count for c in pod.status.container_statuses or []])
                print(f"{name:<40} {status:<15} {restarts}")
                
        except ApiException as e:
            print(f"Error listing pods: {e}")
    
    def create_deployment(self, name, image, replicas=1, namespace='default', port=80):
        """Create a deployment"""
        try:
            deployment = client.V1Deployment(
                api_version="apps/v1",
                kind="Deployment",
                metadata=client.V1ObjectMeta(name=name),
                spec=client.V1DeploymentSpec(
                    replicas=replicas,
                    selector=client.V1LabelSelector(
                        match_labels={"app": name}
                    ),
                    template=client.V1PodTemplateSpec(
                        metadata=client.V1ObjectMeta(
                            labels={"app": name}
                        ),
                        spec=client.V1PodSpec(
                            containers=[
                                client.V1Container(
                                    name=name,
                                    image=image,
                                    ports=[client.V1ContainerPort(container_port=port)]
                                )
                            ]
                        )
                    )
                )
            )
            
            self.apps_v1.create_namespaced_deployment(namespace, deployment)
            print(f"✓ Created deployment: {name}")
            
        except ApiException as e:
            print(f"✗ Error creating deployment: {e}")
    
    def scale_deployment(self, name, replicas, namespace='default'):
        """Scale a deployment"""
        try:
            # Get current deployment
            deployment = self.apps_v1.read_namespaced_deployment(name, namespace)
            
            # Update replicas
            deployment.spec.replicas = replicas
            
            # Patch deployment
            self.apps_v1.patch_namespaced_deployment(name, namespace, deployment)
            print(f"✓ Scaled deployment '{name}' to {replicas} replicas")
            
        except ApiException as e:
            print(f"✗ Error scaling deployment: {e}")
    
    def delete_deployment(self, name, namespace='default'):
        """Delete a deployment"""
        try:
            self.apps_v1.delete_namespaced_deployment(
                name,
                namespace,
                body=client.V1DeleteOptions()
            )
            print(f"✓ Deleted deployment: {name}")
        except ApiException as e:
            print(f"✗ Error deleting deployment: {e}")
    
    def get_pod_logs(self, pod_name, namespace='default', lines=50):
        """Get pod logs"""
        try:
            logs = self.v1.read_namespaced_pod_log(
                pod_name,
                namespace,
                tail_lines=lines
            )
            print(f"\nLogs for pod '{pod_name}' (last {lines} lines):")
            print("=" * 80)
            print(logs)
        except ApiException as e:
            print(f"✗ Error getting logs: {e}")
    
    def apply_yaml(self, yaml_file):
        """Apply YAML manifest"""
        try:
            with open(yaml_file, 'r') as f:
                docs = yaml.safe_load_all(f)
                for doc in docs:
                    kind = doc.get('kind')
                    name = doc.get('metadata', {}).get('name')
                    
                    if kind == 'Deployment':
                        deployment = self.apps_v1.create_namespaced_deployment(
                            body=doc,
                            namespace=doc.get('metadata', {}).get('namespace', 'default')
                        )
                        print(f"✓ Created deployment: {name}")
                    elif kind == 'Service':
                        service = self.v1.create_namespaced_service(
                            body=doc,
                            namespace=doc.get('metadata', {}).get('namespace', 'default')
                        )
                        print(f"✓ Created service: {name}")
                    # Add more resource types as needed
                    
        except Exception as e:
            print(f"✗ Error applying YAML: {e}")

def main():
    manager = K8sManager()
    
    print("Kubernetes Deployment Manager")
    print("=" * 80)
    
    # List namespaces
    manager.list_namespaces()
    
    # List pods
    manager.list_pods('default')
    
    # Create deployment
    # manager.create_deployment('nginx-app', 'nginx:latest', replicas=3)
    
    # Scale deployment
    # manager.scale_deployment('nginx-app', 5)
    
    # Get logs
    # manager.get_pod_logs('nginx-app-xxxxx-yyyyy')

if __name__ == '__main__':
    main()
```
</details>

---

### Exercise 6: Backup Automation Script
**Objective**: Automate backups with rotation and S3 upload

<details>
<summary>Solution</summary>

```python
#!/usr/bin/env python3
"""
Automated Backup Script with S3 Upload
"""
import os
import tarfile
import boto3
from datetime import datetime, timedelta
import logging
import sys

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/var/log/backup.log'),
        logging.StreamHandler()
    ]
)

class BackupManager:
    def __init__(self, config):
        self.config = config
        self.s3_client = boto3.client('s3')
        
    def create_backup(self, source_dir):
        """Create compressed backup of directory"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_name = f"backup_{timestamp}.tar.gz"
        backup_path = os.path.join(self.config['backup_dir'], backup_name)
        
        try:
            logging.info(f"Creating backup: {backup_path}")
            
            with tarfile.open(backup_path, 'w:gz') as tar:
                tar.add(source_dir, arcname=os.path.basename(source_dir))
            
            # Get backup size
            size_mb = os.path.getsize(backup_path) / (1024 * 1024)
            logging.info(f"Backup created: {backup_path} ({size_mb:.2f} MB)")
            
            return backup_path
            
        except Exception as e:
            logging.error(f"Error creating backup: {e}")
            return None
    
    def upload_to_s3(self, file_path, bucket_name):
        """Upload backup to S3"""
        try:
            file_name = os.path.basename(file_path)
            logging.info(f"Uploading to S3: {bucket_name}/{file_name}")
            
            self.s3_client.upload_file(
                file_path,
                bucket_name,
                file_name,
                ExtraArgs={'StorageClass': 'STANDARD_IA'}
            )
            
            logging.info(f"Upload complete: {file_name}")
            return True
            
        except Exception as e:
            logging.error(f"Error uploading to S3: {e}")
            return False
    
    def rotate_local_backups(self, backup_dir, keep_days=7):
        """Remove old local backups"""
        try:
            cutoff_date = datetime.now() - timedelta(days=keep_days)
            removed_count = 0
            
            for filename in os.listdir(backup_dir):
                if filename.startswith('backup_') and filename.endswith('.tar.gz'):
                    file_path = os.path.join(backup_dir, filename)
                    file_time = datetime.fromtimestamp(os.path.getmtime(file_path))
                    
                    if file_time < cutoff_date:
                        os.remove(file_path)
                        logging.info(f"Removed old backup: {filename}")
                        removed_count += 1
            
            logging.info(f"Removed {removed_count} old backups")
            
        except Exception as e:
            logging.error(f"Error rotating backups: {e}")
    
    def rotate_s3_backups(self, bucket_name, keep_days=30):
        """Remove old S3 backups"""
        try:
            cutoff_date = datetime.now() - timedelta(days=keep_days)
            removed_count = 0
            
            response = self.s3_client.list_objects_v2(Bucket=bucket_name)
            
            for obj in response.get('Contents', []):
                if obj['LastModified'].replace(tzinfo=None) < cutoff_date:
                    self.s3_client.delete_object(
                        Bucket=bucket_name,
                        Key=obj['Key']
                    )
                    logging.info(f"Removed old S3 backup: {obj['Key']}")
                    removed_count += 1
            
            logging.info(f"Removed {removed_count} old S3 backups")
            
        except Exception as e:
            logging.error(f"Error rotating S3 backups: {e}")
    
    def run_backup(self):
        """Run complete backup process"""
        logging.info("=" * 60)
        logging.info("Starting backup process")
        logging.info("=" * 60)
        
        # Create backup
        backup_file = self.create_backup(self.config['source_dir'])
        
        if not backup_file:
            logging.error("Backup creation failed")
            return False
        
        # Upload to S3
        if self.config.get('s3_bucket'):
            if not self.upload_to_s3(backup_file, self.config['s3_bucket']):
                logging.error("S3 upload failed")
                return False
        
        # Rotate backups
        self.rotate_local_backups(
            self.config['backup_dir'],
            self.config.get('local_retention_days', 7)
        )
        
        if self.config.get('s3_bucket'):
            self.rotate_s3_backups(
                self.config['s3_bucket'],
                self.config.get('s3_retention_days', 30)
            )
        
        logging.info("Backup process completed successfully")
        return True

if __name__ == '__main__':
    config = {
        'source_dir': '/opt/myapp',
        'backup_dir': '/backup',
        's3_bucket': 'my-backups-bucket',
        'local_retention_days': 7,
        's3_retention_days': 30
    }
    
    # Create backup directory if it doesn't exist
    os.makedirs(config['backup_dir'], exist_ok=True)
    
    # Run backup
    manager = BackupManager(config)
    success = manager.run_backup()
    
    sys.exit(0 if success else 1)
```
</details>

---

**Complete all exercises to master Python for DevOps! 🐍🚀**

For more resources:
- [Learning Guide](python-learning-guide.md)
- [Quick Reference](python-quick-reference.md)
- [Troubleshooting Guide](python-troubleshooting-guide.md)
- [Interview Questions](python-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

