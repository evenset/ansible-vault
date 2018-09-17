# Guide to Using Ansible Vault and Encrypting Other Files
### Running the Example
1. Ensure ansible, ansible-playbook and ansible-vault are installed.
1. Run `ansible-playbook main.yml`
1. Verify the output in your terminal. The important thing to note is that all of the output is not encrypted. The output should look like
```
$ ansible-playbook main.yml

PLAY [Main File] ****************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [base : Print env vars] ****************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Env Var 'Some Env Var'"
}

TASK [base : Print password var] ************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Env Var 'p'"
}

TASK [base : Unencrypt file using openssl] **************************************************************************************************************************************
changed: [localhost]

TASK [base : Check Unencrypted file exists] *************************************************************************************************************************************
ok: [localhost]

TASK [base : Create read script (encrypted)] ************************************************************************************************************************************
changed: [localhost]

TASK [base : Display the read file] *********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Here is some text for that we want to protect"
}

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=7    changed=2    unreachable=0    failed=0
```

### Encrypting Ansible Vars (yml files)
If you need to encrypt values for ansible (stored in .yml files) this can be done be using Ansible Vault. It is important to note that all the files in one repository must have the same password.
1. Create a password file. It can be called anything, but I've used `.vault_pass`. Inside this file, enter what you want your password to be in plaintext.
1. Add the `.vault_pass` file to your ansible config. It should be `vault_password_file = ./.vault_pass`.
1. Add the `.vault_pass` file to the git ignore file. In this example, I've commited it so you can see what it looks like, but normally it should be be added.
1. Now we can encrypt any files we want. To encrypt an existing file run `ansible-vault encrypt file.yml`. To create a new encrypted file run `ansible-vault create newFile.yml`. Ansible shouldn't prompt you for a password if the `.vault_pass` file was setup correctly.
1. You can now run any deployments as needed. Ansible will automatically use the `.vault_pass` file to decrypt the files as needed.

### Encrypting Non-Ansible files
If you need to encrypt other, non-yml files that are not directly used by ansible, we cannot use ansible-vault. Instead what we will do is use openssl to encrypt the files locally with a password and then decrypt the files on the server later. We can securely store the encryption password in the ansible-vault so that we don't need to manually copy it onto the server.
1. Encrypt the file you want to use with `openssl enc -aes-256-cbc -salt -in original.txt -k YOUR_PASSWORD > encrypted.txt.enc`
1. Add `YOUR_PASSWORD` to the your var file in ansible. I've called mine `sslpass`.
1. Encrypt that var file using the steps above.
1. Use that variable to unencrypt the file in an ansible playbook by with the command. You can change the name of the output as needed by modifing the -out flag argument.
```
- name: Unencrypt file using openssl
  shell: openssl enc -aes-256-cbc -d -salt -in encrypted.txt.enc -out decrypted.txt -k {{ sslpass }}
```

### Viewing, Editting and Decrypting Files
You can view or modify an encrypted file with the following commands.  
You can find the documentation at https://docs.ansible.com/ansible/2.6/user_guide/vault.html#decrypting-encrypted-files
```
# View
ansible-vault view foo.yml bar.yml

# Edit (One at a time only)
ansible-vault edit foo.yml

# Decrypt
ansible-vault decrypt foo.yml bar.yml

# Change password (rekey)
ansible-vault rekey foo.yml bar.yml
```

