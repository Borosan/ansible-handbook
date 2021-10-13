# Ansible Vault

### What is Ansible Vault?

Ansible is a configuration management tool. While working with Ansible, we can create various playbooks, inventory files, variable files, etc. Some of the files contain sensitive and important data like usernames and passwords. Ansible provides a feature named Ansible Vault that prevents this data from being exposed. It keeps passwords and other sensitive data in an encrypted file rather than in plain text files. It provides password-based authentication.

Ansible Vault performs various operations. Specifically, it can

* Create an encrypted file
* Decrypt a file
* Encrypt a file
* View an encrypted file without breaking the encryption
* Edit an encrypted file
* Generate or reset the encrypted key

### Create an encrypted file

 Use`ansible-vault create` command to create the encrypted file.

```
[user1@controller demo-vault]$ ansible-vault create secret-playbook.yaml
New Vault password:
Confirm New Vault password:
```

After typing this command, it will ask for a password, this password is for vault password and will be used later.

```
---
#sample playbook to test vault secret-playbook.yaml

- hosts: localhost
  become: yes
  vars:
   - ansible_sudo_pass: Srv@123?

  tasks:
   - name: install httpd
     yum: name=httpd state=latest
```

> make user1 sudoer as we are working on localhost `usermod –aG wheel UserName`

Lets check that the file has been encrypted, using `cat` command:

```
[user1@controller demo-vault]$ cat secret-playbook.yaml
$ANSIBLE_VAULT;1.1;AES256
36653530663931656132366133626365386535333264636262343036666333613439623866643138
3436393663653034306466356162623235326363616266380a623963316131613933633161353064
66333662636136356464313038326538336132666133653761393265323166393637653236393333
3831366364393335340a613739366539353434333738363937306137323966323935626366663634
62646433353337313831356138626531393562656264643734353531623537346464353230366566
36613839643366626364626635333866346134623431633365363164303131383830353033363332
64613066646665343264343361643434623131666232353733396434663232393763623932656339
36623461376561333836313761623035396564643630653461383531333762326464656162643031
65393238613035393839323632353633613964396230623332643261643761636239323732356463
62663735636663353864333439356666366261373839353630633839333366373231616163626638
61313565613261383037386461313863326332646163653831636133373064333863613331613561
65623431393864396534646461363135643532656637663838623337373631373336303533316530
62633237646339363962336564323332356431373632393433343266623233636466313562323463
3432316237646364636263303232623331633836363330666263
```

### Decrypting a file

 The `ansible-vault decrypt` command is used to decrypt the encrypted file

```
[user1@controller demo-vault]$ ansible-vault decrypt secret-playbook.yaml
Vault password:
Decryption successful
```

```
[user1@controller demo-vault]$ cat secret-playbook.yaml
---
#sample playbook to test vault secret-playbook.yaml

- hosts: localhost
  become: yes
  vars:
   - ansible_sudo_pass: Srv@123?

  tasks:
   - name: install httpd
     yum: name=httpd state=latest
```

### Encrypting a file

If you want to encrypt an already existing file which is unencrypted use `ansible-vault encrypt` command:

```
[user1@controller demo-vault]$ ansible-vault encrypt secret-playbook.yaml
New Vault password:
Confirm New Vault password:
Encryption successful
```

_check the result using `cat secret-playbook.yaml` command._

### Decrypt a running playbook

```
[user1@controller demo-vault]$ ansible-playbook secret-playbook.yaml
ERROR! Attempting to decrypt but no vault secrets found
```

 Prior to Ansible 2.4, decrypting files during run time required the use of the** `--ask-vault-pass`**` `parameter as shown with either **`ansible`** or **`ansible-playbook`** commands:

```
[user1@controller demo-vault]$ vim /etc/resolv.conf
[user1@controller demo-vault]$ ansible-playbook secret-playbook.yaml --ask-vault-pass
Vault password:

PLAY [localhost] ************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [install httpd] ********************************************************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

However, that has been deprecated. Since Ansible 2.4 the standard method of prompting for a password is to utilize the **`--vault-id`** option as shown

```
[user1@controller demo-vault]$ ansible-playbook secret-playbook.yaml --vault-id @prompt
Vault password (default):

PLAY [localhost] ************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [install httpd] ********************************************************************************************************************
ok: [localhost]

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

 _The **`@prompt`** will prompt for the password**.**_

A simple trick to avoid being prompted for a password every time you are decrypting files during runtime is to store the vault password in a file.

```
[user1@controller demo-vault]$ cat password.txt
P@ssw0rd
```

Prior to Ansible 2.4 the way to achieve this was the use of the **–vault-password-file** parameter to specify the path to the file that contains the stored password.

```
[user1@controller demo-vault]$ ansible-playbook secret-playbook.yaml --vault-password-file password.txt

PLAY [localhost] ************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [install httpd] ********************************************************************************************************************
ok: [localhost]

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

 However, just like the **`--ask-vault-pass`** option, the option **`--vault-password-file`** has been deprecated to pave the way for the` `**`--vault-id`** option.

```
[user1@controller demo-vault]$ ansible-playbook secret-playbook.yaml --vault-id password.txt

PLAY [localhost] ************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [install httpd] ********************************************************************************************************************
ok: [localhost]

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

{% hint style="danger" %}
It is not recommended to store passwords in plain text because if anybody gets a hold of the playbook file, your security can be compromised. You are therefore presented with 2 options:

* to encrypt the entire file 
* or encrypt the value of the variable.
{% endhint %}

### **Encrypt the value of the variable**

 Apart from encrypting an entire playbook, **ansible-vault** also gives you the ability to encrypt variables only. In most cases these are variables bearing highly confidential & sensitive information such as passwords and API keys.

 To encrypt a variable, use the **`encrypt_string`** option **:**

```
[user1@controller demo-vault]$ ansible-vault encrypt_string 'Srv@123?' --name 'ansible_sudo_pass'
New Vault password:
Confirm New Vault password:
ansible_sudo_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666633333939326632336237663262383465663232666633373263393939623439386663643336
          3638373263343738613466633765383836353739383733650a643661623131633861396262613861
          33663035363334653866623165333337623566376165663739313332333033313939333739343730
          6135653863653962350a633161396262353462353636663566656433663632313264373037653562
          3233
Encryption successful
```

 The output above indicates that the password has been encrypted with **AES 256 encryption**. From here, copy the entire encrypted code from !vault | . Head out to the playbook file and delete the plaintext password value and paste the encrypted value as shown.

```
---
#sample playbook to test vault unenc-playbook.yaml

- hosts: localhost
  become: yes

  vars:
    - ansible_sudo_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666633333939326632336237663262383465663232666633373263393939623439386663643336
          3638373263343738613466633765383836353739383733650a643661623131633861396262613861
          33663035363334653866623165333337623566376165663739313332333033313939333739343730
          6135653863653962350a633161396262353462353636663566656433663632313264373037653562
          3233
  tasks:
   - name: install httpd
     yum: name=httpd state=latest

   - name: print a secure variable
     debug:
       var: ansible_sudo_pass
```

 Save and exit the file. Now run the playbook and verify whether it will still display the value of the password stored in the **my_secret** variable.

```
[user1@controller demo-vault]$ ansible-playbook unenc-playbook.yaml --ask-vault-pass
Vault password:

PLAY [localhost] ************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [install httpd] ********************************************************************************************************************
ok: [localhost]

TASK [print a secure variable] **********************************************************************************************************
ok: [localhost] => {
    "ansible_sudo_pass": "Srv@123?"
}

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

_The output above shows that we succeeded in encrypting the variable._

### **View an Encrypted File**

To have a peek at an encrypted file, use** **`ansible-vault view`** **command:

```
[user1@controller demo-vault]$ ansible-vault view secret-playbook.yaml
Vault password:
---
#sample playbook to test vault secret-playbook.yaml

- hosts: localhost
  become: yes
  vars:
   - ansible_sudo_pass: Srv@123?

  tasks:
   - name: install httpd
     yum: name=httpd state=latest
```

### Editing the encrypted file

 If the file is encrypted and changes are required, use the `edit` command.

```
[user1@controller demo-vault]$ ansible-vault edit secret-playbook.yaml
Vault password:
```

### **Reset Ansible vault Password**

 Also, we can reset or change the Vault’s password. This is done using the **`rekey` **option:

```
[user1@controller demo-vault]$ ansible-vault rekey secret-playbook.yaml
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful
```

and it is done.



.

.

.

[https://www.redhat.com/sysadmin/introduction-ansible-vault](https://www.redhat.com/sysadmin/introduction-ansible-vault)

[https://www.linuxtechi.com/use-ansible-vault-secure-sensitive-data/](https://www.linuxtechi.com/use-ansible-vault-secure-sensitive-data/)

.
