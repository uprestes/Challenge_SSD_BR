# PostgreSQL Day 2 Operations

This project contains an Ansible playbook for managing Day 2 operations on PostgreSQL databases.

## Project Structure

```
project_challenge_SSD/
├── playbook.yml       # Main playbook
├── README.md          # Documentation
├── .vault_pass        # Vault password file (example/placeholder)
└── vars/
    └── secrets.yml    # Encrypted credentials
```

## Prerequisites

*   Ansible 2.9+
*   PostgreSQL 12+
*   Collection `community.postgresql`

## Configuration

1.  **Security**: The `vars/secrets.yml` file must contain the credentials for the `readonly` user. It is recommended to encrypt it using Ansible Vault:
    ```bash
    ansible-vault encrypt vars/secrets.yml
    ```

## Usage

To run the standard checks and user creation:

```bash
ansible-playbook playbook.yml --ask-vault-password
```

### Restore Backup (WARNING)

To restore the database from a backup (destructive operation):

```bash
ansible-playbook playbook.yml --ask-vault-password -e "restore_backup=true"
```
