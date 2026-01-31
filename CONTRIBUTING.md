# Contributing to Mac Monitoring

Thank you for your interest in contributing to the Mac Monitoring project! 🎉

This project helps people monitor their Apple Silicon Macs with Prometheus and Grafana. Your contributions make it better for the entire community.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Pull Request Process](#pull-request-process)
- [Style Guidelines](#style-guidelines)
- [Community](#community)

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code.

## How Can I Contribute?

### 🐛 Reporting Bugs

Before creating a bug report, please check existing issues.

When reporting a bug, include:

- **Clear title** describing the issue
- **Steps to reproduce** the behavior
- **Expected behavior** vs actual behavior
- **Environment details:**
  - Mac model and chip (e.g., MacBook Pro M3 Pro)
  - macOS version
  - Telegraf version (`telegraf --version`)
  - mactop version (`mactop --version`)
  - Docker version on server
- **Logs** if applicable
- **Screenshots** if helpful

### 💡 Suggesting Features

Feature suggestions are welcome! Please:

- Check existing issues/discussions first
- Describe the feature and its use case
- Explain why it would benefit the project
- Consider Apple Silicon-specific features

### 📖 Improving Documentation

Documentation improvements are always appreciated:

- Fix typos or unclear explanations
- Add examples for different Mac models
- Improve wiki pages
- Add Grafana dashboard examples
- Document new metrics

### 🔧 Submitting Code Changes

We welcome contributions for:

- New metrics exporters
- Grafana dashboard improvements
- Telegraf configuration examples
- Bug fixes
- Performance improvements

## Getting Started

### Prerequisites

**On Mac:**
- Apple Silicon Mac (M1/M2/M3/M4)
- Homebrew
- Telegraf
- mactop (optional)

**On Server:**
- Docker and Docker Compose
- Git

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/mac_monitoring.git
   cd mac_monitoring
   ```
3. Add upstream remote:
   ```bash
   git remote add upstream https://github.com/getangar/mac_monitoring.git
   ```

### Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/issue-description
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `dashboard/` - Grafana dashboard changes

## Development Setup

### Mac Setup

```bash
# Install Telegraf
brew install telegraf

# Install mactop
brew install mactop

# Start Telegraf
brew services start telegraf

# Verify metrics
curl http://localhost:9273/metrics | head -10
```

### Server Setup

```bash
# Start monitoring stack
docker-compose up -d

# Verify Prometheus
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[].health'

# Access Grafana
open http://localhost:3000
```

### Testing Changes

1. Make your changes
2. Verify Telegraf config: `telegraf --config telegraf.conf --test`
3. Check Prometheus scrapes successfully
4. Verify Grafana displays data correctly

## Pull Request Process

### Before Submitting

- [ ] Test your changes on an Apple Silicon Mac
- [ ] Verify metrics are collected correctly
- [ ] Update documentation if needed
- [ ] Test with Prometheus and Grafana
- [ ] Ensure no sensitive data is committed

### Submitting

1. Push your branch:
   ```bash
   git push origin feature/your-feature-name
   ```

2. Open a Pull Request on GitHub

3. Fill out the PR description with:
   - Description of changes
   - Mac model tested on
   - Related issue (if any)
   - Screenshots (for dashboard changes)

### PR Review

- PRs require review before merging
- Address review feedback promptly
- Keep PRs focused — one feature/fix per PR

## Style Guidelines

### Git Commit Messages

- Use present tense: "Add metric" not "Added metric"
- Use imperative mood: "Fix bug" not "Fixes bug"
- Keep first line under 72 characters
- Reference issues: "Fix #123"

Examples:
```
Add GPU temperature metric for M3 chips

Fix Telegraf config for macOS Sonoma

Update Grafana dashboard with memory pressure panel

Closes #42
```

### Configuration Files

#### Telegraf (TOML)

```toml
# Use consistent indentation (2 spaces)
# Comment each section
# Group related inputs

# CPU metrics
[[inputs.cpu]]
  percpu = true
  totalcpu = true
```

#### Prometheus (YAML)

```yaml
# Use 2-space indentation
# Add comments for non-obvious settings
scrape_configs:
  - job_name: 'mac-telegraf'
    static_configs:
      - targets: ['192.168.1.10:9273']
```

#### Grafana Dashboards (JSON)

- Use descriptive panel titles
- Include units in panel configuration
- Add thresholds where appropriate
- Test on different time ranges

### Documentation

- Write clear, concise explanations
- Include examples for different Mac models
- Specify which chips support which metrics
- Keep README focused, details in wiki

## Project Structure

```
mac_monitoring/
├── README.md                 # Main documentation
├── docker-compose.yml        # Server stack
├── prometheus.yml            # Prometheus config
├── alerts.yml               # Alert rules
├── telegraf/
│   └── telegraf.conf        # Mac agent config
├── grafana/
│   └── dashboards/          # Dashboard JSON files
└── scripts/
    └── mactop-exporter.sh   # Apple Silicon exporter
```

## Testing

### Checklist

- [ ] Telegraf starts without errors
- [ ] Metrics available at `:9273/metrics`
- [ ] Prometheus scrapes successfully
- [ ] Grafana displays data
- [ ] Works on target Mac chip (M1/M2/M3/M4)

### Testing Different Chips

If you have access to multiple Mac models, test on:
- [ ] M1 / M1 Pro / M1 Max
- [ ] M2 / M2 Pro / M2 Max
- [ ] M3 / M3 Pro / M3 Max
- [ ] M4 / M4 Pro

Note any chip-specific differences in your PR.

## Community

### Getting Help

- 💬 [GitHub Discussions](https://github.com/getangar/mac_monitoring/discussions)
- 🐛 [GitHub Issues](https://github.com/getangar/mac_monitoring/issues)
- 📖 [Wiki](https://github.com/getangar/mac_monitoring/wiki)

### Recognition

Contributors are recognized in:
- GitHub contributors page
- Release notes (for significant contributions)

---

Thank you for contributing! 🙏
