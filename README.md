# Dev Odoo Base Docker

[![Build & Push per Odoo version](https://github.com/BorovlevAS/dev_odoo_base_docker/actions/workflows/docker_build.yml/badge.svg)](https://github.com/BorovlevAS/dev_odoo_base_docker/actions/workflows/docker_build.yml)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-borovlevas%2Fodoo--base-blue)](https://hub.docker.com/r/borovlevas/odoo-base)

Base Docker images for Odoo development with VS Code Dev Containers.

## 📋 Overview

This repository contains **base Docker images** for Odoo development environments across multiple versions. Each version is built on Ubuntu (LTS releases) and provides a foundation with all necessary system dependencies, Python packages, and development tools pre-installed.

These images are designed to be **extended** in downstream projects where you add your specific Odoo source code, custom addons, and project-specific requirements. Images are automatically built and published to Docker Hub via GitHub Actions when changes are detected.

> **Note**: This repository contains only base images. For complete DevContainer setup examples, see separate project repositories that extend these base images.

## 🎯 Purpose

- **Multi-Version Support**: Separate base images for different Odoo versions (currently 14.0 and 17.0)
- **Base Images Only**: Provides foundation layers with all system dependencies and tools
- **Designed for Extension**: Meant to be extended with `FROM` directive in downstream Dockerfiles
- **DevContainer Ready**: Optimized for use as a base in VS Code DevContainer projects
- **Automated Builds**: CI/CD with GitHub Actions automatically rebuilds images when changes are detected
- **Version Isolation**: Each Odoo version maintains its own Dockerfile and configuration
- **Zero Odoo Source**: Contains no Odoo source code - add it in your project

## 🗂️ Repository Structure

```
docker/
├── 14.0/
│   ├── Dockerfile         # Odoo 14 base image
│   └── conf/
│       └── odoo-server.conf
└── 17.0/
    ├── Dockerfile         # Odoo 17 base image
    └── conf/
        └── odoo-server.conf
```

Each version directory contains:
- **Dockerfile**: Complete image definition for that Odoo version
- **conf/**: Configuration templates specific to that version

New Odoo versions can be added by creating a new directory following the same structure.

## 🚀 Features

### Odoo Version Support

- **Odoo 14.0**: Python 3.8+, PostgreSQL 12+
- **Odoo 17.0**: Python 3.10+, PostgreSQL 14+
- **Expandable**: Easy to add new versions following the existing structure

### System Components

Each image includes:
- **Base OS**: Ubuntu LTS (version specific to Odoo requirements)
- **Python**: Version appropriate for the Odoo release
- **Node.js**: Latest LTS version with npm
- **PostgreSQL Client**: Version matching Odoo requirements
- **wkhtmltopdf**: For PDF generation

### Development Tools

- **Git**: Version control
- **sudo**: Full sudo access for odoo user
- **debugpy**: Python debugging support
- **pre-commit**: Git hooks framework
- **Prettier**: Code formatting with XML and SQL plugins

### Odoo Dependencies

All required Python packages from respective Odoo version requirements:
- Web framework dependencies (Werkzeug, Jinja2, etc.)
- Database drivers (psycopg2)
- Image processing (Pillow, reportlab)
- Document handling (lxml, xlrd, xlwt, odfpy)
- Version-specific dependencies

### Additional Libraries

- **phonenumbers**: Phone number parsing and formatting
- **qrcode**: QR code generation
- **num2words**: Number to words conversion
- **python-magic**: File type identification
- **watchdog**: File system event monitoring

## 🏗️ Image Architecture

### User Configuration

- **User**: `odoo` (UID: 1000, GID: 1000)
- **Home**: `/home/odoo`
- **Workspace**: `/workspace`
- **Virtual Environment**: `/workspace/venv`

### Directory Structure

```
/workspace/
├── conf/               # Configuration files
│   └── odoo-server.conf
├── extra_addons/       # Custom addons directory
└── venv/              # Python virtual environment
```

### Exposed Ports

- **8069**: Odoo HTTP
- **8071**: Odoo HTTP (alternative)
- **8072**: Odoo Longpolling

## 🐳 Docker Hub

Images are available on Docker Hub under the `borovlevas/odoo-base` repository:

```bash
# Pull Odoo 14 base image
docker pull borovlevas/odoo-base:14.0

# Pull Odoo 17 base image
docker pull borovlevas/odoo-base:17.0
```

### Available Tags

Each version has multiple tags:
- `14.0` - Latest Odoo 14 build
- `14.0-YYYYMMDD` - Date-tagged builds
- `14.0-{SHA}` - Git commit-tagged builds
- `17.0` - Latest Odoo 17 build
- `17.0-YYYYMMDD` - Date-tagged builds
- `17.0-{SHA}` - Git commit-tagged builds

## 🔧 Usage

### Selecting the Right Version

Choose the base image that matches your Odoo version:

```dockerfile
# For Odoo 14 projects
FROM borovlevas/odoo-base:14.0

# For Odoo 17 projects
FROM borovlevas/odoo-base:17.0
```

### Extending the Base Image

These images are designed to be extended in your project's Dockerfile. Here's the typical pattern:

**1. Create your project Dockerfile:**

```dockerfile
# Extend the base image for your Odoo version
FROM borovlevas/odoo-base:17.0

USER odoo

# Install your project-specific Python packages
COPY extra_requirements.txt /tmp/extra_requirements.txt
RUN pip3 install -r /tmp/extra_requirements.txt

COPY requirements.txt /tmp/requirements.txt
RUN pip3 install -r /tmp/requirements.txt

# Your additional setup steps here
# Note: Odoo source code and addons are typically mounted as volumes
```

**2. Build your project image:**

```bash
docker build -t my-odoo17-dev -f ./docker/Dockerfile .
```

**3. Use with Docker Compose:**

```yaml
version: '3.8'
services:
  odoo:
    build:
      context: ./
      dockerfile: docker/Dockerfile
    ports:
      - "8069:8069"
      - "8072:8072"
    volumes:
      - ./:/workspaces
      - ./conf/odoo-server.conf:/etc/odoo/odoo-server.conf
    environment:
      - PGHOST=db
      - PGUSER=odoo
      - PGPASSWORD=odoo
    depends_on:
      - db
  
  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_DB=postgres
    volumes:
      - odoo-pg-data:/var/lib/postgresql/data

volumes:
  odoo-pg-data:
```

**4. Integrate with VS Code DevContainer:**

Your `.devcontainer/devcontainer.json` references the extended image:

```json
{
  "name": "Odoo 17 Development",
  "dockerComposeFile": [
    "../docker/docker-compose.yml",
    "docker-compose.yml"
  ],
  "service": "odoo",
  "workspaceFolder": "/workspaces",
  "remoteUser": "odoo"
}
```

### Why Not Use Directly?

These base images do **not** include:
- Odoo source code (mount or clone in your project)
- Your custom addons
- Project-specific Python packages
- Project-specific configuration

These are intentionally left to your downstream project for flexibility and version control.

## 🔨 Building Locally

### Prerequisites

- Docker Desktop or Docker Engine
- Git

### Build Command

```bash
# Clone the repository
git clone https://github.com/BorovlevAS/dev_odoo_base_docker.git
cd dev_odoo_base_docker

# Build specific version
docker build -t odoo-base:14.0 -f ./docker/14.0/Dockerfile ./docker/14.0
docker build -t odoo-base:17.0 -f ./docker/17.0/Dockerfile ./docker/17.0

# Or build all versions
for version in 14.0 17.0; do
  docker build -t odoo-base:${version} -f ./docker/${version}/Dockerfile ./docker/${version}
done
```

## ⚙️ Configuration

### Included Configuration

Each version includes a minimal Odoo configuration template in its `conf/` directory with basic settings appropriate for that version.

> **Note**: These configurations are starting points. In your project, you should mount your own configuration file with project-specific paths and settings.

### Using Your Own Configuration

In your project's docker-compose.yml:

```yaml
volumes:
  - ./conf/odoo-server.conf:/etc/odoo/odoo-server.conf
```

Or set via environment variable:

```yaml
environment:
  - ODOO_RC=/workspaces/conf/odoo-server.conf
```

## 🔄 CI/CD

### GitHub Actions Workflow

The repository uses a universal GitHub Actions workflow that automatically builds and publishes images to Docker Hub when changes are detected.

**Features**:
- **Smart Detection**: Automatically detects which Odoo versions have changes using path filters
- **Selective Builds**: Only rebuilds versions that have been modified
- **Multi-Version Support**: Handles multiple versions in parallel
- **Automated Tagging**: Creates version, date, and commit-based tags
- **Cache Optimization**: Uses Docker layer caching for faster builds

**Triggers**:
- Push to `main` branch with changes in `docker/**`
- Manual workflow dispatch

**Process for each changed version**:
1. Detects changes in version-specific directories (`docker/14.0/**`, `docker/17.0/**`)
2. Prepares build matrix with affected versions
3. Checks out repository
4. Sets up Docker Buildx
5. Logs in to Docker Hub
6. Builds and pushes images with multiple tags:
   - `{version}` - Latest for that version (e.g., `14.0`, `17.0`)
   - `{version}-YYYYMMDD` - Date-tagged build
   - `{version}-{SHA}` - Git commit-tagged build

### Adding New Versions

To add support for a new Odoo version:

1. Create a new directory: `docker/{VERSION}/`
2. Add `Dockerfile` and `conf/odoo-server.conf`
3. Update the workflow file to include the new version in path filters:
   ```yaml
   filters: |
     odoo14: 'docker/14.0/**'
     odoo17: 'docker/17.0/**'
     odoo18: 'docker/18.0/**'  # Add new version
   ```
4. Update the version detection loop in the workflow

The workflow will automatically start building the new version on the next push.

### Required Secrets

Configure these secrets in your GitHub repository:

- `DOCKERHUB_USER` - Docker Hub username
- `DOCKERHUB_PASSW` - Docker Hub password or access token

## 📦 Dependencies

### System Packages

Each image includes version-appropriate packages:
- **Build tools**: gcc, g++, make, python3-dev
- **Image libraries**: libjpeg, libpng, libwebp, libtiff
- **Database**: libpq-dev, postgresql-client
- **XML/HTML**: libxml2, libxslt, libfontconfig
- **Fonts**: fonts-noto-cjk (CJK language support)
- **PDF**: poppler-utils, wkhtmltopdf
- **LDAP**: libldap2-dev, libsasl2-dev

### Python Packages

All packages from the respective Odoo version's requirements.txt plus common development tools:
- debugpy (Python debugging)
- pre-commit (Git hooks framework)
- Version-specific packages as needed

### Node.js Packages (Global)

- npm (latest LTS)
- less
- prettier
- @prettier/plugin-xml
- prettier-plugin-sql

## 🤝 Contributing

Contributions are welcome! This includes:
- Adding support for new Odoo versions
- Improving existing Dockerfiles
- Updating dependencies
- Documentation improvements

Please feel free to submit a Pull Request:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. For new Odoo versions, create a new directory under `docker/` following the existing structure
4. Update the GitHub Actions workflow to include the new version
5. Test your changes locally
6. Commit your changes (`git commit -m 'Add support for Odoo X.Y'`)
7. Push to the branch (`git push origin feature/AmazingFeature`)
8. Open a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🔗 Related Projects

- [Odoo Official Repository](https://github.com/odoo/odoo)
- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)

> **Looking for a complete example?**  
> Full DevContainer setups that extend this base image will be available in separate project repositories.

## 💡 Tips and Best Practices

### Extending the Image

1. **Keep it minimal**: Only add what's specific to your project
2. **Layer efficiently**: Group related RUN commands to reduce image size
3. **Stay as USER odoo**: Maintain security by avoiding root operations
4. **Version your builds**: Tag your extended images with version numbers

### Virtual Environment

All Python packages in the base image are installed in `/workspace/venv`. This environment is automatically activated. In your extended Dockerfile:

```dockerfile
# pip3 will automatically use the virtual environment
RUN pip3 install -r /tmp/requirements.txt
```

### Development Workflow

1. **Mount Odoo source**: Clone Odoo in your project and mount it as a volume
2. **Mount your addons**: Keep custom addons in your project repository
3. **Mount configuration**: Use your project's odoo-server.conf
4. **Database persistence**: Use Docker volumes for PostgreSQL data

### Debugging

The image includes `debugpy` for Python debugging. When working in a DevContainer, VS Code runs inside the container, allowing direct debugging of Odoo processes. Your project's `.vscode/launch.json` can directly launch and debug Odoo:

```json
{
  "name": "Python: Odoo",
  "type": "debugpy",
  "request": "launch",
  "program": "/path/to/odoo-bin",
  "args": ["--config=/path/to/odoo-server.conf"],
  "console": "integratedTerminal"
}
```

### Performance

- Allocate at least 4GB RAM to Docker
- Use volume mounts with `consistency=cached` (macOS/Windows)
- Consider PostgreSQL version matching your Odoo version for better performance
  - PostgreSQL 12+ for Odoo 14
  - PostgreSQL 14+ for Odoo 17

## 🐛 Troubleshooting

### Common Issues

**Permission Denied**
- Ensure the odoo user (UID: 1000) has access to mounted volumes
- Check directory permissions on the host

**Database Connection Failed**
- Verify PostgreSQL container is running
- Check `db_host` in odoo-server.conf matches your database service name

**Port Already in Use**
- Change the port mapping: `-p 8070:8069` instead of `-p 8069:8069`

**Python Package Installation Fails**
- Ensure you're using the virtual environment: `source /workspace/venv/bin/activate`
- Update pip: `pip install --upgrade pip`

## 📧 Contact

- **Author**: BorovlevAS
- **GitHub**: [@BorovlevAS](https://github.com/BorovlevAS)
- **Docker Hub**: [borovlevas](https://hub.docker.com/u/borovlevas)

## 🙏 Acknowledgments

- Odoo Community for the amazing ERP framework
- Ubuntu and PostgreSQL teams for robust base images
- Contributors to all the open-source dependencies included in this image