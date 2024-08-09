# Using Ansible to Setup GitLab using Podman Systemd Rootless Containers 

1. [Install and Setup](https://github.com/HerbBoy/ansible-podman-gitlab-server-role#install-and-setup)
2. [Using the Role](https://github.com/HerbBoy/ansible-podman-gitlab-server-role#using-the-role)
3. [Tags Explained](https://github.com/HerbBoy/ansible-podman-gitlab-server-role#tags-explained)
4. [Potential Bugs](https://github.com/HerbBoy/ansible-podman-gitlab-server-role#potential-bugs)

## Prepare Enviornment
```bash
git clone https://github.com/tosin2013/ansible-podman-gitlab-server-role.git /opt/ansible-podman-gitlab-server-role
mkdir -p ~/.ansible/roles/
sudo ln -s /opt/ansible-podman-gitlab-server-role/podman-gitlab-server-role /etc/ansible/roles
```


## Install and Setup
**Note**: This Role was built on RHEL 9.4, using Ansible 2.10.8 and Podman 2.2.1.

1. Install Ansible and Git
2. Clone the repository
3. Move podman-gitlab-server-role to /etc/ansible/roles/
    ```
    cp -r podman-gitlab-server-role /etc/ansible/roles/

    ```
4. Move /opt/ansible-podman-gitlab-server-role/playbooks/gitlab-mgmt.yml to $HOME 
    ```
    cp /opt/ansible-podman-gitlab-server-role/playbooks/gitlab-mgmt.yml $HOME 

    ```
5. Modify hosts inside of gitlab-mgmt.yml to match the host that will be running your Rootless GitLab Server Container
6. Modify the varialbes inside of ~/.ansible/roles/podman-gitlab-server-role/defaults/main.yml to your desired values.
    ```
    $ GILAB_SERVICE_ACCOUNT=gitlab
    $ DOMAIN=.example.com
    $ cat > /etc/ansible/roles/podman-gitlab-server-role/defaults/main.yml <<EOF
    ---
    # Username Variables
    gitlab_service_account: ${GILAB_SERVICE_ACCOUNT}

    # Container Image Variables
    gitlab_server_image_name: gitlab/gitlab-ce:latest

    # Container Name Variables
    gitlab_server_name: gitlab_server

    # GitLab Hostname
    gitlab_server_hostname: gitlab

    # Domain
    domain: '${DOMAIN}'

    # Podman Ports
    gitlab_container_ssl_port: '2443:2443/tcp'
    gitlab_gui_ssl_port: '2443'
    gitlab_container_ssh_port: '2222:22/tcp'

    # FirewallD Ports
    gitlab_firewall_ssl_port: '2443/tcp'
    gitlab_firewall_ssh_port: '2222/tcp'

    # GitLab Container Specific Variables
    gitlab_server_restart_policy: always
    EOF
    ```
7. There are three ansible collections you will need to install:
    ```
    ansible-galaxy collection install containers.podman
    ansible-galaxy collection install community.crypto
    ansible-galaxy collection install ansible.posix

    ```
8. Execute the Role

## Using the Role

- To execute the playbook for the first time use the following:
    ```
    ansible-playbook gitlab-mgmt.yml --tag initial_setup --ask-become --ask-pass

    ```
- Anytime after this use the respected tag that fits your use case.

## Tags Explained

- initial_setup
    - This tag should be used on the initial execution of the role. This tag will setup systemd for a rootless user, setup a self signed certificate, and deploy the GitLab Server Instance.
- create_gitlab
    - This tag can be leveraged at any point after the initial execution of the role. This tag will rebuild the container instance, setup systemd for management of the container and open the necessary ports to access the container.
- stop_gitlab
    - This tag will leverage systemd to stop the container instance gently. 
- restart_gitlab
    - This tag will leverage systemd to restart the container instance gently. 
- remove_gitlab
    - This tag will leverage systemd to stop the container instance gently. It will then use the podman module to remove the container instance (this is equal to "Podman rm <container_ID>). THIS TAG WILL NOT REMOVE VOLUMES (AKA Persistent Storage) 
- destroy_gitlab
    - This tag will leverage systemd to stop the container instance gently. It will then use the podman module to remove the container instance (this is equal to "Podman rm <container_ID>). THIS TAG WILL REMOVE VOLUMES (AKA Persistent Storage) It will be as if you never had GitLab Running.

## Potential Bugs
Please Report all bugs found by making an issue.