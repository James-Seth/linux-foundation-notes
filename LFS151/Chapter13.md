# Chapter 13: Configuration Management Tools

## Table of Contents

1. [Learning Objectives](#learning-objectives)
2. [Infrastructure as Code Overview](#infrastructure-as-code-overview)
3. [Ansible](#ansible)
4. [Puppet](#puppet)
5. [Chef](#chef)
6. [Salt](#salt)
7. [Tool Comparison](#tool-comparison)

---

## Learning Objectives

By the end of this chapter, you should be able to:

- Explain Infrastructure as Code concepts
- List tools used to configure and manage cloud infrastructure
- Discuss and use configuration management tools: Ansible, Puppet, Chef, and Salt
- Understand the architecture and use cases for each tool

---

## Infrastructure as Code Overview

### The Problem

Managing numerous systems (bare-metal and virtual) across different environments:
- Development
- QA (Quality Assurance)
- Production

### The Solution: Infrastructure as Code

**Definition**: Infrastructure resources are defined declaratively in configuration files that can be reused to provision consistently reproducible systems across environments.

**Key Benefits**:
- Automation of system provisioning
- Consistent and desired state of systems
- Repeatable configurations
- Version-controlled infrastructure definitions

**Configuration Management Tools**: Allow us to define the desired state of systems in an automated way.

---
## Ansible

### Overview

**Provider**: Red Hat

**Type**: Open source configuration management tool

**Key Characteristic**: Agentless - works through SSH

**Capabilities**:
- Infrastructure provisioning (on-premises or public cloud)
- Application deployment
- Orchestration
- Configuration management

**Why Popular**: Simplicity and ease of use made it one of the top in-demand CM tools in DevOps.

### Architecture

**Ansible Management Node**:
- Can be installed on any Unix-based system (Linux or macOS)
- Manages nodes via SSH
- Requires no agent installation on managed nodes
- Managed nodes only need SSH and Python support

### Playbooks

**Definition**: Ansible's configuration, deployment, and orchestration language written in YAML.

**Example Playbook** - Deploying application stack based on roles:

---
# This playbook deploys the whole application stack in this site.
- name: apply common configuration to all nodes
  hosts: all
  remote_user: root

  roles:
    - common

- name: configure and deploy the webservers and application code
  hosts: webservers
  remote_user: root

  roles:
    - web

- name: deploy MySQL and configure the databases
  hosts: dbservers
  remote_user: root

  roles:
    - db

**Explanation**: This playbook defines three plays targeting different host groups (all nodes, webservers, dbservers) and applies specific roles to each.

**Example Tasks** - Installing HTTP and PHP:

---
# These tasks install http and the php modules.

- name: Install http and php etc
  yum: name={{ item }} state=present
  with_items:
   - httpd
   - php
   - php-mysql
   - git
   - libsemanage-python
   - libselinux-python

- name: insert iptables rule for httpd
  lineinfile: dest=/etc/sysconfig/iptables create=yes state=present regexp="{{ httpd_port }}" insertafter="^:OUTPUT "
              line="-A INPUT -p tcp --dport {{ httpd_port }} -j ACCEPT"
  notify: restart iptables

- name: http service state
  service: name=httpd state=started enabled=yes

**Explanation**: These tasks install packages via yum, configure firewall rules, and ensure the httpd service is running.

### Ansible Ecosystem

**Ansible Galaxy**:
- Free community site for sharing Ansible roles
- Download ready-to-use roles
- Share custom roles with the community

**Ansible Automation Platform** (Enterprise):
- UI interface
- Access control
- Central management
- Commercial support from Red Hat

### Practical Example: Configuring a System with Ansible

**Setup Requirements**:

Managed system (target):
- Ubuntu Linux VM
- Non-root user "ubuntu" with sudo privileges
- Python installed
- OpenSSH server running
- IP address: 192.168.12.129

Ansible managing server:

- Ansible package installed
- SSH key-pair generated for passwordless authentication
- Public key copied to managed VM

**Step 1: Create Inventory File**

The inventory file defines managed hosts and groups:

```ini
[nginx_web]
192.168.12.129 ansible_user=ubuntu
```

**Step 2: Test Connectivity**

Verify hosts are reachable and Python is available:

```bash
ansible -i inventory.cfg -m ping all
```

**Step 3: Create Installation Playbook**

File: `install_nginx.yml`

```yaml
---
- name: Install and start nginx
  hosts: nginx_web
  become: yes
  tasks:
    - name: Install nginx package
      apt:
        name: nginx
        state: present
    
    - name: Start nginx service
      systemd:
        name: nginx
        state: started
        enabled: yes
```

**Step 4: Run the Playbook**

```bash
ansible-playbook -i inventory.cfg install_nginx.yml -b
```

The `-b` flag enables privilege escalation (become sudo).

**Step 5: Verify Installation**

Check service status:

```bash
sudo systemctl status nginx
```

Test the web server:

```bash
curl -s 192.168.12.129 | grep title
```

**Step 6: Create Uninstall Playbook** (Optional)

File: `uninstall_nginx.yml`

```yaml
---
- name: Stop and uninstall nginx
  hosts: nginx_web
  become: yes
  tasks:
    - name: Stop nginx service
      systemd:
        name: nginx
        state: stopped
    
    - name: Uninstall nginx package
      apt:
        name: nginx
        state: absent
```

**Step 7: Run Uninstall Playbook**

```bash
ansible-playbook -i inventory.cfg uninstall_nginx.yml -b
```

**Note**: The ansible CLI features may change without notice. Output on your system may vary slightly.

### Ansible Security Best Practices

**Critical Security Considerations**:

1. **Vault for Secrets Management**:
   - Never store passwords or sensitive data in plain text
   - Use Ansible Vault to encrypt sensitive variables
   - Encrypt entire files or specific variables

**Example - Creating an encrypted file**:

```bash
ansible-vault create secrets.yml
```

**Example - Encrypting existing file**:

```bash
ansible-vault encrypt inventory_passwords.yml
```

**Example - Using encrypted variables in playbook**:

```yaml
---
- name: Deploy application with secrets
  hosts: webservers
  vars_files:
    - secrets.yml
  tasks:
    - name: Create database user
      mysql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"
        priv: "*.*:ALL"
```

**Running playbook with vault**:

```bash
ansible-playbook deploy.yml --ask-vault-pass
```

2. **SSH Key Management**:
   - Use SSH keys instead of passwords
   - Rotate keys regularly
   - Use different keys for different environments
   - Never commit private keys to version control

3. **Privilege Escalation**:
   - Use `become` instead of running as root
   - Limit sudo privileges to specific commands
   - Use `become_user` to specify which user to become

**Example - Controlled privilege escalation**:

```yaml
- name: Install package with specific user
  hosts: servers
  become: yes
  become_user: appuser
  become_method: sudo
  tasks:
    - name: Install application
      apt:
        name: myapp
        state: present
```

4. **Inventory Security**:
   - Don't store sensitive data in inventory files
   - Use dynamic inventory when possible
   - Restrict inventory file permissions (chmod 600)

5. **Playbook Security**:
   - Validate inputs to prevent injection attacks
   - Use `no_log: true` for tasks handling sensitive data
   - Pin package versions to prevent supply chain attacks

**Example - Hiding sensitive output**:

```yaml
- name: Set user password
  user:
    name: admin
    password: "{{ admin_password | password_hash('sha512') }}"
  no_log: true
```

### Why Ansible is Important for DevOps

**Key Importance Factors**:

1. **Speed and Efficiency**:
   - Agentless architecture = faster deployment
   - No bootstrapping required
   - Parallel execution across hundreds of nodes
   - Average setup time: 15-30 minutes vs hours for agent-based tools

2. **Lower Total Cost of Ownership (TCO)**:
   - No agent licenses or management overhead
   - Reduced infrastructure complexity
   - Less maintenance required
   - Lower learning curve = faster team productivity

3. **Industry Adoption**:
   - Used by major companies: NASA, Red Hat, Cisco, Verizon
   - Strong job market demand
   - Part of Red Hat ecosystem (Enterprise Linux, OpenShift)
   - Integration with cloud providers (AWS, Azure, GCP)

4. **Idempotency**:
   - Run playbooks multiple times safely
   - Only changes what's needed
   - Predictable results every time

**Example - Idempotent playbook**:

```yaml
- name: Ensure nginx is configured correctly
  hosts: webservers
  tasks:
    - name: Ensure nginx is installed
      apt:
        name: nginx
        state: present  # Installs only if not present
    
    - name: Ensure config file is present
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: reload nginx  # Only reloads if file changed
    
    - name: Ensure nginx is running
      systemd:
        name: nginx
        state: started  # Starts only if not running
        enabled: yes    # Enables only if not enabled
  
  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

5. **Multi-Cloud and Hybrid Support**:
   - Manage on-premises and cloud infrastructure uniformly
   - Cloud modules for AWS, Azure, GCP, DigitalOcean
   - Container orchestration (Docker, Kubernetes)

### Common Ansible Use Cases

**1. Configuration Management**:
- Standardize server configurations
- Enforce security policies
- Manage application settings

**2. Application Deployment**:
- Zero-downtime deployments
- Rolling updates
- Blue-green deployments

**3. Cloud Provisioning**:
- Provision VMs, networks, storage
- Configure load balancers
- Manage cloud resources

**4. Security and Compliance**:
- Patch management
- Security hardening
- Compliance auditing

**5. Orchestration**:
- Multi-tier application deployments
- Complex workflow automation
- Disaster recovery procedures

### Benefits of Using Ansible

| Benefit | Description |
|---------|-------------|
| **Easy to Use** | Low learning curve with simple YAML syntax |
| **Agentless** | No agent installation required on managed nodes |
| **Automation** | Cloud provisioning, application deployment, orchestration |
| **Reliability** | Consistent, reliable, and secure management |
| **Community** | Large, active community of developers |
| **Access Control** | Role-based access control available |
| **Cross-Platform** | Available for all major operating systems |

---

## Puppet

### Overview

**Type**: Open source configuration management tool

**Architecture**: Agent-based client/server model

**Key Components**:
1. Puppet Agent (installed on managed nodes)
2. Puppet Server (central management)
3. Catalog File (compiled instructions)

### Puppet Agent

**Required on**: Each system to be managed

**Responsibilities**:
- Connect securely to Puppet Server
- Pull updates and Catalog File
- Perform operations to reach desired state
- Send status reports back to server

**Supported Platforms**:
- Linux
- Windows
- macOS

### Puppet Server

**Platform**: Unix-based systems only

**Responsibilities**:
- Compile Catalog Files for hosts
- Send Catalog Files to Agents on request
- Store environment information (host info, metadata, authentication keys)
- Gather and prepare overall reports from all Agents

### Manifest Files and Catalog Files

**Manifest File**: Written in Puppet Code to define desired state

Example manifest - creating a user:

user { 'student':
  ensure     => present,
  uid        => '1001',
  gid        => '1001',
  shell      => '/bin/bash',
  home       => '/home/student'
}

**Explanation**: This code creates a user "student" with:
- UID/GID: 1001
- Shell: /bin/bash
- Home directory: /home/student

**Puppet Code Structure**:

General syntax pattern:

resource_type { 'resource_name'
  attribute => value
  ...
}

**Resource Types**: Puppet defines various resource types:
- `file` - File management
- `user` - User accounts
- `package` - Software packages
- `service` - System services
- And many more (see Puppet documentation)

**Catalog File**: After processing the manifest, Puppet Server compiles a platform-specific Catalog File for each managed node.

### Practical Example: Managing Packages with Puppet

**Example 1: Installing and Managing Apache**

```puppet
# Manifest file: apache.pp
class apache {
  package { 'apache2':
    ensure => installed,
  }
  
  service { 'apache2':
    ensure  => running,
    enable  => true,
    require => Package['apache2'],
  }
  
  file { '/var/www/html/index.html':
    ensure  => file,
    content => '<h1>Managed by Puppet</h1>',
    require => Package['apache2'],
  }
}

include apache
```

**Explanation**:
- `ensure => installed` ensures package is present
- `require` creates dependency (service needs package first)
- Resources are applied in dependency order

**Example 2: Managing Multiple Users**

```puppet
# Manifest file: users.pp
$users = {
  'alice' => {
    'uid'   => '2001',
    'shell' => '/bin/bash',
  },
  'bob' => {
    'uid'   => '2002',
    'shell' => '/bin/zsh',
  },
}

$users.each |String $username, Hash $attributes| {
  user { $username:
    ensure => present,
    uid    => $attributes['uid'],
    gid    => $attributes['uid'],
    shell  => $attributes['shell'],
    home   => "/home/${username}",
  }
  
  file { "/home/${username}":
    ensure => directory,
    owner  => $username,
    group  => $username,
    mode   => '0755',
  }
}
```

**Example 3: Conditional Configuration Based on OS**

```puppet
# Manifest file: conditional.pp
case $facts['os']['family'] {
  'RedHat': {
    package { 'httpd':
      ensure => installed,
    }
    service { 'httpd':
      ensure => running,
    }
  }
  'Debian': {
    package { 'apache2':
      ensure => installed,
    }
    service { 'apache2':
      ensure => running,
    }
  }
  default: {
    fail('Unsupported operating system')
  }
}
```

**Explanation**: Uses Puppet facts to determine OS and apply appropriate configuration.

### Puppet Ecosystem

**PuppetDB**:
- Centralized reporting
- Generate reports
- Search systems
- Query infrastructure state

**Live System Management**: Real-time management capabilities

**Puppet Forge**:
- Community repository
- Ready-to-use modules
- Share custom modules

### Benefits of Using Puppet

| Benefit | Description |
|---------|-------------|
| **Open Source** | Free configuration management tool |
| **Scalability** | Handles large-scale deployments |
| **Automation** | Automated configuration management |
| **Reporting** | Centralized reporting via PuppetDB |
| **Cross-Platform** | Available on all major operating systems |
| **Access Control** | Role-based access control |

---

## Chef

### Overview

**Type**: Configuration management system

**Architecture**: Client/server model or clientless (agentless) model

**Key Components**:
1. **Chef Client**: Installed on each managed host
2. **Chef Server**: Central management server
3. **Chef Workstation**: Development environment

### Chef Workstation

**Purpose**: Development and management interface

**Responsibilities**:
- Develop cookbooks and recipes
- Synchronize chef-repo with version control
- Run command line tools (knife CLI)
- Configure policies and roles
- Interact with nodes for one-off configurations

### Chef Cookbooks

**Definition**: Basic unit of configuration that defines a scenario and contains everything needed to support it.

**Key Components**:

1. **Recipes**: Most fundamental configuration unit

Example recipe - Installing and starting Apache:

package "apache2" do
  action :install
end

service "apache2" do
  action [:enable, :start]
end

**Explanation**: Installs apache2 package and starts/enables the service.

2. **Attributes**: Define the state of the node

After each chef-client run, the node's state is updated on the Chef Server.

### Advanced Chef Examples

**Example 1: Complete Web Server Cookbook**

Directory structure:
```
webserver/
├── recipes/
│   └── default.rb
├── templates/
│   └── index.html.erb
├── attributes/
│   └── default.rb
└── metadata.rb
```

**Attributes file** (`attributes/default.rb`):
```ruby
default['webserver']['port'] = 80
default['webserver']['document_root'] = '/var/www/html'
default['webserver']['server_name'] = 'example.com'
```

**Recipe** (`recipes/default.rb`):
```ruby
# Install and configure web server
package 'nginx' do
  action :install
end

# Create document root
directory node['webserver']['document_root'] do
  owner 'www-data'
  group 'www-data'
  mode '0755'
  action :create
end

# Deploy index page from template
template "#{node['webserver']['document_root']}/index.html" do
  source 'index.html.erb'
  owner 'www-data'
  group 'www-data'
  mode '0644'
  variables(
    server_name: node['webserver']['server_name'],
    port: node['webserver']['port']
  )
end

# Ensure service is running
service 'nginx' do
  action [:enable, :start]
  subscribes :reload, "template[#{node['webserver']['document_root']}/index.html]", :delayed
end
```

**Template** (`templates/index.html.erb`):
```html
<!DOCTYPE html>
<html>
<head>
  <title><%= @server_name %></title>
</head>
<body>
  <h1>Welcome to <%= @server_name %></h1>
  <p>Listening on port <%= @port %></p>
  <p>Managed by Chef</p>
</body>
</html>
```

**Example 2: Managing Firewall Rules**

```ruby
# Recipe: firewall.rb
package 'ufw' do
  action :install
end

# Allow SSH
execute 'ufw-allow-ssh' do
  command 'ufw allow 22/tcp'
  not_if 'ufw status | grep "22/tcp"'
end

# Allow HTTP
execute 'ufw-allow-http' do
  command 'ufw allow 80/tcp'
  not_if 'ufw status | grep "80/tcp"'
end

# Enable firewall
execute 'ufw-enable' do
  command 'ufw --force enable'
  not_if 'ufw status | grep "Status: active"'
end
```

**Example 3: Database Setup with Error Handling**

```ruby
# Recipe: database.rb
package 'mysql-server' do
  action :install
end

service 'mysql' do
  action [:enable, :start]
end

# Wait for MySQL to be ready
ruby_block 'wait_for_mysql' do
  block do
    30.times do
      begin
        require 'mysql2'
        client = Mysql2::Client.new(host: 'localhost', username: 'root')
        break
      rescue
        sleep 1
      end
    end
  end
end

# Create database
execute 'create_database' do
  command "mysql -u root -e 'CREATE DATABASE IF NOT EXISTS myapp'"
  action :run
end

# Create user with password from data bag
ruby_block 'create_db_user' do
  block do
    require 'mysql2'
    client = Mysql2::Client.new(host: 'localhost', username: 'root')
    client.query("CREATE USER IF NOT EXISTS 'appuser'@'localhost' IDENTIFIED BY 'secure_password'")
    client.query("GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost'")
    client.query("FLUSH PRIVILEGES")
  end
  action :run
end
```

### Knife CLI Tool

**Purpose**: Interface between local chef-repo and Chef Server

**Capabilities**:
- Upload cookbooks to Chef Server
- Manage nodes
- Bootstrap new systems
- Query Chef Server

### Supported Platforms

**Chef Client** platforms:
- AIX
- Linux distributions and FreeBSD
- Unix-based systems
- macOS
- Windows

**Chef Server** platforms:
- Red Hat Enterprise Linux
- SUSE Linux Enterprise Server
- Ubuntu Linux

### Chef Manage (GUI)

Web-based interface providing:
- Run cookbooks from browser
- Generate reports
- Manage nodes and policies
- Monitor infrastructure

**Note**: Chef CLI tools may change without notice. Commands may produce slightly different outputs on your system.

### Benefits of Using Chef

Chef also has a GUI built on top of Chef Server, which can help us running the cookbook from a browser, prepare reports, etc.
Configuring a System with Chef Cookbooks

Let's illustrate how to configure a managed system with Chef.

**Note**: Chef CLI tools may change without notice. Commands may produce slightly different outputs on your system.

### Benefits of Using Chef

| Benefit | Description |

To get this demo started, the following preparatory steps have already been completed:

    Installed the Chef-workstation collection of packages onto the local workstation.
    A Hosted Chef account created at api.chef.io (or at manage.chef.io). This is a Chef server platform hosted by Chef.
    From the Manage Chef console an organization created and the Starter Kit downloaded to the local workstation. Upon extraction of the Starter Kit in the local user home directory, a chef-repo is generated to store all Chef cookbooks and ssl key for secure communication between the local Chef workstation and the Chef server.

On the Manage Chef console inspect the available tabs. The Administration tab should display the earlier created organization. The Nodes and Policy tabs should not display any resources just yet.

 

The Manage Chef console

 

From the local Chef workstation verify connectivity between the Chef workstation and the Chef server with the knife CLI tool, installed earlier part of the chef-workstation collection. The knife CLI tool will be used to interact securely with the Chef server.

 

$ knife ssl check

 

Let's download from the Chef supermarket an existing cookbook. The supermarket.chef.io is a community driven public repository of Chef cookbooks and tools.

 

$ knife supermarket download nginx

 

The downloaded cookbook is a tarball composed of definition and configuration files that will eventually be ingested by Chef clients on managed nodes. Let's extract the contents of the tarball into the cookbooks subdirectory of our local Chef repository.

 

$ tar -zxvf nginx-12.2.5.tar.gz -C cookbooks

 

Once extracted, navigate into the top directory of the downloaded cookbook and create a recipes subdirectory.

 

$ mkdir recipes

 

Compose a simple default recipe with instructions to be executed by the Chef clients. The Chef client will install the desired “nginx” package on the managed node and will start its service.

 

$ cat recipes/default.rb

 

Upload the cookbook to the Chef server. The contents of the cookbook top directory, including the newly created recipe are packaged and uploaded to the server.

 

$ knife cookbook upload nginx

 

The Policy tab of the Manage Chef console should display the uploaded cookbook.

 

The Policy tab of the Manage Chef console should display the uploaded cookbook

 

The cookbook contents can be explored to validate all desired configuration files have been uploaded, such as the newly created default recipe.

 

Explore cookbook contents

 

Now we are ready to provision the infrastructure that will be managed by Chef. Many methods are available to us to provision infrastructure, be it local or cloud. We are provisioning a Google Cloud GCE VM and in the process we upload a public key for the root user that will allow the Chef server to connect to log in as the node's root without a password. The GCE VM provisioned is of e2-medium type running Ubuntu 20.04 LTS, with a private and a public IP addresses.

 

Provision the infrastructure that will be managed by Chef

 

Quickly validate that login is possible via SSH with the private key of the root user and the public IP address of the GCE VM.

 

$ ssh -i ~/.ssh/id_rsa_root root@162.222.176.32

 

Upon successful login, exit, to regain control of the local workstation terminal. The GCE VM is ready for bootstrapping. The VM target of the bootstrapping will become a managed node. The supplied identity file is the private key for the root user who will be tasked to install and configure the Chef client package and prepare it for its first run. The run is an execution cycle of a supplied cookbook.

 

$ knife bootstrap 162.222.176.32 --connection-user root --sudo --ssh-identity-file

 

The Nodes tab of the Manage Chef console should display the new node and its details.

 

The Nodes tab of the Manage Chef console should display the new node and its details

 

We validate that the application defined by the cookbook is running on the managed node. Since we installed a webserver, we open a browser window on the local workstation and type the public IP address of the node VM. We should see the default welcome page of our webserver application.

 

The default welcome page of our webserver application

 

We exemplified the installation of a webserver application on a managed Chef node, however, any changes to the cookbook stored on the Chef workstation, such as reconfigurations of the webserver application, service status changes, or packages uninstallation, can be triggered by invoking the Chef client on the managed nodes.
Key benefits of using Chef are:

**Note**: Chef CLI tools may change without notice. Commands may produce slightly different outputs on your system.

### Benefits of Using Chef

| Benefit | Description |
|---------|-------------|
| **Open Source** | Free systems integration framework |
| **Automation** | Automated configuration and deployment |
| **Scalability** | Handles large infrastructures |
| **High Availability** | Built-in HA support |
| **Consistency** | Consistent deployments across environments |
| **Cross-Platform** | Available for all major operating systems |
| **Access Control** | Role-based access control |
| **Visibility** | Real-time visibility, audits, compliance (Chef Automate) |

---

## Salt

### Overview

**Type**: Open source configuration management system built on remote execution framework

**Developer**: VMware, Inc.

**Architecture**: Client/server model or agentless model

**Communication**: 
- Agent-based: High-speed data bus (ZeroMQ)
- Agentless: SSH (limited functionality)

**Enterprise Product**: VMware Aria Automation (Salt-based)

### Architecture Components

**Salt Minions** (Clients):
- Receive configuration commands from Master
- Execute commands
- Report results back to Master

**Supported Platforms**:
- Unix-based systems
- Windows
- macOS

**Salt Masters** (Servers):
- Central management server
- Push commands to minions in parallel
- Collect status reports
- Multi-master support available

**Communication Method**:
- Default: ZeroMQ high-speed data bus (requires agent)
- Alternative: SSH for agentless setup (secure and encrypted)

### Core Concepts

**Execution Modules**:
- Provide basic functionality
- Package installation
- File management
- Container management
- Custom modules supported

**Returner Modules**:
- Save minion responses on master or other locations
- Default returners available
- Custom returners can be written

**Grains**:
- Information collected from minions
- Stored on the master
- Used for targeting specific minions
- Examples: OS type, hostname, IP address

**Pillar Data**:
- Private information specific to each minion
- Cryptographic keys
- Sensitive configuration data
- Shared between master and individual minion only

### Targeting and State Management

**Targeting Minions**: Combine Grains and Pillar Data to target specific systems

Example: Query all nodes running Fedora 23:

```bash
salt -G 'os:Fedora' and -G 'osrelease:23' test.ping
```

**State Management**: 
- Define desired state for minions
- State Modules manage configuration
- Ensures consistent system configuration

### Practical Salt Examples

**Example 1: Basic Web Server State File**

File: `/srv/salt/webserver.sls`

```yaml
# Install nginx
nginx:
  pkg.installed: []
  service.running:
    - enable: True
    - require:
      - pkg: nginx

# Create web directory
/var/www/html:
  file.directory:
    - user: www-data
    - group: www-data
    - mode: 755
    - makedirs: True

# Deploy index.html
/var/www/html/index.html:
  file.managed:
    - source: salt://files/index.html
    - user: www-data
    - group: www-data
    - mode: 644
    - require:
      - file: /var/www/html
```

**Apply the state**:
```bash
salt 'webserver*' state.apply webserver
```

**Example 2: User Management with Salt States**

File: `/srv/salt/users.sls`

```yaml
# Create multiple users
{% for user, uid in [('alice', 3001), ('bob', 3002), ('charlie', 3003)] %}
{{ user }}:
  user.present:
    - uid: {{ uid }}
    - gid: {{ uid }}
    - home: /home/{{ user }}
    - shell: /bin/bash
    - createhome: True
  
  file.directory:
    - name: /home/{{ user }}/.ssh
    - user: {{ user }}
    - group: {{ user }}
    - mode: 700
    - makedirs: True
    - require:
      - user: {{ user }}
{% endfor %}
```

**Example 3: Pillar Data for Environment-Specific Configuration**

File: `/srv/pillar/webserver.sls`

```yaml
webserver:
  port: 80
  ssl_port: 443
  server_name: production.example.com
  max_workers: 4
  
database:
  host: db.example.com
  port: 3306
  name: production_db
  user: app_user
```

File: `/srv/salt/nginx_config.sls`

```yaml
# Use pillar data in configuration
/etc/nginx/sites-available/default:
  file.managed:
    - source: salt://templates/nginx_site.conf
    - template: jinja
    - context:
        port: {{ pillar['webserver']['port'] }}
        server_name: {{ pillar['webserver']['server_name'] }}
        workers: {{ pillar['webserver']['max_workers'] }}
    - require:
      - pkg: nginx
  
  cmd.run:
    - name: nginx -t && systemctl reload nginx
    - onchanges:
      - file: /etc/nginx/sites-available/default
```

**Example 4: Orchestration - Multi-Step Deployment**

File: `/srv/salt/orchestrate/deploy.sls`

```yaml
# Stage 1: Update database servers
update_database:
  salt.state:
    - tgt: 'role:database'
    - tgt_type: grain
    - sls:
      - database.update

# Stage 2: Update application servers (after database)
update_application:
  salt.state:
    - tgt: 'role:application'
    - tgt_type: grain
    - sls:
      - application.deploy
    - require:
      - salt: update_database

# Stage 3: Update load balancers (last)
update_loadbalancer:
  salt.state:
    - tgt: 'role:loadbalancer'
    - tgt_type: grain
    - sls:
      - loadbalancer.config
    - require:
      - salt: update_application

# Stage 4: Verify deployment
verify_deployment:
  salt.function:
    - name: cmd.run
    - tgt: 'role:application'
    - tgt_type: grain
    - arg:
      - curl -f http://localhost:8080/health || exit 1
    - require:
      - salt: update_application
```

**Run orchestration**:
```bash
salt-run state.orchestrate orchestrate.deploy
```

**Example 5: Using Salt Grains for Targeting**

```bash
# Target all Ubuntu systems
salt -G 'os:Ubuntu' test.ping

# Target systems with specific memory
salt -G 'mem_total:>8000' cmd.run 'free -h'

# Target by custom grain
salt -G 'role:webserver' state.apply nginx

# Compound matching
salt -C 'G@os:Ubuntu and G@role:webserver' state.apply security
```

**Example 6: Event-Driven Automation with Reactor**

File: `/etc/salt/master.d/reactor.conf`

```yaml
reactor:
  # Auto-scale when high CPU detected
  - 'salt/beacon/*/cpu/high':
    - /srv/reactor/scale_up.sls
  
  # Auto-remediate failed services
  - 'salt/beacon/*/service/nginx/stopped':
    - /srv/reactor/restart_nginx.sls
```

File: `/srv/reactor/restart_nginx.sls`

```yaml
restart_nginx:
  local.state.single:
    - tgt: {{ data['id'] }}
    - arg:
      - service.running
      - nginx
      - enable: True
```

### Benefits of Using Salt

| Benefit | Description |
|---------|-------------|
| **Open Source** | Free configuration management system |
| **Automation** | Automated configuration and deployment |
| **High Availability** | Built-in HA support |
| **Event-Driven** | Event-driven infrastructure automation |
| **Flexible** | Agent-based or agentless deployments |
| **Fast** | High-speed communication via ZeroMQ |
| **Access Control** | Role-based access control |
| **Cross-Platform** | Available for all major operating systems |

---

## Tool Comparison

This table compares the four configuration management tools covered in this chapter:

| Feature | Ansible | Puppet | Chef | Salt |
|---------|---------|--------|------|------|
| **Type** | Agentless | Agent-based | Agent-based or Agentless | Agent-based or Agentless |
| **Language** | YAML (Playbooks) | Puppet DSL | Ruby DSL | YAML/Python |
| **Architecture** | SSH-based | Master/Agent | Server/Client/Workstation | Master/Minion |
| **Learning Curve** | Low | Medium | Medium-High | Medium |
| **Communication** | SSH | HTTPS | HTTPS | ZeroMQ or SSH |
| **Configuration Unit** | Playbook | Manifest | Cookbook/Recipe | State File |
| **Enterprise Version** | Automation Platform | Puppet Enterprise | Chef Automate | VMware Aria Automation |
| **Community Hub** | Ansible Galaxy | Puppet Forge | Chef Supermarket | SaltStack Formulas |
| **Best For** | Simple automation | Large enterprises | Complex workflows | Fast, parallel execution |

### Choosing the Right Tool

**Choose Ansible if**:
- You want simplicity and ease of use
- You prefer agentless architecture
- You have a small to medium infrastructure
- You need quick setup with minimal learning curve

**Choose Puppet if**:
- You have large-scale enterprise infrastructure
- You need mature, proven technology
- You want strong reporting capabilities
- You prefer declarative configuration

**Choose Chef if**:
- You need complex workflow automation
- Your team is comfortable with Ruby
- You want test-driven infrastructure
- You need detailed control over configuration

**Choose Salt if**:
- You need fast, parallel execution
- You want event-driven automation
- You need both agent-based and agentless options
- You require high-speed communication between nodes

---

## Glossary

**Agent**: Software installed on managed nodes that communicates with a central server to receive and execute configuration commands.

**Agentless**: Configuration management approach that doesn't require installing software on managed nodes, typically using SSH for communication.

**Ansible Galaxy**: Community hub for sharing and downloading Ansible roles, playbooks, and collections.

**Ansible Vault**: Encryption feature in Ansible for protecting sensitive data like passwords and API keys.

**Attributes** (Chef): Variables that define the state and characteristics of a node in Chef.

**Become**: Ansible directive for privilege escalation, allowing tasks to run with elevated permissions (sudo).

**Bootstrap**: Process of preparing a new node for management by installing necessary agents and initial configuration.

**Catalog File** (Puppet): Compiled, platform-specific instructions sent from Puppet Server to Puppet Agent describing desired system state.

**Chef Client**: Agent installed on managed nodes that executes cookbooks and reports back to Chef Server.

**Chef Server**: Central management server in Chef architecture that stores cookbooks and node information.

**Chef Workstation**: Development environment where Chef cookbooks and recipes are created and tested.

**Configuration Management**: Practice of systematically handling changes to infrastructure and applications to maintain desired state and consistency.

**Cookbook** (Chef): Package containing recipes, templates, files, and attributes that define a specific configuration scenario.

**Declarative**: Configuration approach where you specify the desired end state, and the tool determines how to achieve it.

**Execution Modules** (Salt): Modules providing basic functionality in Salt for tasks like package installation and file management.

**Facts** (Puppet): System information collected by Puppet Agent about the managed node (OS, memory, network, etc.).

**Grains** (Salt): Static information about Salt minions collected at startup (OS type, CPU, memory, etc.).

**Handler** (Ansible): Special task triggered by the `notify` directive, typically used to restart services after configuration changes.

**Idempotency**: Property ensuring that running the same operation multiple times produces the same result as running it once.

**Infrastructure as Code (IaC)**: Practice of managing infrastructure through machine-readable definition files rather than manual processes.

**Inventory** (Ansible): File or script listing managed hosts and organizing them into groups.

**Jinja2**: Templating engine used by Ansible and Salt for creating dynamic configuration files.

**Knife**: Command-line tool in Chef for interacting with Chef Server and managing infrastructure.

**Manifest** (Puppet): File written in Puppet DSL that defines the desired state of system resources.

**Master** (Salt): Central Salt server that sends commands to and receives results from Salt minions.

**Minion** (Salt): Managed node running Salt agent that receives commands from Salt Master.

**Module**: Reusable code component providing specific functionality (different meaning in each tool).

**Orchestration**: Coordination of multiple automated tasks across different systems in a specific sequence.

**Pillar Data** (Salt): Sensitive or node-specific data in Salt that is only available to targeted minions.

**Playbook** (Ansible): YAML file containing a series of plays that define automation tasks.

**Puppet Agent**: Client software installed on managed nodes that applies catalogs received from Puppet Server.

**Puppet Forge**: Public repository for sharing and downloading Puppet modules.

**Puppet Server**: Central server that compiles manifests into catalogs and distributes them to Puppet Agents.

**PuppetDB**: Database component storing configuration data, facts, and reports from Puppet infrastructure.

**Recipe** (Chef): Fundamental configuration unit in Chef, written in Ruby DSL, containing resource definitions.

**Resource** (Puppet/Chef): Fundamental unit representing a piece of the system (file, package, service, user, etc.).

**Returner** (Salt): Module that saves minion command results to various locations or formats.

**Role** (Ansible/Chef): Collection of related tasks or recipes grouped together for organizing functionality.

**Salt Master**: See Master (Salt).

**Salt Minion**: See Minion (Salt).

**SSH (Secure Shell)**: Network protocol for secure remote login and command execution, used by agentless configuration management tools.

**State** (Salt): Desired configuration described in SLS files, similar to Ansible playbooks or Puppet manifests.

**State File** (Salt): YAML file (`.sls`) defining desired system state using Salt state modules.

**Task** (Ansible): Single unit of action in a playbook, such as installing a package or copying a file.

**Template**: File with embedded variables and logic that generates final configuration files dynamically.

**YAML (Yet Another Markup Language)**: Human-readable data serialization format used by Ansible, Salt, and modern Chef for configuration files.

**ZeroMQ**: High-performance messaging library used by Salt for fast communication between master and minions.

---

## Summary

This chapter covered four major configuration management tools used in modern DevOps:

**Key Takeaways**:

1. **Infrastructure as Code** enables consistent, repeatable infrastructure provisioning across environments

2. **Ansible** offers simplicity with agentless architecture, making it ideal for quick adoption and simple automation tasks

3. **Puppet** provides enterprise-grade scalability with strong reporting, best suited for large organizations with complex infrastructure

4. **Chef** delivers powerful automation through Ruby DSL and test-driven infrastructure, ideal for complex workflows

5. **Salt** excels at high-speed parallel execution and event-driven automation, perfect for large-scale dynamic environments

**Decision Factors**:
- **Team expertise**: Consider existing skills (YAML, Ruby, Python)
- **Infrastructure size**: Small teams may prefer Ansible; enterprises may need Puppet or Chef
- **Speed requirements**: Salt offers fastest execution for large infrastructures
- **Learning curve**: Ansible has lowest barrier to entry
- **Agent requirements**: Agentless (Ansible) vs agent-based (Puppet, Chef, Salt)

**Security Considerations**:
- Always encrypt sensitive data (Ansible Vault, Chef Data Bags, Salt Pillar)
- Use SSH keys, never passwords in production
- Implement least-privilege access
- Regular security audits of configuration code
- Version control all configuration files

**Best Practices**:
- Start small and iterate
- Test configurations in non-production first
- Use version control (Git) for all configuration code
- Document your infrastructure as code
- Implement CI/CD for configuration management
- Regular backups of configuration servers
- Monitor and log all configuration changes

All four tools are production-ready and widely adopted. The choice depends on your specific requirements, team capabilities, and infrastructure characteristics.

