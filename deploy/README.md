# Ghost Deployment

This directory contains Docker Compose files for deploying Ghost in different environments.

## Files

- `docker-compose.production.yml` - Production deployment configuration
- `docker-compose.staging.yml` - Staging deployment configuration
- `.env.example` - Example environment variables file

## Environment Variables

The deployment configurations use environment variables to avoid hardcoding sensitive values.

### Required Variables

- `AWS_ACCOUNT_ID` - Your AWS Account ID for ECR image repository access

## Setup

1. **Copy the environment file:**
   ```bash
   cp .env.example .env
   ```

2. **Update the environment variables:**
   Edit the `.env` file and replace the example values with your actual AWS account ID:
   ```
   AWS_ACCOUNT_ID=your-actual-aws-account-id
   ```

3. **Deploy to staging:**
   ```bash
   docker-compose -f docker-compose.staging.yml up -d
   ```

4. **Deploy to production:**
   ```bash
   docker-compose -f docker-compose.production.yml up -d
   ```

## Alternative: Using Environment Variables Directly

You can also set environment variables directly without a `.env` file:

### Windows (PowerShell)
```powershell
$env:AWS_ACCOUNT_ID="your-actual-aws-account-id"
docker-compose -f docker-compose.production.yml up -d
```

### Linux/macOS
```bash
export AWS_ACCOUNT_ID="your-actual-aws-account-id"
docker-compose -f docker-compose.production.yml up -d
```

## Security Notes

- **Never commit your `.env` file with actual credentials to version control**
- The `.env.example` file shows the structure but contains placeholder values only
- **Always replace placeholder values** (like `123456789012`) with your actual AWS account ID
- Make sure your `.env` file is listed in `.gitignore`
- The example AWS account ID (`123456789012`) is a standard placeholder - not a real account