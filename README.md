## Reusable Github workflow to build, deploy and verify solana programs and IDLs

This repository provides GitHub workflows that automatically build, verify, and deploy solana programs including their IDL uploads.
These workflows use the [solana developers github actions](https://github.com/solana-developers/github-actions) and combine them into an easy to use workflow. 
There is also squads multisig support which is highly recommended to be used.
For the workflow you just need to set `use-squads` to true and add the needed secrets to use the [squads program action](https://github.com/solana-developers/squads-program-action) automatically.

### Features

- ✅ Automated program builds
- ✅ Program verification against source code
- ✅ IDL buffer creation and uploads
- ✅ Squads multisig integration
- ✅ Program deploys for both devnet and mainnet
- ✅ Compute budget optimization
- ✅ Retry mechanisms for RPC failures
- ✅ Running anchor tests
- ✅ Caching for faster reruns for all installs
- ✅ Extend program automatically
  
### How to use

Easiest is to follow this [Video Guide](https://youtu.be/h-ngRgWW_IM). 

Create a Solana Program. Either a [native program](https://solana.com/de/developers/guides/getstarted/local-rust-hello-world) and an [anchor program](https://www.anchor-lang.com/docs/quickstart/local). The program with the `anchor.toml` and/or `cargo.toml` need to be in the root of your repository for these workflows to work out of the box.

```yaml
name: Devnet Build and Deploy

on:
  workflow_dispatch:
    inputs:
      priority_fee:
        description: "Priority fee for transactions"
        required: true
        default: "300000"
        type: string

jobs:
  build:
    uses: solana-developers/github-workflows/.github/workflows/reusable-build.yaml@v0.3.6
    with:
      program: "hello_world"
      program-id: "YOUR_PROGRAM_ID"
      network: "devnet"
      deploy: true
      upload_idl: true
      verify: false
      use-squads: false
      priority-fee: ${{ github.event.inputs.priority_fee }}
    secrets:
      DEVNET_SOLANA_DEPLOY_URL: ${{ secrets.DEVNET_SOLANA_DEPLOY_URL }}
      DEVNET_DEPLOYER_KEYPAIR: ${{ secrets.DEVNET_DEPLOYER_KEYPAIR }}
      PROGRAM_ADDRESS_KEYPAIR: ${{ secrets.PROGRAM_ADDRESS_KEYPAIR }}
```

Or for mainnet with source code verification and squads integration:

```yaml
name: Release to mainnet with IDL and verify

on:
  workflow_dispatch:
    inputs:
      priority_fee:
        description: "Priority fee for transactions"
        required: true
        default: "300000"
        type: string

jobs:
  build:
    uses: solana-developers/github-workflows/.github/workflows/reusable-build.yaml@v0.3.6
    with:
      program: "transaction_example"
      program-id: "YOUR_PROGRAM_ID"
      network: "mainnet"
      deploy: true
      upload_idl: true
      verify: true
      use-squads: true
      priority-fee: ${{ github.event.inputs.priority_fee }}

    secrets:
      MAINNET_SOLANA_DEPLOY_URL: ${{ secrets.MAINNET_SOLANA_DEPLOY_URL }}
      MAINNET_DEPLOYER_KEYPAIR: ${{ secrets.MAINNET_DEPLOYER_KEYPAIR }}
      PROGRAM_ADDRESS_KEYPAIR: ${{ secrets.PROGRAM_ADDRESS_KEYPAIR }}
      MAINNET_MULTISIG: ${{ secrets.MAINNET_MULTISIG }}
      MAINNET_MULTISIG_VAULT: ${{ secrets.MAINNET_MULTISIG_VAULT }}
```

There are three examples:

- [Anchor Program](https://github.com/Woody4618/anchor-github-action-example)
- [Native Program](https://github.com/Woody4618/native-solana-github-action-example)
- [Anchor Program using Squads](https://github.com/Woody4618/workflow-tutorial) 

### Required Secrets for specific actions

Some of the options of the build workflow require you to add secrets to your repository:

```bash
# Network RPC URLs
DEVNET_SOLANA_DEPLOY_URL=   # Your devnet RPC URL - Recommended to use a payed RPC url
MAINNET_SOLANA_DEPLOY_URL=  # Your mainnet RPC URL - Recommended to use a payed RPC url

# Deployment Keys
DEVNET_DEPLOYER_KEYPAIR=    # Keypaig in the format of byte array [3, 45, 23, ...]
MAINNET_DEPLOYER_KEYPAIR=   # Keypaig in the format of byte array [3, 45, 23, ...]

PROGRAM_ADDRESS_KEYPAIR=    # Keypair of the program address - Needed for initial deploy and for native programs to find the program address. Can also be overwritten in the workflow if you dont have the keypair.

# For Squads integration (There is sadly no devnet squads ui)
MAINNET_MULTISIG=          # Mainnet Squads multisig address
MAINNET_MULTISIG_VAULT=    # Mainnet Squads vault address
```

This is how you can run anchor tests everytime tests or the program changed on push: 

```yaml
name: Anchor Tests

on:
  push:
    branches: [main]
    paths:
      - "programs/**"
      - "tests/**"
      - "Anchor.toml"
      - "Cargo.toml"
      - "Cargo.lock"
  workflow_dispatch:
    inputs:
      program:
        description: "Program to test"
        required: true
        default: "transaction_example"

jobs:
  test:
    uses: solana-developers/github-workflows/.github/workflows/test.yaml@v0.3.6
    with:
      program: ${{ 'transaction_example' }}
```

### Extend and automate

You can easily extend or change your workflow. For example run the build workflow automatically on every push to a development branch.

```bash
  push:
    branches:
      - develop
      - dev
      - development
    paths:
      - 'programs/**'
      - 'Anchor.toml'
      - 'Cargo.toml'
      - 'Cargo.lock'
```

Or run a new release to mainnet on every tag push for example.

```bash
  push:
    tags:
      - 'v*'
```

Or you can setup a matrix build for multiple programs and networks.
Customize the workflow to your needs!

## How to setup Squads integration:

In general its recommended to use the [Squads Multisig](https://docs.squads.so/main/getting-started/create-a-squad) or any other multisig plattform to manage your programs.
It makes your program deployments more secure and is considered good practice.

1. Setup a new squad in [Squads](https://v4.squads.so/squads/) then transfer your program authority to the squad.

<img width="1345" alt="image" src="https://github.com/user-attachments/assets/c1b9d003-806f-4389-bf4c-3275f180f479" />


2. Add your local keypair to the squad as a member (At least needs to have voter permissions) so that it can propose transactions. And also add that keypair as a github secret.
   To run it locally add the following to your .secrets file:

```bash
DEVNET_DEPLOYER_KEYPAIR=
MAINNET_DEPLOYER_KEYPAIR=
```

<img width="832" alt="image" src="https://github.com/user-attachments/assets/492eee0c-48d0-4748-838e-849d7b91f773" />


2. Add the multisig information to your `.secrets` file if you want to run it locally or add them to your github action secrets (Not workflow secrets) if you want to run it in github actions:

<img width="1384" alt="image" src="https://github.com/user-attachments/assets/8bb62dab-d17b-4163-be0f-52ce51affc32" />


```bash
DEVNET_MULTISIG=        # Sadly at the time of writing squads V4 does not support devnet
DEVNET_MULTISIG_VAULT=  # Sadly at the time of writing squads V4 does not support devnet
MAINNET_MULTISIG=
MAINNET_MULTISIG_VAULT=
```

Where Multisig vault is the address you can find on the top left corner in the [Squads Dachboard](https://v4.squads.so/squads/)
The MULTISIG is the address of the multisig you want to use this one you can find the the settings. Its a bit more hidden so that people dont accidentally use it as program upgrade authority.

<img width="1735" alt="image" src="https://github.com/user-attachments/assets/34584a9a-62b9-42c9-a6c4-4e4bf99e6631" />

What this workflow will do is write a program and an IDL buffer for your program and then propose a transaction that you can approve in the Squads UI.

Once the build was successful you can see the program upgrade transaction in your squads ui: 

<img width="1836" alt="image" src="https://github.com/user-attachments/assets/fde50e11-00b8-4c3b-923f-f18029edacdf" />


### Additional step for verification when using squads

The verification process with Osec API will start automatically as soon as you submitted the transaction in squads that wrote the Verify PDA. 

If it does not show as verified in the explorer after a while you can use this command to see the proggress of the verification or trigger a new run if it did not trigger automatcally: 

```bash
solana-verify remote submit-job --program-id <yourProgramId>  --uploader <yourSquadVaultAddress>
```


Close Buffer:

In case your workflow fails and the buffer was already created and transfered to your squads vault you can close that buffer using this [script](https://github.com/solana-developers/github-actions?tab=readme-ov-file#close-buffer-in-case-of-failure). 


### Running the actions locally (optional)

If you for some reason want to run the actions locally you can do so with the following commands using the act command.

Follow the instructions [here](https://nektosact.com/installation/index.html) to install act.

1. Build

You need to copy the workflow file to your local `.github/workflows` directory because act does not support reusable workflows.
Just pick the parameters you want. This is using act to run the workflow locally. Good for testing or if you dont want to install anything because this is running in docker and outputs the build artifacts as well.

```bash
act -W .github/workflows/reusable-build.yaml \
 --container-architecture linux/amd64 \
 --secret-file .secrets \
 workflow_dispatch \
 --input program=transaction-example \
 --input network=devnet \
 --input deploy=true \
 --input upload_idl=true \
 --input verify=true \
 --input use-squads=false
```

2. Run anchor tests

Note: The anchor tests use solana-test-validator which does not work in act docker container on mac because of AVX dependency. Either run them in github, locally without docker or open PR to fix it. I couldnt find a nice way to make local-test-validator run in act.
You can adjust the workflow to run your specific tests as well.

```bash
act -W .github/workflows/test.yaml \
 --container-architecture linux/amd64 \
 --secret-file .secrets \
 workflow_dispatch \
 --input program=transaction-example
```


## 📝 Todo List

### Program Verification

- [x] Trigger verified build PDA upload
- [x] Verify build remote trigger
- [x] Support and test squads Verify
- [x] Support and test squads IDL
- [x] Support and test squads Program deploy

### Action Improvements

- [x] Separate IDL and Program buffer action
- [x] Remove deprecated cache functions
- [x] Remove node-version from anchor build
- [x] Skip anchor build when native program build
- [ ] Make verify build and anchor build in parallel
- [x] Trigger release build on tag push
- [x] Trigger devnet releases on develop branch?
- [x] Make solana verify also work locally using cat
- [x] Use keypairs to find deployer address to remove 2 secrets
- [x] Add priority fees
- [x] Add extend program if needed
- [x] Bundle the needed TS scripts with the .github actions for easier copy paste

### Testing & Integration

- [x] Add running tests
  - Research support for different test frameworks
- [ ] Add Codama support
- [ ] Add to solana helpers or mucho -> release

