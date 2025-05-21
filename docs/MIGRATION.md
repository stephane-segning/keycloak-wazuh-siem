# Migration Plan for Project Restructuring

This document outlines the plan for restructuring the Keycloak-Wazuh Integration project. Since the Architect mode can only edit Markdown files, this plan will need to be implemented in Code mode.

## Directory Structure

The new directory structure will be:

```
keycloak-wazuh-monitoring/
├── .github/                          # GitHub-specific files
│   ├── ISSUE_TEMPLATE/               # Issue templates
│   └── workflows/                    # GitHub Actions workflows
├── docs/                             # Documentation
│   ├── architecture/                 # Architecture diagrams and explanations
│   ├── guides/                       # How-to guides
│   ├── images/                       # Images for documentation
│   └── examples/                     # Example use cases
├── config/                           # Configuration files
│   ├── keycloak/                     # Keycloak-specific configurations
│   │   ├── realm/                    # Realm configurations
│   │   └── providers/                # Information about required providers
│   ├── wazuh/                        # Wazuh-specific configurations
│   │   ├── decoders/                 # Custom decoders
│   │   ├── rules/                    # Custom rules
│   │   └── manager/                  # Manager configurations
│   └── syslog/                       # Syslog configuration
├── deployment/                       # Deployment configurations
│   ├── docker/                       # Docker deployment
│   │   ├── docker-compose.yml        # Main docker-compose file
│   │   └── .env.example              # Example environment variables
│   └── kubernetes/                   # Kubernetes deployment (future expansion)
├── templates/                        # Template files
│   └── certs/                        # Certificate configuration templates
├── scripts/                          # Utility scripts
│   ├── setup.sh                      # Setup script
│   └── generate-certs.sh             # Certificate generation script
├── .gitignore                        # Git ignore file
├── LICENSE                           # License file
└── README.md                         # Main README
```

## File Migration Plan

### 1. Create Directories

Create the following directories:

```bash
mkdir -p .github/ISSUE_TEMPLATE
mkdir -p .github/workflows
mkdir -p docs/images
mkdir -p config/keycloak/realm
mkdir -p config/keycloak/providers
mkdir -p config/wazuh/decoders
mkdir -p config/wazuh/rules
mkdir -p config/wazuh/manager
mkdir -p config/syslog
mkdir -p deployment/docker
mkdir -p deployment/kubernetes
mkdir -p templates/certs
mkdir -p scripts
```

### 2. Move Configuration Files

Move the existing configuration files to their new locations:

```bash
# Keycloak configuration
cp config/keycloak_config/realm.json config/keycloak/realm/

# Wazuh configuration
cp config/wazuh_cluster/local_decoder.xml config/wazuh/decoders/
cp config/wazuh_cluster/local_rules.xml config/wazuh/rules/
cp config/wazuh_cluster/wazuh_manager.conf.xml config/wazuh/manager/
cp config/wazuh_dashboard/opensearch_dashboards.yml config/wazuh/dashboard/
cp config/wazuh_dashboard/wazuh.yml config/wazuh/dashboard/
cp config/wazuh_indexer/internal_users.yml config/wazuh/indexer/
cp config/wazuh_indexer/wazuh.indexer.yml config/wazuh/indexer/

# Certificate configuration
cp config/certs.yml templates/certs/
```

### 3. Move Deployment Files

Move the deployment files to their new locations:

```bash
cp docker-compose.yml deployment/docker/
cp generate-indexer-certs.yml deployment/docker/
```

### 4. Create Script Files

Create script files for common tasks:

```bash
# Create setup script
cat > scripts/setup.sh << 'EOF'
#!/bin/bash
# Setup script for Keycloak-Wazuh Integration

# Check if Docker is installed
if ! command -v docker &> /dev/null; then
    echo "Docker is not installed. Please install Docker and Docker Compose."
    exit 1
fi

# Check if Docker Compose is installed
if ! command -v docker-compose &> /dev/null; then
    echo "Docker Compose is not installed. Please install Docker Compose."
    exit 1
fi

# Generate certificates
echo "Generating certificates..."
cd deployment/docker
docker-compose -f generate-indexer-certs.yml up

# Start the environment
echo "Starting the environment..."
docker-compose up -d

echo "Setup complete. You can access:"
echo "- Keycloak: http://localhost:9000 (admin/password)"
echo "- Wazuh Dashboard: https://localhost:5601 (admin/SecretPassword)"
EOF
chmod +x scripts/setup.sh

# Create certificate generation script
cat > scripts/generate-certs.sh << 'EOF'
#!/bin/bash
# Certificate generation script for Keycloak-Wazuh Integration

cd deployment/docker
docker-compose -f generate-indexer-certs.yml up
EOF
chmod +x scripts/generate-certs.sh
```

### 5. Create License File

Create a license file:

```bash
cat > LICENSE << 'EOF'
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF
```

### 6. Update .gitignore

Update the .gitignore file:

```bash
cat > .gitignore << 'EOF'
# Docker runtime files
.docker

# IDE files
.idea
.vscode

# Log files
log
logs

# Certificate files
config/wazuh_indexer_ssl_certs
templates/certs/generated

# Environment files
.env
deployment/docker/.env

# Node modules
node_modules

# OS files
.DS_Store
Thumbs.db
EOF
```

### 7. Create GitHub Templates

Create GitHub issue templates:

```bash
cat > .github/ISSUE_TEMPLATE/bug_report.md << 'EOF'
---
name: Bug report
about: Create a report to help us improve
title: ''
labels: bug
assignees: ''

---

**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected behavior**
A clear and concise description of what you expected to happen.

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Environment (please complete the following information):**
 - OS: [e.g. Ubuntu 20.04]
 - Docker version: [e.g. 20.10.7]
 - Docker Compose version: [e.g. 1.29.2]
 - Keycloak version: [e.g. 26.2.3]
 - Wazuh version: [e.g. 4.12.0]

**Additional context**
Add any other context about the problem here.
EOF

cat > .github/ISSUE_TEMPLATE/feature_request.md << 'EOF'
---
name: Feature request
about: Suggest an idea for this project
title: ''
labels: enhancement
assignees: ''

---

**Is your feature request related to a problem? Please describe.**
A clear and concise description of what the problem is. Ex. I'm always frustrated when [...]

**Describe the solution you'd like**
A clear and concise description of what you want to happen.

**Describe alternatives you've considered**
A clear and concise description of any alternative solutions or features you've considered.

**Additional context**
Add any other context or screenshots about the feature request here.
EOF
```

## Next Steps

After implementing this migration plan in Code mode, the project will have a more organized structure that is better suited for open source sharing on GitHub. The documentation has already been created in Architect mode, so the focus in Code mode will be on moving the actual files to their new locations.