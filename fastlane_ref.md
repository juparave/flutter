# Fastlane iOS CI/CD Setup Reference

This guide documents the complete setup process for Fastlane with iOS CI/CD based on real-world implementation experience.

## Prerequisites

- Apple Developer Account with admin access
- GitHub repository for storing certificates and provisioning profiles
- App Store Connect API Key

## Initial Setup

### 1. Install Fastlane

```bash
sudo gem install fastlane
```

### 2. Initialize Fastlane in iOS project

```bash
cd your_project/ios/
fastlane init
```

### 3. Initialize Match for Certificate Management

```bash
fastlane match init
```

When prompted, provide your Git repository URL for storing certificates (e.g., `https://github.com/username/certificates`).

## Configuration Files

### Gemfile
```ruby
source "https://rubygems.org"

gem "fastlane"
```

### Appfile
```ruby
app_identifier("com.yourcompany.yourapp")
apple_id("your-apple-id@email.com")

itc_team_id("Your_ITC_Team_ID") # App Store Connect Team ID
team_id("Your_Team_ID") # Developer Portal Team ID
```

### Matchfile
```ruby
git_url("https://github.com/username/certificates.git")
storage_mode("git")
type("appstore") # Default type
app_identifier("com.yourcompany.yourapp")
username("your-apple-id@email.com")
```

### Fastfile
```ruby
# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools

GIT_AUTHORIZATION = ENV["GIT_AUTHORIZATION"]

lane :beta do
  # Sanitize environment variables
  asc_key_content = ENV["APP_STORE_CONNECT_API_KEY_CONTENT"].strip
  asc_key_id = ENV["APP_STORE_CONNECT_API_KEY_ID"]
  asc_issuer_id = ENV["APP_STORE_CONNECT_ISSUER_ID"]

  # Create App Store Connect API key once
  api_key = app_store_connect_api_key(
    key_id: asc_key_id,
    issuer_id: asc_issuer_id,
    key_content: asc_key_content
  )

  # Fetch development and app-store certificates & profiles
  match(
    type: "development",
    app_identifier: "com.yourcompany.yourapp",
    git_basic_authorization: Base64.strict_encode64(GIT_AUTHORIZATION),
    readonly: true,
    api_key: api_key
  )
  match(
    type: "appstore",
    app_identifier: "com.yourcompany.yourapp",
    git_basic_authorization: Base64.strict_encode64(GIT_AUTHORIZATION),
    readonly: true,
    api_key: api_key
  )

  # Disable automatic signing and use manual profiles
  update_code_signing_settings(
    use_automatic_signing: false,
    path: "Runner.xcodeproj",
    profile_name: "match AppStore com.yourcompany.yourapp",
    code_sign_identity: "Apple Distribution: Your Name (TEAM_ID)"
  )

  # Build with provisioning update flag
  build_app(
    export_method: "app-store",
    workspace: "Runner.xcworkspace",
    scheme: "Runner",
    xcargs: "-allowProvisioningUpdates"
  )

  # Upload to TestFlight
  upload_to_testflight(
    skip_waiting_for_build_processing: true,
    api_key: api_key
  )
end
```

## GitHub Actions Workflow

### .github/workflows/ios-deploy.yml
```yaml
name: Build and Deploy iOS App

on:
  push:
    branches:
      - deploy

jobs:
  build-and-deploy:
    runs-on: macos-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable

      - name: Install dependencies
        working-directory: ./app
        run: flutter pub get

      - name: Run tests
        working-directory: ./app
        run: flutter test

      - name: Set up Ruby (for Fastlane)
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.2

      - name: Install Bundler and dependencies
        working-directory: ./app/ios
        run: |
          gem install bundler
          bundle install
          cd .. && flutter pub get
          cd ios && pod install

      - name: Deploy to TestFlight
        working-directory: ./app/ios
        env:
          APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
          APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY_CONTENT: ${{ secrets.APP_STORE_CONNECT_API_KEY_CONTENT }}
          GIT_AUTHORIZATION: ${{ secrets.GIT_AUTHORIZATION }}
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
        run: bundle exec fastlane beta
```

## Required GitHub Secrets

Set these in your GitHub repository settings:

1. **APP_STORE_CONNECT_API_KEY_ID**: The key ID from App Store Connect
2. **APP_STORE_CONNECT_ISSUER_ID**: The issuer ID from App Store Connect
3. **APP_STORE_CONNECT_API_KEY_CONTENT**: Base64 encoded .p8 file content
4. **GIT_AUTHORIZATION**: Base64 encoded `username:personal_access_token` for certificate repo
5. **MATCH_PASSWORD**: Password used to encrypt certificates in the Git repository

## Step-by-Step Setup Process

### 1. Create App Store Connect API Key

1. Go to App Store Connect → Users and Access → Keys
2. Create a new API key with App Manager role
3. Download the .p8 file
4. Note the Key ID and Issuer ID

### 2. Encode API Key for GitHub Secrets

```bash
# Encode the .p8 file content
cat AuthKey_XXXXXXXXXX.p8 | base64

# Create Git authorization token
echo -n "username:github_personal_access_token" | base64
```

### 3. Set up Certificate Repository

```bash
# Initialize match (creates certificate repo)
fastlane match init

# Generate development certificates
fastlane match development

# Generate app store certificates
fastlane match appstore
```

### 4. Local Testing

Before deploying to CI, test locally:

```bash
cd ios/
bundle install
bundle exec fastlane beta
```

## Common Issues and Solutions

### 1. "No profiles found" Error
- Ensure both development and appstore profiles are fetched
- Use manual code signing instead of automatic
- Verify team IDs match in all configuration files

### 2. "String contains null byte" Error
- Strip whitespace from API key content: `ENV["APP_STORE_CONNECT_API_KEY_CONTENT"].strip`
- Ensure base64 encoding is correct
- Try also this format in github repo sectrets: `"-----BEGIN EC PRIVATE KEY-----\nfewfawefawfe\n-----END EC PRIVATE KEY-----"`

### 3. CocoaPods Build Errors
- Run `pod install` before building
- Ensure Flutter dependencies are installed first

### 4. Keychain Password Warnings
- These can be ignored in CI environment
- Set `keychain_password` in match if needed locally

### 5. Bundle Exec Required
- Always use `bundle exec fastlane` instead of just `fastlane`
- Install bundler and dependencies in CI

## Environment Variables (.env for local development)

```bash
# .env file (DO NOT commit to repository)
APP_STORE_CONNECT_API_KEY_ID=XXXXXXXXXX
APP_STORE_CONNECT_ISSUER_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
APP_STORE_CONNECT_API_KEY_CONTENT=base64_encoded_p8_content
GIT_AUTHORIZATION=base64_encoded_username_token
MATCH_PASSWORD=your_match_password
```

## Command History Reference

```bash
# Initial setup
sudo gem install fastlane
fastlane init
fastlane match init

# Bundle management
bundle clean --force
bundle install
bundle update

# Certificate generation
fastlane match development
fastlane match appstore

# Local testing
bundle exec fastlane beta

# Git operations for deployment
git checkout deploy && git pull --rebase && git merge main && git push && git checkout main
```

## Best Practices

1. **Always use `bundle exec`** to ensure correct gem versions
2. **Test locally first** before pushing to CI
3. **Use manual code signing** for CI environments
4. **Store certificates in private Git repository** with Match
5. **Use environment variables** for all sensitive data
6. **Run `pod install`** after Flutter dependencies
7. **Keep Fastlane and gems updated** regularly
8. **Use specific Ruby version** in CI for consistency

## Troubleshooting

- Check Fastlane logs for detailed error messages
- Verify all environment variables are set correctly
- Ensure certificate repository is accessible
- Check App Store Connect API key permissions
- Verify team IDs and bundle identifiers match across all files
