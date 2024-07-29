---
- name: Setup and Deploy NestJS Boilerplate
  hosts: hng
  become: yes
  vars:
    repo_url: "https://github.com/hngprojects/hng_boilerplate_nestjs"
    branch: "devops"
    app_dir: "/opt/stage_5b"
    pg_user: "admin"
    pg_password: "devopsnext"
    pg_database: "hngnext_db"
    pg_pw_file: "/var/secrets/pg_pw.txt"
    node_version: "16.x"
    rabbitmq_version: "3.8"
    nginx_conf: "/etc/nginx/sites-available/default"
    app_port: 3000
    proxy_port: 80
    log_dir: "/var/log/stage_5b"
    hng_user: "hng"

  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Create hng user with sudo privileges
      user:
        name: "{{ hng_user }}"
        shell: /bin/bash
        groups: sudo
        append: yes

    - name: Create log directory
      file:
        path: "{{ log_dir }}"
        state: directory
        owner: "{{ hng_user }}"
        group: "{{ hng_user }}"
        mode: '0755'

    - name: Clone the boilerplate repository
      git:
        repo: "{{ repo_url }}"
        dest: "{{ app_dir }}"
        version: "{{ branch }}"
        force: yes
      become_user: "{{ hng_user }}"

    - name: Install required packages
      apt:
        name:
          - git
          - curl
          - gnupg
          - ca-certificates
          - lsb-release
          - apt-transport-https
          - software-properties-common
        state: present

    - name: Install Node.js
      shell: |
        curl -fsSL https://deb.nodesource.com/setup_{{ node_version }} | bash -
        apt-get install -y nodejs

    - name: Install PM2 globally
      npm:
        name: pm2
        global: yes

    - name: Install project dependencies
      npm:
        path: "{{ app_dir }}"
        state: present
      become_user: "{{ hng_user }}"

    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present

    - name: Ensure PostgreSQL service is running
      service:
        name: postgresql
        state: started
        enabled: yes

    - name: Setup PostgreSQL database
      become_user: postgres
      postgresql_db:
        name: "{{ pg_database }}"

    - name: Setup PostgreSQL user
      become_user: postgres
      postgresql_user:
        name: "{{ pg_user }}"
        password: "{{ pg_password }}"
        priv: "ALL"

    - name: Save PostgreSQL credentials
      copy:
        content: "User: {{ pg_user }}\nPassword: {{ pg_password }}\nDatabase: {{ pg_database }}"
        dest: "{{ pg_pw_file }}"
        owner: root
        group: root
        mode: '0600'

    - name: Install RabbitMQ
      shell: |
        echo "deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang" | tee /etc/apt/sources.list.d/bintray.erlang.list
        curl -fsSL https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc | apt-key add -
        apt-get update
        apt-get install -y rabbitmq-server

    - name: Ensure RabbitMQ service is running
      service:
        name: rabbitmq-server
        state: started
        enabled: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Configure Nginx to reverse proxy the application
      template:
        src: nginx.conf.j2
        dest: "{{ nginx_conf }}"
        owner: root
        group: root
        mode: '0644'
      notify: restart nginx

    - name: Ensure Nginx service is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Start the application with PM2
      shell: |
        cd {{ app_dir }}
        pm2 start dist/main.js --name boilerplate -- -p {{ app_port }}

    - name: Configure PM2 to save stderr and stdout logs
      lineinfile:
        path: "{{ app_dir }}/ecosystem.config.js"
        line: |
          module.exports = {
            apps: [{
              name: "boilerplate",
              script: "./dist/main.js",
              instances: "max",
              exec_mode: "cluster",
              env: {
                NODE_ENV: "production",
                PORT: {{ app_port }},
              },
              error_file: "{{ log_dir }}/error.log",
              out_file: "{{ log_dir }}/out.log",
              log_file: "{{ log_dir }}/combined.log",
              time: true,
              merge_logs: true,
            }],
          };
      become_user: "{{ hng_user }}"

    - name: Change ownership of log files
      file:
        path: "{{ log_dir }}"
        state: directory
        owner: "{{ hng_user }}"
        group: "{{ hng_user }}"
        recurse: yes

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
