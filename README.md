# Platform Engineering Portfolio

[![Deploy MkDocs](https://github.com/mohammed-suhail-devops/mkdocs/actions/workflows/ci.yaml/badge.svg)](https://github.com/mohammed-suhail-devops/mkdocs/actions/workflows/ci.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Professional portfolio showcasing production platform engineering work, including Kubernetes platforms, Infrastructure as Code, GitOps workflows, and cloud architecture.

🌐 **Live Site**: [https://mohammed-suhail-devops.github.io/mkdocs/](https://mohammed-suhail-devops.github.io/mkdocs/)

## About This Portfolio

This site documents real production platform engineering work covering:

- **Azure Kubernetes Service (AKS)** platform design and operations
- **Terraform and Terragrunt** infrastructure automation
- **GitOps workflows** with Argo CD and Helm
- **Observability foundations** using Prometheus, Grafana, and Loki
- **Cloud networking** and security patterns
- **CI/CD pipelines** with GitHub Actions

All case studies are based on production systems with sensitive details removed for security and confidentiality.

## Key Features

✨ **Professional Design**
- Material for MkDocs theme with custom styling
- Dark/light mode support
- Responsive layout for all devices
- SEO optimized with Open Graph tags

📚 **Comprehensive Documentation**
- Detailed case studies with architecture diagrams
- Complete terraform module documentation
- Step-by-step implementation guides
- Production readiness checklists

🎯 **Career-Focused Content**
- About page with professional background
- Skills and certifications showcase
- Contact page with clear call-to-action
- LinkedIn and GitHub integration

🔍 **Discoverability**
- Google Analytics integration
- Social media cards
- Search functionality
- Mobile-friendly navigation

## Technology Stack

- **Static Site Generator**: [MkDocs](https://www.mkdocs.org/)
- **Theme**: [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- **Deployment**: GitHub Actions → GitHub Pages
- **Version Control**: Git/GitHub
- **Analytics**: Google Analytics (configurable)

## Local Development

### Prerequisites

- Python 3.x
- pip

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/mohammed-suhail-devops/mkdocs.git
   cd mkdocs
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Run local development server:
   ```bash
   mkdocs serve
   ```

4. Open your browser to `http://127.0.0.1:8000`

### Build Static Site

```bash
mkdocs build
```

The static site will be generated in the `site/` directory.

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yaml              # GitHub Actions deployment workflow
├── docs/                         # Source markdown files
│   ├── index.md                 # Homepage
│   ├── about.md                 # About page
│   ├── skills.md                # Skills & Certifications
│   ├── contact.md               # Contact page
│   ├── Azure/                   # Azure case studies & modules
│   ├── ingress-controller-migration/
│   └── stylesheets/
│       └── extra.css            # Custom CSS styling
├── site/                         # Built static site (auto-generated)
├── mkdocs.yaml                   # MkDocs configuration
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## Configuration

### Google Analytics

To enable Google Analytics tracking:

1. Create a Google Analytics 4 property
2. Get your Measurement ID (format: `G-XXXXXXXXXX`)
3. Update `mkdocs.yaml`:
   ```yaml
   extra:
     analytics:
       provider: google
       property: G-XXXXXXXXXX  # Replace with your ID
   ```

### Social Cards

Automatic social media preview cards are enabled. Customize in `mkdocs.yaml`:

```yaml
plugins:
  - social:
      cards: true
      cards_layout_options:
        background_color: "#009688"
        color: "#ffffff"
```

## Deployment

The site automatically deploys to GitHub Pages on push to `main` or `master` branch via GitHub Actions.

### Manual Deployment

```bash
mkdocs gh-deploy --force
```

## Content Guidelines

When adding new content:

1. **Use clear headers**: H1 for page title, H2-H3 for sections
2. **Include code examples**: Use fenced code blocks with language specification
3. **Add diagrams**: Place in relevant `assets/` directories
4. **Link internally**: Use relative paths for internal links
5. **Test locally**: Always preview changes with `mkdocs serve`

## Customization

### Custom CSS

Edit `docs/stylesheets/extra.css` to customize styling. Current custom components:

- Portfolio hero section
- Contact cards
- Skills competency grid
- Certification showcase
- About page layout

### Navigation

Update the `nav` section in `mkdocs.yaml` to modify site navigation structure.

## Contributing

This is a personal portfolio repository. However, if you find issues or have suggestions:

1. Open an issue describing the problem or suggestion
2. For typos or small fixes, feel free to submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contact

**Mohammed Suhail**  
Platform Engineer | Kubernetes | Azure | AWS

- 🌐 Website: [mohammed-suhail-devops.github.io/mkdocs](https://mohammed-suhail-devops.github.io/mkdocs/)
- 💼 LinkedIn: [mohammedsuhail-cloud](https://www.linkedin.com/in/mohammedsuhail-cloud/)
- 🐙 GitHub: [@mohammed-suhail-devops](https://github.com/mohammed-suhail-devops)
- 📧 Email: suhailakofficial@gmail.com
