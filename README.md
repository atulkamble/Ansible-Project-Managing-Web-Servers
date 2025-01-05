# Ansible Project: Managing Web Servers

## **Project Overview**
This document describes a complete Ansible project, including code, steps to run the project, expected outputs, client requirements, and the proposed solution.

---

## **Client Requirements**
1. Automate the configuration of multiple web servers (e.g., Nginx).
2. Ensure consistent deployment of a custom configuration file.
3. Create a centralized `index.html` in the document root for all servers.
4. Use a single command to configure and deploy the solution.
5. Provide logs and reports of the deployment status.

---

## **Proposed Solution**
We will use Ansible to automate the configuration and deployment of Nginx web servers. The project will:

1. Set up a directory structure for an organized Ansible project.
2. Use Ansible roles for modular management.
3. Include playbooks to manage `common` tasks and specific configurations for `webservers`.
4. Provide an inventory file to define server groups.

---

## **Directory Structure**
```plaintext
ansible_project/
├── ansible.cfg
├── inventory/
│   ├── hosts
│   └── group_vars/
│       ├── all.yml
│       └── webservers.yml
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── motd.j2
│   │   └── defaults/
│   │       └── main.yml
│   └── webserver/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── templates/
│       │   ├── nginx.conf.j2
│       │   └── index.html.j2
│       └── defaults/
│           └── main.yml
├── playbooks/
│   ├── site.yml
│   ├── common.yml
│   └── webserver.yml
└── README.md
```

---

## **Code Files**

### **1. `ansible.cfg`**
```ini
[defaults]
inventory = inventory/hosts
roles_path = roles
host_key_checking = False
retry_files_enabled = False
```

### **2. `inventory/hosts`**
```ini
[webservers]
web1.example.com
web2.example.com
```

### **3. `inventory/group_vars/all.yml`**
```yaml
---
ansible_user: ubuntu
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

### **4. `inventory/group_vars/webservers.yml`**
```yaml
---
nginx_port: 80
document_root: /var/www/html
welcome_message: "Welcome to {{ ansible_hostname }}!"
```

### **5. `roles/common/tasks/main.yml`**
```yaml
---
- name: Update apt cache
  apt:
    update_cache: yes

- name: Install common packages
  apt:
    name: ["curl", "git"]
    state: present

- name: Deploy custom MOTD
  template:
    src: motd.j2
    dest: /etc/motd
    mode: 0644
```

### **6. `roles/common/templates/motd.j2`**
```plaintext
{{ welcome_message }}
Managed by Ansible.
```

### **7. `roles/webserver/tasks/main.yml`**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
    mode: 0644
  notify: Restart Nginx

- name: Ensure document root exists
  file:
    path: "{{ document_root }}"
    state: directory
    owner: www-data
    group: www-data

- name: Deploy custom index.html
  template:
    src: index.html.j2
    dest: "{{ document_root }}/index.html"
    mode: 0644
```

### **8. `roles/webserver/templates/nginx.conf.j2`**
```nginx
server {
    listen {{ nginx_port }};
    server_name localhost;

    root {{ document_root }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### **9. `roles/webserver/templates/index.html.j2`**
```html
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>{{ welcome_message }}</h1>
    <p>This is the default page served by Nginx.</p>
</body>
</html>
```

### **10. `roles/webserver/handlers/main.yml`**
```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

### **11. `playbooks/site.yml`**
```yaml
---
- import_playbook: common.yml
- import_playbook: webserver.yml
```

### **12. `playbooks/common.yml`**
```yaml
---
- hosts: all
  roles:
    - common
```

### **13. `playbooks/webserver.yml`**
```yaml
---
- hosts: webservers
  roles:
    - webserver
```

---

## **Steps to Run**
1. **Set up Ansible environment**:
   Ensure Ansible is installed on your local machine.
   ```bash
   sudo apt update && sudo apt install ansible -y
   ```

2. **Clone or create the project directory**:
   Place all files and directories as per the structure above.

3. **Test the inventory**:
   Verify connectivity to the remote servers.
   ```bash
   ansible all -m ping
   ```

4. **Run the playbook**:
   Execute the main `site.yml` playbook.
   ```bash
   ansible-playbook playbooks/site.yml
   ```

5. **Check the deployment**:
   - SSH into one of the servers.
   - Verify Nginx is running.
   ```bash
   systemctl status nginx
   ```
   - Open a browser and visit the server's IP address to view the `index.html` page.

---

## **Expected Output**
1. **Console Output**:
   ```plaintext
   PLAY [all] ********************************************************************

   TASK [common : Update apt cache] **********************************************
   ok: [web1.example.com]
   ok: [web2.example.com]
   
   TASK [webserver : Install Nginx] **********************************************
   changed: [web1.example.com]
   changed: [web2.example.com]

   PLAY RECAP ********************************************************************
   web1.example.com        : ok=5    changed=3    unreachable=0    failed=0
   web2.example.com        : ok=5    changed=3    unreachable=0    failed=0
   ```

2. **Browser Output**:
   Visiting the server IP (`http://web1.example.com`) displays:
   ```html
   <html>
   <head>
       <title>Welcome</title>
   </head>
   <body>
       <h1>Welcome to web1.example.com!</h1>
       <p>This is the default page served by Nginx.</p>
   </body>
   </html>
   ```

---

This project provides a robust, automated solution for managing web servers using Ansible, meeting all client requirements efficiently.

