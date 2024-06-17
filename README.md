# Ansible-Kubernetes_Cluster-Open5gs

![Screenshot from 2024-06-14 15-44-00](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/20c1835d-3e39-4dcb-a7a8-352693f04d9a)

# Ansible 

Ansible is an open-source automation tool used for IT tasks such as configuration management, application deployment, and task automation. It is simple, agentless, and powerful, making it a popular choice for managing infrastructure and deploying applications.

# Key Features of Ansible:

Agentless: Ansible uses SSH for communication, eliminating the need for installing and managing agents on client machines.

Human-readable YAML: Ansible playbooks are written in YAML, making them easy to read and write.

Idempotency: Ansible ensures tasks are repeatable and only makes changes when necessary, avoiding redundancy.

Modular: Ansible comes with a vast collection of built-in modules, and you can also create custom ones.

# How Does Ansible Work?

Ansible works by connecting to your nodes (clients) and pushing small programs called "modules" to them. It then executes these modules over SSH, and the modules communicate with Ansible to report back on the state of the nodes. The architecture is simple, with no need for a centralized server or database.

# Ansible Playbooks

Ansible playbooks are files where you define your automation tasks. They are written in YAML and consist of a series of plays. Each play targets a group of hosts and defines tasks to be executed on those hosts. Playbooks are the cornerstone of Ansible's automation, allowing you to manage configurations, deployments, and orchestrations in a clear and structured manner.


# Basic Structure of an Ansible Playbook

```
---
- name: Ensure Apache is installed
  hosts: webservers
  become: true
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
```

In this example:
- name: Describes what the playbook does.
- hosts: Specifies the group of hosts the playbook will run on.
- become: Elevates privileges to run the tasks as a superuser.
- tasks: Lists the actions to perform.

# Deploying a Kubernetes Cluster Using Ansible Playbook

Deploying a Kubernetes cluster can be automated using Ansible playbooks. Here’s a step-by-step guide to deploying a single-node Kubernetes cluster using MicroK8s.

# Step 1: Set Up Your Environment

Ensure you have:
- Ansible installed on your control machine.
```
sudo apt-get install ansible
```
- SSH access to your target machine.
```
sudo apt-get install sshpass
```

# Step 2: Create the Ansible Inventory

Create an inventory file (e.g., `inventory.ini`) that lists your target machines.

```
[microk8s_nodes]
Microk8s_node ansible_host=192.168.1.240 ansible_user=<username> ansible_password=<password> ansible_sudo_pass=<password>
```

![ansible-helm](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/17ab4b2c-d253-4219-9ad5-8484c88fb8b4)


# Step 3: Write the Ansible Playbook

Create a playbook (e.g., `install_k8s.yml`) that installs MicroK8s and Helm, and deploys Open5GS.


```
---
- name: Install MicroK8s on Ubuntu
  hosts: all
  become: yes
  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

  tasks:
    - name: Ensure apt cache is up to date
      apt:
        update_cache: yes

    - name: Install snapd
      apt:
        name: snapd
        state: present

    - name: Install MicroK8s
      snap:
        name: microk8s
        classic: true
        state: present

    - name: Add user to microk8s group
      user:
        name: "<username>"
        groups: microk8s
        append: yes

    - name: Change ownership of the .kube directory
      file:
        path: /home/<username>/.kube
        state: directory
        owner: "<username>"
        group: "<username>"

    - name: Enable MicroK8s addons
      shell: |
        microk8s enable dns dashboard storage ingress
      args:
        executable: /bin/bash

    - name: Ensure MicroK8s is running
      shell: |
        microk8s status --wait-ready
      args:
        executable: /bin/bash

    - name: Configure kubectl alias
      shell: |
        snap alias microk8s.kubectl kubectl
      args:
        executable: /bin/bash

    - name: Ensure .kube config exists
      file:
        path: /home/<username>/.kube/config
        state: touch
        owner: "<username>"
        group: "<username>"

    - name: Set up kubeconfig for the user
      shell: |
        microk8s config > /home/<username>/.kube/config
      args:
        executable: /bin/bash
      become_user: "<username>"

    - name: Add the Open5GS Helm repository
      command: microk8s helm repo add adaptivenetlab https://adaptivenetlab.github.io/charts/

    - name: Update Helm repositories
      command: microk8s helm repo update

    - name: Open5gs Namespace
      command: microk8s kubectl create ns open5gs

    - name: Install Open5GS using Helm
      command: microk8s helm install my-open5gs adaptivenetlab/open5gs --version 1.0.3 -n open5gs

    - name: Check Open5gs NFV deployment
      command: microk8s kubectl get pod -n open5gs

```

# To run Ansible-Playbook

```
ansible-playbook -i inventory.ini install_microk8s.yml
```

![Screenshot from 2024-06-17 15-34-56](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/b8fbd85e-8744-4f96-9808-084b67e81a77)


![Screenshot from 2024-06-14 16-04-03](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/b5518163-30c3-4f44-8c5d-e7a1d33baf54)


# Explanation of the Playbook

1. Update and Upgrade Apt Packages: Ensures the latest updates and upgrades are applied.
2. Install snapd: Installs `snapd`, a package manager for snaps.
3. Install MicroK8s: Install MicroK8s with the `classic` confinement.
4. Add User to MicroK8s Group: Adds the user to the `microk8s` group to avoid needing `sudo` for MicroK8s commands.
5. Create Alias for Kubectl: Creates an alias for `microk8s.kubectl` as `kubectl`.
6. Ensure MicroK8s is Running: Wait for MicroK8s to be fully started and ready.
7. Enable DNS and Storage Addons: Enable essential addons in MicroK8s.
8. Download and Install Helm: Install Helm using the official installation script.
9. Add Open5GS Helm Repository: Adds the Open5GS Helm repository.
10. Update Helm Repositories: Updates the Helm repositories to ensure the latest charts are available.
11. Install Open5GS Using Helm: Installs Open5GS using Helm.
12. Check Whether Open5gs NFV deployed on the cluster.

![Screenshot from 2024-06-17 15-55-24](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/8fa9726c-b93b-4bdb-b1c7-f831e5ee3aba)

![Screenshot from 2024-06-17 15-51-53](https://github.com/sambathkumarj/Ansible-Kubernetes/assets/42794636/6c7294ca-90a9-48c2-886f-f32ef14039de)


Thus, Ansible makes automating complex tasks like deploying a Kubernetes cluster straightforward and efficient. By writing clear, reusable playbooks, you can manage your infrastructure consistently and reliably. This example demonstrates how you can deploy a Kubernetes cluster with MicroK8s and install Open5GS using Helm, showcasing the power and flexibility of Ansible.
