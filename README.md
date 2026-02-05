# OPNsense User Guide

A comprehensive, professionally-styled PDF guide for OPNsense firewall administration. Designed as a reference for both human administrators and LLM/AI agents.

![OPNsense Logo](assets/opnsense-logo.png)

## Overview

This guide covers OPNsense firewall configuration from initial setup through advanced topics like High Availability, IDS/IPS, and VPN configurations. It's generated programmatically using Python and ReportLab, producing a consistent, version-controlled document.

**Features:**
- 78 pages of comprehensive documentation
- Official OPNsense branding and color scheme
- Custom network diagrams and illustrations
- Quick reference appendices
- pf syntax guide
- REST API documentation

## Table of Contents

1. Introduction & Overview
2. Installation & Initial Setup
3. Dashboard & System Configuration
4. Interfaces & Networking
5. Firewall Rules & NAT
6. VPN (IPsec, OpenVPN, WireGuard)
7. DNS & Unbound
8. Intrusion Detection (Suricata)
9. Traffic Shaping & QoS
10. High Availability (CARP)
11. Additional Services
12. Troubleshooting
13. Best Practices
14. Backup & Recovery
15. Certificates & PKI

**Appendices:**
- A: Quick Reference (defaults, ports, CLI commands)
- B: pf Syntax Reference
- C: REST API Reference

## Quick Start

### Prerequisites

```bash
# Python 3.8+
pip install reportlab
```

### Generate the PDF

```bash
python3 src/opnsense_user_guide.py
```

Output: `output/OPNsense_User_Guide.pdf`

## Project Structure

```
opnsense-user-guide/
├── README.md
├── LICENSE
├── assets/
│   ├── opnsense-logo.png    # Official logo (600x177)
│   └── opnsense-logo.svg    # Source SVG
├── src/
│   └── opnsense_user_guide.py   # PDF generator
└── output/
    └── OPNsense_User_Guide.pdf  # Generated PDF
```

## Customization

### Colors

The guide uses official OPNsense branding. Colors are defined at the top of `opnsense_user_guide.py`:

```python
OPNSENSE_ORANGE = HexColor("#FF6900")  # Primary brand color
OPNSENSE_DARK = HexColor("#2C3E50")    # Headers, text
OPNSENSE_BLUE = HexColor("#3498DB")    # Links, highlights
OPNSENSE_GREEN = HexColor("#27AE60")   # Success, LAN
OPNSENSE_RED = HexColor("#E74C3C")     # Warnings, alerts
```

### Adding Content

Content is organized in the `build_content()` function. Each chapter follows this pattern:

```python
# Chapter heading
story.append(chapter_heading("Chapter Title"))

# Section
story.append(section_heading("Section Name"))
story.append(Paragraph("Content here...", styles["BodyText"]))

# Code blocks
story.append(code_block("configctl webgui restart"))

# Tips and warnings
story.append(tip_box("Pro tip content"))
story.append(warning_box("Warning content"))
```

## Use Cases

### For Administrators
- Quick reference during firewall configuration
- Training material for new team members
- Standardized documentation for compliance

### For LLM/AI Agents
- Structured reference for OPNsense-related queries
- Consistent formatting for information extraction
- Comprehensive coverage of CLI commands and API endpoints

## Related Projects

- [OPNsense MCP Server](https://github.com/vespo92/OPNsenseMCP) - MCP server for OPNsense API integration
- [OPNsense](https://opnsense.org/) - Official OPNsense project

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

For content corrections or additions, please open an issue first to discuss the changes.

---

*This guide is not officially affiliated with the OPNsense project. OPNsense is a registered trademark of Deciso B.V.*
