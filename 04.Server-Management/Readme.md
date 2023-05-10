![](.Readme_images/48533959.png)

# membuat ssh config
bila belum membuat ssh key, buat menggunakan perintah

```shell
ssh-keygen
```
![](.Readme_images/cd07309c.png)

buat file .ssh/config
```shell
Host *
    User reiya24
    IdentityFile /home/reiya24/.ssh/id_rsa
    Port 22

Host appserver
  HostName 103.67.186.93

Host cicd
  HostName 103.67.186.92

Host gateway
  HostName 103.67.186.89

Host monitoring
  HostName 103.67.186.91

```

![](.Readme_images/64b3c2af.png)

# Password Login Disable via ansible

buat ansible playbook

```yaml
---
- hosts: all
  become: true
  become_user: "{{ansible_user}}"
  gather_facts: true
  tasks:
    - name: kirim folder .ssh
      copy:
        src: /home/{{ansible_user}}/.ssh/
        dest: /home/{{ansible_user}}/.ssh/
        owner: "{{ansible_user}}"
        mode: "0600"

    - name: "matikan SSH login menggunakan password"
      lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "^PasswordAuthentication"
        line: "PasswordAuthentication no"
        state: present

    - name: "restart service sshd"
      service:
        name: sshd
        state: restarted
```
![](.Readme_images/e18b8da9.png)

jalankan ansible playbook
```shell
ansible-playbook nama_file.yaml
```
![](.Readme_images/4d3d0e7a.png)

mematikakn password login ssh berhasil 
![](.Readme_images/1d4297b8.png)

# menambahkan firewall

nyalakan ufw firewall
```yaml
sudo ufw enabled
```

![](.Readme_images/6ddc8704.png)

buka port yang hanya diperlukan 

![](.Readme_images/716b9b7e.png)
