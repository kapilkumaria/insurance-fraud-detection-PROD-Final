# Setting Up HashiCorp Vault - Sequential Overview

## Part 1 – Installing HashiCorp Vault

1. Add PGP for the package signing key:
```
sudo apt update && sudo apt install gpg
```

2. Add the HashiCorp GPG key:
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

3. Verify the key's fingerprint:
```
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
```

4. Add the official HashiCorp Linux repository:
```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

5. Update and install Vault:
```
sudo apt update && sudo apt install vault
```

6. Verify Vault installation:
```
vault -version
```

## Part 2 – Configuring Vault

7. Create "RAFT" storage backend directory:
```
mkdir -p ./vault/data
```

8. Create Vault's config.hcl file:
```
vi config.hcl
```
```
storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = "true"
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true
```

9. Start Vault server:
```
vault server -config=config.hcl
```
10. Set the Vault address:
```
export VAULT_ADDR='http://127.0.0.1:8200'
```

11. Initialize Vault:
```
vault operator init
# This command outputs the unseal keys and root token, which are crucial for accessing Vault.
```
12. Unseal Vault:

- In the UI or via the command line, use the unseal keys to unseal the Vault.

13. Access Vault UI:

- Open a browser and navigate to:
```
http://<public-ip-of-ec2>:8200/ui/
# Use the 3 unseal keys and
# Use the root token for access.
```
## Part 3 – Storing and Managing Secrets

14. Verify Secrets Engine:
```
vault secrets list
```

15. Create and Write Policy: Create a policy file (myapp-policy.hcl):
```
path "secret/data/myapp/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/metadata/myapp/*" {
  capabilities = ["list"]
}
```
16. Write the policy to Vault:
```
vault policy write myapp-policy myapp-policy.hcl
```
17. Generate a Token with the Policy:
```
vault token create -policy=myapp-policy -ttl=48h
```

18. Set the New Token:
```
export VAULT_TOKEN=<newly-created-token>
```
 19. Store Secrets in Vault:
```
vault kv put secret/myapp aws_access_key_id='YOUR_AWS_ACCESS_KEY_ID' aws_secret_access_key='YOUR_AWS_SECRET_ACCESS_KEY' sonarqube_token='YOUR_SONARQUBE_TOKEN' artifactory_user='YOUR_ARTIFACTORY_USER' artifactory_password='YOUR_ARTIFACTORY_PASSWORD'
```

20. Retrieve Secrets via CLI:
```
vault kv get secret/myapp
```

21. Retrieve Secrets via UI:
- Go to browser:
```
http://<ec2-public-ip>:8200/ui/vault/secrets/secret/kv/list
```

This sequence covers the installation, initial configuration, policy management, and secret storage. 
It ensures that Vault is properly set up, unsealed, and accessible both through the command line and the web UI. 
Make sure to handle the unseal keys and tokens securely, as they are critical for Vault operations. 
If you're planning on using this setup in a production environment, consider enabling TLS and using a more secure storage backend than local file storage.

# Storing GitHub Credentials:

1. Create GitHub Secret in Vault:
```
vault kv put secret/github token='<replace-it-with-your-github-token>'
```

2. Verify the Stored GitHub Credentials:
```
vault kv get secret/github
``` 




