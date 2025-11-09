# Dev Odoo Base Docker

[![Build and push Docker images](https://github.com/BorovlevAS/dev_odoo_base_docker/actions/workflows/docker_build.yml/badge.svg)](https://github.com/BorovlevAS/dev_odoo_base_docker/actions/workflows/docker_build.yml)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-borovlevas%2Fdev__odoo17__base__docker-blue)](https://hub.docker.com/r/borovlevas/dev_odoo17_base_docker)

Base Docker image for Odoo 17 development with VS Code Dev Containers.

## 📋 Overview

This repository contains a **base Docker image** for Odoo 17 development environments. Built on Ubuntu Jammy (22.04 LTS), it provides a foundation with all necessary system dependencies, Python packages, and development tools pre-installed.

This image is designed to be **extended** in downstream projects where you add your specific Odoo source code, custom addons, and project-specific requirements. It is automatically built and published to Docker Hub via GitHub Actions.

> **Note**: This repository contains only the base image. For a complete DevContainer setup example, see separate project repositories that extend this base image.

## 🎯 Purpose

- **Base Image Only**: Provides a foundation layer with all system dependencies and tools
- **Designed for Extension**: Meant to be extended with `FROM` directive in downstream Dockerfiles
- **DevContainer Ready**: Optimized for use as a base in VS Code DevContainer projects
- **Multi-Architecture**: Supports AMD64, ARM64, and PPC64EL architectures
- **Zero Odoo Source**: Contains no Odoo source code - add it in your project
- **Automated Builds**: CI/CD with GitHub Actions ensures fresh images

## 🚀 Features

### System Components

- **Base OS**: Ubuntu Jammy (22.04 LTS)
- **Python**: Python 3 with virtual environment support
- **Node.js**: Version 18.x with npm 10
- **PostgreSQL Client**: Latest version from official PostgreSQL repository
- **wkhtmltopdf**: Version 0.12.6.1-3 for PDF generation

### Development Tools

- **Git**: Version control
- **sudo**: Full sudo access for odoo user
- **debugpy**: Python debugging support
- **pre-commit**: Git hooks framework
- **Prettier**: Code formatting with XML and SQL plugins

### Odoo Dependencies

All required Python packages from Odoo 17 requirements:
- Web framework dependencies (Werkzeug, Jinja2, etc.)
- Database drivers (psycopg2)
- Image processing (Pillow, reportlab)
- Document handling (lxml, xlrd, xlwt, odfpy)
- And many more...

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

The image is available on Docker Hub:

```bash
docker pull borovlevas/dev_odoo17_base_docker:latest
```

### Available Tags

- `latest` - Latest stable build
- `1.0.0` - Specific version tag

## 🔧 Usage

### Extending the Base Image

This image is designed to be extended in your project's Dockerfile. Here's the typical pattern:

**1. Create your project Dockerfile:**

```dockerfile
# Extend the base image
FROM borovlevas/dev_odoo17_base_docker:latest

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

This base image does **not** include:
- Odoo source code (mount or clone in your project)
- Your custom addons
- Project-specific Python packages
- Project-specific configuration

These are intentionally left to your downstream project for flexibility.

## 🔨 Building Locally

### Prerequisites

- Docker Desktop or Docker Engine
- Docker Buildx (for multi-architecture builds)

### Build Command

```bash
# Clone the repository
git clone https://github.com/BorovlevAS/dev_odoo_base_docker.git
cd dev_odoo_base_docker

# Build for your architecture
docker build -t dev_odoo17_base_docker -f ./docker/Dockerfile .

# Build for multiple architectures
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t dev_odoo17_base_docker \
  -f ./docker/Dockerfile .
```

## ⚙️ Configuration

### Included Configuration

The image includes a minimal Odoo configuration template at `/workspace/conf/odoo-server.conf` with basic settings:

```ini
[options]
addons_path = /workspace/odoo, /workspace/odoo/addons, /workspace/simbioz_repo/extra_addons
admin_passwd = admin
db_host = db
db_port = 5432
db_user = odoo
db_password = odoo
http_port = 8069
longpolling_port = 8072
```

> **Note**: This configuration is a starting point. In your project, you should mount your own configuration file with project-specific paths and settings.

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

The repository includes a GitHub Actions workflow that automatically builds and publishes the image to Docker Hub.

**Trigger**: Manual workflow dispatch

**Process**:
1. Checkout repository
2. Set up QEMU for multi-architecture builds
3. Set up Docker Buildx
4. Login to Docker Hub
5. Build and push image with tags: `latest` and `1.0.0`

### Required Secrets

Configure these secrets in your GitHub repository:

- `DOCKERHUB_USER` - Docker Hub username
- `DOCKERHUB_PASSW` - Docker Hub password or access token

## 📦 Dependencies

### System Packages

- **Build tools**: gcc, g++, make, python3-dev
- **Image libraries**: libjpeg, libpng, libwebp, libtiff
- **Database**: libpq-dev, postgresql-client
- **XML/HTML**: libxml2, libxslt, libfontconfig
- **Fonts**: fonts-noto-cjk (CJK language support)
- **PDF**: poppler-utils, wkhtmltopdf
- **LDAP**: libldap2-dev, libsasl2-dev

### Python Packages

All packages from [Odoo 17 requirements.txt](https://github.com/odoo/odoo/blob/17.0/requirements.txt) plus:

- debugpy
- pre-commit
- gevent==21.8.0 (specific version to avoid build issues)

### Node.js Packages (Global)

- npm@10
- less
- prettier
- @prettier/plugin-xml
- prettier-plugin-sql

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

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
- Consider PostgreSQL 14+ for better performance

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