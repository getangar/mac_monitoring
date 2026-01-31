## Description

<!-- Briefly describe your changes. What does this PR do? -->

## Type of Change

- [ ] 🐛 Bug fix (non-breaking change that fixes an issue)
- [ ] ✨ New feature (non-breaking change that adds functionality)
- [ ] 📊 Dashboard (new or updated Grafana dashboard)
- [ ] 📖 Documentation (updates to README, wiki, or comments)
- [ ] 🔧 Configuration (Telegraf, Prometheus, or Grafana configs)
- [ ] 🍎 Apple Silicon (new metrics or chip support)
- [ ] ♻️ Refactoring (no functional changes)

## Related Issue

<!-- Link to the issue this PR addresses. Use "Fixes #123" to auto-close when merged. -->

Fixes #

## Mac Environment Tested

<!-- Check the chip(s) you tested on -->

- [ ] M1 / M1 Pro / M1 Max / M1 Ultra
- [ ] M2 / M2 Pro / M2 Max / M2 Ultra
- [ ] M3 / M3 Pro / M3 Max
- [ ] M4 / M4 Pro / M4 Max
- [ ] Not applicable (server-side only)

**macOS Version:** <!-- e.g., Sonoma 14.2 -->

## Changes Made

<!-- List the main changes in this PR -->

- 
- 
- 

## How to Test

<!-- Provide steps to test your changes -->

1. Install/update Telegraf config on Mac
2. 
3. 
4. Verify metrics in Prometheus/Grafana

## Checklist

- [ ] Tested on Apple Silicon Mac
- [ ] Telegraf starts without errors
- [ ] Metrics are collected correctly (`curl localhost:9273/metrics`)
- [ ] Prometheus scrapes successfully
- [ ] Grafana displays data (if dashboard changes)
- [ ] Documentation updated (if applicable)
- [ ] No sensitive data committed (IPs, passwords, hostnames)

## Screenshots (if applicable)

<!-- Add screenshots for dashboard changes, new metrics, etc. -->

## Additional Notes

<!-- Any chip-specific notes, limitations, or context for reviewers -->

