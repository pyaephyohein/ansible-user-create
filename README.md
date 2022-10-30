### Ansible for Server Security Setup

**[Inventory](./inventory)**
```bash
[staging]

hostname=<host name> ansible_ssh_host=<your server ip address> ansible_user=ubuntu  ansible_ssh_private_key_file=./host_ssh/<add your key name>

[staging:vars]
ansible_python_interpreter=/usr/bin/python3
```

**If you use nginx server, you need to remove # of nginx-ratelimit task in [site.yaml](./site.yaml)**
```yaml 
roles:
    # - { role: nginx-ratelimit, tags: 'nginx-ratelimit'} # if you use nginx server remove "#" 
    - { role: fix-security, tags: 'fix-security' }
```
**Create User Password**
```shell
python3 createpass.py
```
type password and copy hash

paste to user.password (./group_var/all.yaml)

```yaml
user:
  name: 'admin'
  password: 'password-hash-here'
  ```
**Add Allow IP address to access ssh**

Add SSH allow ip to ssh.allow_ip [group_vars/all.yaml](./group_vars/all.yaml) (you can add ip/network range example 192.168.1.2 or 192.168.1.1/24) 
```yaml 
ssh:
  allow_ip: 
    - 192.168.1.2
    - 192.168.1.1/24
```
save your host's accessable private key at **[host_ssh](./host_ssh)**

add your public key to **[host_ssh](./host_ssh)**

```bash 
ssh-keygen -y -f {yourkey}.pem > {yourkey}.pub

```
after key generate 

you need to add your public key name in follow task of [fix-security/task/all.yaml](./roles/fix-security/tasks/main.yaml),

```yaml
- name: Add SSH public key for "{{ user.name }}"
  become: yes
  become_user: root
  authorized_key:
    user: "{{ user.name }}"
    key: "{{item }}"
    state: present
  loop:
    - "{{ lookup('file', './host_ssh/<your public key name>.pub') }}"
```

After finished "restart sshd" TASK, task will fail, Don't worry it's normal because we block normal connection 😄

1. Make sure you update [inventory](./inventory) file before running the playbook.
2. Review variables in [group_vars/all.yaml](./group_vars/all.yaml),
3. Install `jmespath` package on the Ansible control machine
   ```shell
   pip install jmespath
   ```

### Useful commands

**Playbook syntax check**
```bash
ansible-playbook -v --syntax-check site.yaml
```

**Run playbook**

```bash
ansible-playbook -v site.yaml
```
