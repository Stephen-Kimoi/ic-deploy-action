# Internet Computer Deploy Action

A GitHub Action for deploying Internet Computer (IC) projects, including backend canisters and frontend assets.

## Features

- 🔐 Secure authentication using PEM keys
- 🌐 Deploy to multiple networks (local, testnet, ic)
- 🚀 Deploy specific canisters or all canisters
- 🎨 Optional frontend asset deployment
- ⚡️ Fast and reliable deployment process

## Usage

```yaml
name: Deploy to IC

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to IC
        uses: your-username/ic-deploy-action@v1
        with:
          pem_key: ${{ secrets.IC_PEM_KEY }}
          network: 'ic'
          canister_name: 'backend'  # Optional: deploy specific canister
          deploy_frontend: 'true'   # Optional: deploy frontend assets
          frontend_dir: 'dist'      # Optional: frontend assets directory
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pem_key` | Base64 encoded PEM key for authentication | Yes | - |
| `network` | Network to deploy to (local, testnet, ic) | Yes | 'ic' |
| `canister_name` | Name of the canister to deploy | No | - |
| `deploy_frontend` | Whether to deploy frontend assets | No | 'true' |
| `frontend_dir` | Directory containing frontend assets | No | 'dist' |


## Generating your PEM key: 
```bash 
# Create a new identity
dfx identity new test-identity --storage-mode=plaintext

# Export the identity to a PEM file
dfx identity export test-identity > test-identity.pem

# Convert to base64
base64 -i test-identity.pem > test-identity.pem.base64
```

## Security

- The PEM key should be stored as a GitHub secret
- The key is used only for deployment and is not stored permanently
- The action uses secure storage mode for identity management

## Example Workflows

### Deploy All Canisters with Frontend

```yaml
- name: Deploy to IC
  uses: your-username/ic-deploy-action@v1
  with:
    pem_key: ${{ secrets.IC_PEM_KEY }}
    network: 'ic'
    deploy_frontend: 'true'
```

### Deploy Specific Canister Only

```yaml
- name: Deploy to IC
  uses: your-username/ic-deploy-action@v1
  with:
    pem_key: ${{ secrets.IC_PEM_KEY }}
    network: 'ic'
    canister_name: 'backend'
    deploy_frontend: 'false'
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 