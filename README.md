[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)      [![Deploy to IC](https://github.com/Stephen-Kimoi/actions-testing/actions/workflows/deploy.yml/badge.svg)](https://github.com/Stephen-Kimoi/actions-testing/actions/workflows/deploy.yml)

# Internet Computer (ICP) Deploy Action

A GitHub Action for automating deployments on Internet Computer (ICP), for both backend canisters (Smart Contracts) and frontend assets. 

## Features

- 🔐 Secure authentication using PEM keys
- 🌐 Deploy to multiple networks (local, testnet, ic)
- 📦 Deploy specific canisters or all canisters
- 🎨 Optional frontend asset deployment
- ⚡️ Fast and reliable deployment process
- 🔄 Upgrade mode for existing canisters
- 📦 Automatic WASM file management

## Usage

### Prerequisites

1. **Mainnet Deployment**: Ensure you have deployed your canisters first. This action relies on the `canister_ids.json` file that is generated during deployment.
  - If you provision canisters via the [CycleOps canister creation feature](https://docs.cycleops.dev/changelog/canister-creation), make sure to run `dfx deploy --network ic` afterwards so the canister already has code before the action attempts an upgrade. 
2. **GitHub Secrets**: Set up your `IC_PEM_KEY` secret in your GitHub repository settings. See [Generating your PEM key](#generating-your-pem-key) below for instructions.
3. **Recommended Project Structure**:

   Your project should have a structure similar to the following:

   ```
   your-project/
   ├── src/
   │   └── backend/                  # Backend Rust canister source
   │   └── frontend/                 # Frontend source (if any)
   │       └── dist/                 # Built frontend assets
   ├── dfx.json                      # DFINITY project configuration
   ├── canister_ids.json             # Canister IDs (generated after deployment)
   ├── Cargo.toml                    # Rust package manifest (for backend)
   ├── package.json                  # Node.js manifest (for frontend, if any)
   ├── tsconfig.json                 # TypeScript config (if any)
   ├── node_modules/                 # Node.js dependencies (if any)
   └── ...
   ```

   - Adjust `backend` and `frontend` to match your actual canister/package names.
   - The action expects the backend and frontend directories to be specified in the workflow inputs. 

### Setup Steps

1. **Deploy Canisters Locally:**
   ```bash
   dfx deploy --network ic
   ```
   This generates the required `canister_ids.json` file.

2. **Create Workflow Directory:**
   ```bash
   mkdir -p .github/workflows
   ```

3. **Create Deploy File:**
   Create a `deploy.yml` file inside the `.github/workflows` directory.

4. **Add Workflow Content:**
   Paste the following into your `deploy.yml` file:

   ```yaml
   name: Deploy to IC

   on:
     push:
       branches: [ main ]
     workflow_dispatch:

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Deploy to IC
           uses: Stephen-Kimoi/ic-deploy-action/action@v0.1.0
           with:
             pem_key: ${{ secrets.IC_PEM_KEY }}
             network: 'ic'
             canister_name: 'your_backend_canister_name'
             deploy_frontend: 'true'
             frontend_dir: 'src/your_frontend_dir/dist'
             backend_package: 'your_backend_package'
             frontend_package: 'your_frontend_package'
             frontend_src_dir: 'src/your_frontend_dir'
   ```

5. **Generate PEM Key:**
   - Create a new identity:
     ```bash
     dfx identity new test-identity --storage-mode=plaintext
     ```
   - Export the identity to a PEM file:
     ```bash
     dfx identity export test-identity > test-identity.pem
     ```
   - Convert to base64:
     ```bash
     base64 -i test-identity.pem > test-identity.pem.base64
     ```

6. **Add to GitHub Secrets:**
   - Add your base64-encoded PEM key as `IC_PEM_KEY` in your GitHub repository secrets.
   - Go to your repository settings, scroll to "Secrets and Variables" > "Actions", and add a new repository secret named `IC_PEM_KEY`.

7. **Push to GitHub:**
   - Push your code to GitHub and the deployment will run automatically on push to `main` or via manual workflow dispatch.

### Important Notes

- The action requires a valid `canister_ids.json` file for deployment
- Canisters must be pre-deployed before using this action
- The action will only upgrade existing canisters, it will not create new ones
- Frontend assets are deployed using the `store` method of the frontend canister

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pem_key` | Base64 encoded PEM key for authentication | Yes | - |
| `network` | Network to deploy to (local, testnet, ic) | Yes | 'ic' |
| `canister_name` | Name of the canister to deploy | No | - |
| `deploy_frontend` | Whether to deploy frontend assets | No | 'true' |
| `deploy_backend` | Whether to build and deploy backend canisters | No | 'true' |
| `frontend_dir` | Directory containing frontend assets | No | 'dist' |
| `backend_package` | Name of your backend Rust package as specified in Cargo.toml | No (required when `deploy_backend` is `'true'`) | - |
| `frontend_package` | Name of your frontend package | No | 'frontend' |
| `build_args` | Arguments to pass to cargo build | No | '' |
| `test` | Run tests with cargo test | No | 'true' |
| `gzip` | Gzip the wasm file | No | 'true' |
| `frontend_src_dir` | Directory containing frontend source code | No | 'src/frontend' |

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
<!-- - Canister IDs are stored in `.github/canister_ids.json` for future deployments -->

## Example Workflows

### Deploy Backend and Frontend

```yaml
- name: Deploy to IC
  uses: Stephen-Kimoi/ic-deploy-action/action@v0.1.0
  with:
    pem_key: ${{ secrets.IC_PEM_KEY }}
    network: 'ic'
    canister_name: 'your_backend_canister_name'
    deploy_frontend: 'true'
    deploy_backend: 'true'
    frontend_dir: 'src/your_frontend_dir/dist'
    backend_package: 'your_backend_package'
    frontend_package: 'your_frontend_package'
    frontend_src_dir: 'src/your_frontend_dir'
```

### Deploy Backend Only

```yaml
- name: Deploy to IC
  uses: Stephen-Kimoi/ic-deploy-action/action@v0.1.0
  with:
    pem_key: ${{ secrets.IC_PEM_KEY }}
    network: 'ic'
    canister_name: 'your_backend_canister_name'
    deploy_backend: 'true'
    deploy_frontend: 'false'
    backend_package: 'your_backend_package'
```

### Deploy Frontend Only

```yaml
- name: Deploy to IC
  uses: Stephen-Kimoi/ic-deploy-action/action@v0.1.0
  with:
    pem_key: ${{ secrets.IC_PEM_KEY }}
    network: 'ic'
    canister_name: 'your_frontend_canister_name'
    deploy_frontend: 'true'
    deploy_backend: 'false'
    frontend_dir: 'src/your_frontend_dir/dist'
    frontend_package: 'your_frontend_package'
    frontend_src_dir: 'src/your_frontend_dir'
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 