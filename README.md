# Setting Up HashiCorp vault

## Part 1 – Installation Hashicorp Vault

Step 1 - Add PGP for the package signing key. 
```
sudo apt update && sudo apt install gpg 
```

Step 2 - Add the HashiCorp GPG key. 
```
wget O https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg 
```

Step 3 - Verify the key's fingerprint. 
```
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint 
```

Step 4 - Add the official HashiCorp Linux repository.
```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Step 5 - Update and install. 
```
sudo apt update && sudo apt install vault 
```

Step 6 – Verify vault installation
```
vault -version
```

Step 7 - Create "RAFT" storage backend directory
```
mkdir -p ./vault/data
```

Step 8 - Create Vault's config.hcl file 
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
Step 9 - Starting vault server (Production Env) using config.hcl
```
vault server -config=config.hcl
```

Step 10 - Export VAULT_ADDR
```
export VAULT_ADDR='http://127.0.0.1:8200'
```

Step 11 - Initialize vault
```   
vault operator init
# We will get the unseal key and the root token
# Which are mandatory for later
# We would need the token & unseal key to access the vault UI later
```

Step 12 - Initialize vault
```
vault operator init
```

Step 13 - Go to browser and access the vault Production UI
```
$ <public ip of ec2>:8200/ui/
# vault is sealed that means only we can read but not write into vault
# Unseal Key Portion: <paste here the unseal key – provide 3 keys>
# Now enter the root token
```

Step 14 - Unseal vault
```
vault operator unseal
# provide 3 unseal keys
# provide token
```





Recap of Commands:

1. Verify Secrets Engine:

```
vault secrets list
```

Update and Write Policy:

hcl

path "secret/data/myapp/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/metadata/myapp/*" {
  capabilities = ["list"]
}

Apply:

bash

vault policy write myapp-policy myapp-policy.hcl

Generate a Token with the Correct Policy:

bash

vault token create -policy=myapp-policy -ttl=24h

Use the New Token:

bash

export VAULT_TOKEN=<newly-created-token>

Store the Secrets:

bash

vault kv put secret/myapp aws_access_key_id='YOUR_AWS_ACCESS_KEY_ID' aws_secret_access_key='YOUR_AWS_SECRET_ACCE
