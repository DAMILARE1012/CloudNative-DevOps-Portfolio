# virtualenvwrapper Guide: Global Virtual Environment Management

A comprehensive guide to setting up and using virtualenvwrapper for managing global Python virtual environments on Linux systems.

## Table of Contents
- [Installation](#installation)
- [Configuration](#configuration)
- [Basic Usage](#basic-usage)
- [Command Reference](#command-reference)
- [Workflows](#workflows)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Installation

### Prerequisites
- Python 3.x installed
- Ubuntu/Debian Linux system

### Install virtualenvwrapper
```bash
sudo apt update
sudo apt install python3-virtualenvwrapper
```

## Configuration

### 1. Configure Shell Environment
Add virtualenvwrapper configuration to your `.bashrc`:

```bash
# Remove any existing conflicting lines
sed -i '/virtualenvwrapper/d' ~/.bashrc
sed -i '/WORKON_HOME/d' ~/.bashrc
sed -i '/VIRTUALENVWRAPPER_PYTHON/d' ~/.bashrc

# Add virtualenvwrapper configuration
echo 'export WORKON_HOME=~/.virtualenvs' >> ~/.bashrc
echo 'export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3' >> ~/.bashrc
echo 'source /usr/share/virtualenvwrapper/virtualenvwrapper.sh' >> ~/.bashrc
```

### 2. Reload Configuration
```bash
source ~/.bashrc
```

### 3. Verify Installation
```bash
# Test if virtualenvwrapper is working
virtualenvwrapper
```

## Basic Usage

### Create Your First Environment
```bash
# Create and activate a new virtual environment
mkvirtualenv myproject
# You'll see (myproject) in your prompt

# Install packages
pip install requests django flask

# Save dependencies
pip freeze > requirements.txt

# Deactivate when done
deactivate
```

### Daily Workflow
```bash
# Activate environment (from any directory)
workon myproject

# Work on your project
cd /path/to/your/project
python app.py

# Deactivate when finished
deactivate
```

## Command Reference

### Environment Management
| Command | Description |
|---------|-------------|
| `mkvirtualenv <name>` | Create and activate new environment |
| `mkvirtualenv -p python3.11 <name>` | Create with specific Python version |
| `mkvirtualenv -r requirements.txt <name>` | Create and install from requirements |
| `workon <name>` | Activate environment |
| `workon` | Show interactive list of environments |
| `deactivate` | Deactivate current environment |
| `rmvirtualenv <name>` | Delete environment permanently |
| `cpvirtualenv <source> <target>` | Copy environment |

### Information & Navigation
| Command | Description |
|---------|-------------|
| `lsvirtualenv` | List all environments (detailed) |
| `lsvirtualenv -b` | List all environments (brief) |
| `showvirtualenv <name>` | Show environment details |
| `lssitepackages` | List installed packages in active env |
| `cdvirtualenv` | Go to active environment directory |
| `cdvirtualenv bin` | Go to bin/ folder |
| `cdsitepackages` | Go to site-packages folder |

### Project Integration
| Command | Description |
|---------|-------------|
| `setvirtualenvproject` | Link current directory to active environment |
| `mktmpenv` | Create temporary environment (auto-deleted) |

## Workflows

### Workflow 1: New Project Setup
```bash
# 1. Create environment
mkvirtualenv django-blog

# 2. Install dependencies
pip install django psycopg2-binary pillow

# 3. Create project
django-admin startproject blog .

# 4. Save dependencies
pip freeze > requirements.txt

# 5. Link project directory (optional)
setvirtualenvproject

# 6. Deactivate
deactivate
```

### Workflow 2: Daily Development
```bash
# 1. Activate environment
workon django-blog
# If linked with setvirtualenvproject, automatically goes to project directory

# 2. Work on project
python manage.py runserver

# 3. Install new packages as needed
pip install django-rest-framework
pip freeze > requirements.txt

# 4. Deactivate when done
deactivate
```

### Workflow 3: Team Collaboration
```bash
# Team member receives requirements.txt
# Create identical environment
mkvirtualenv -r requirements.txt team-project

# Or recreate existing environment
rmvirtualenv team-project
mkvirtualenv team-project
pip install -r requirements.txt
```

### Workflow 4: Environment Maintenance
```bash
# List all environments
lsvirtualenv -b

# Check installed packages
workon myproject
lssitepackages

# Remove unused environment
rmvirtualenv old-project

# Copy environment for testing
cpvirtualenv production-app testing-app
workon testing-app
pip install pytest
```

## Best Practices

### 1. Naming Conventions
```bash
# Use descriptive names
mkvirtualenv django-blog
mkvirtualenv api-backend  
mkvirtualenv data-analysis
mkvirtualenv client-webapp
```

### 2. Requirements Management
```bash
# Always maintain requirements.txt
workon myproject
pip freeze > requirements.txt

# Pin important versions
echo "django==4.2.0" >> requirements.txt
echo "requests>=2.28.0" >> requirements.txt
```

### 3. Project Structure
```
~/.virtualenvs/          # All virtual environments
├── django-blog/
├── api-backend/
└── data-analysis/

~/projects/              # Your actual code
├── django-blog/
├── api-backend/
└── data-analysis/
```

### 4. Environment Variables (Advanced)
```bash
# Set environment-specific variables
workon myproject
echo 'export DATABASE_URL="postgresql://localhost/mydb"' >> $VIRTUAL_ENV/bin/postactivate
echo 'export DEBUG=True' >> $VIRTUAL_ENV/bin/postactivate
echo 'unset DATABASE_URL DEBUG' >> $VIRTUAL_ENV/bin/predeactivate
```

## Troubleshooting

### Common Issues

#### 1. "virtualenvwrapper module not found"
```bash
# Ensure correct Python path
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source ~/.bashrc
```

#### 2. "Command not found: mkvirtualenv"
```bash
# Check if virtualenvwrapper is sourced
grep virtualenvwrapper ~/.bashrc

# Reinstall if necessary
sudo apt install python3-virtualenvwrapper
```

#### 3. "Permission denied" errors
```bash
# Check WORKON_HOME permissions
ls -la ~/.virtualenvs/
chmod 755 ~/.virtualenvs/
```

#### 4. Environments not showing up
```bash
# Check WORKON_HOME variable
echo $WORKON_HOME

# Should output: /home/username/.virtualenvs
```

### Verification Commands
```bash
# Check installation
which virtualenvwrapper.sh

# Check Python path
echo $VIRTUALENVWRAPPER_PYTHON

# Check environments directory
echo $WORKON_HOME
ls -la $WORKON_HOME
```

## Quick Reference Card

| Task | Command |
|------|---------|
| Create environment | `mkvirtualenv <name>` |
| Activate environment | `workon <name>` |
| Deactivate environment | `deactivate` |
| List environments | `lsvirtualenv -b` |
| Delete environment | `rmvirtualenv <name>` |
| Link to project | `setvirtualenvproject` |
| Show environment info | `showvirtualenv <name>` |
| List packages | `lssitepackages` |

## Key Benefits

✅ **Global Access** - Work from any directory  
✅ **System-wide** - Available to all users  
✅ **Persistent** - Survives system reboots  
✅ **Professional** - Industry-standard tool  
✅ **Easy Management** - Simple commands  
✅ **Project Integration** - Link environments to projects  

---

**Author**: Generated for DevOps Cloud Native Hands-On  
**Date**: July 2025  
**Version**: 1.0  

For more information, visit the [virtualenvwrapper documentation](https://virtualenvwrapper.readthedocs.io/).
