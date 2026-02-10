# bws-sync

CLI tool to sync environment secrets to [Bitwarden Secrets Manager](https://bitwarden.com/products/secrets-manager/) with GitHub Actions integration.

## What it does

- **Dev**: Reads your `.env` file and syncs all variables to BWS in one shot
- **Staging/Production**: Prompts you for each secret value interactively (like `gh secret set`)
- Automatically updates GitHub Actions workflow files with the correct BWS secret UUIDs
- Manages GitHub environment secrets (creates environments, sets `BWS_ACCESS_TOKEN`)

## Prerequisites

- [Bitwarden Secrets Manager CLI (`bws`)](https://bitwarden.com/help/secrets-manager-cli/)
- [GitHub CLI (`gh`)](https://cli.github.com/) - for staging/production GitHub integration
- `jq`
- Bash 4+ (for associative arrays)

## Setup

1. Create a config file with your BWS access tokens:

```bash
mkdir -p ~/.config/bws-sync

cat > ~/.config/bws-sync/secrets.conf << 'EOF'
BWS_ACCESS_TOKEN_DEV="your-dev-token"
BWS_ACCESS_TOKEN_STAGING="your-staging-token"
BWS_ACCESS_TOKEN_PRODUCTION="your-production-token"
EOF
```

You can override the config directory with `BWS_SYNC_CONFIG_DIR`:

```bash
export BWS_SYNC_CONFIG_DIR="/custom/path"
```

2. Make the script executable:

```bash
chmod +x sync-secrets.sh
```

## Usage

```bash
./sync-secrets.sh /path/to/your/project
```

The script will:

1. Ask you to select an environment (dev, staging, or production)
2. Ask for your BWS Project ID
3. Validate access to the project
4. Sync secrets based on the environment:
   - **Dev**: Reads from `.env` or `.env.local`, lists all variables, and syncs after confirmation
   - **Staging/Production**: Reads keys from `.env.example` (or existing BWS secrets as fallback), prompts for each value, then updates your GitHub Actions workflow file and environment secrets

## How it determines secret keys

The script looks for keys in this order:

1. `.env.example` - preferred, should list all required variables
2. `.env` - fallback for dev
3. `.env.local` - fallback for dev
4. Existing BWS secrets - fallback for staging/production

## GitHub Actions integration

For staging/production, the script:

- Updates `deploy-{env}.yml` workflow files with the correct `UUID > KEY` mappings in the `secrets: |` block
- Creates the GitHub environment if it doesn't exist
- Sets `BWS_ACCESS_TOKEN` as an environment secret
- Offers to clean up any other secrets (since BWS handles them all)

## License

MIT
