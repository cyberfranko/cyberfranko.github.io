# CyberFranko Wiki

A comprehensive cybersecurity knowledge base featuring playbooks, programming notes, defensive security materials, and tech projects.

🔗 **Live Site**: [https://cyberfranko.github.io](https://cyberfranko.github.io)

## 📚 Contents

- **Cybersecurity**: Incident response playbooks, defensive security guides, and security tools
- **Programming**: Python, PowerShell, and Bash scripting for security automation
- **Projects**: Security labs, CTF writeups, and custom tools

## 🚀 Quick Start

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/cyberfranko/cyberfranko.github.io.git
   cd cyberfranko.github.io
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run local server**
   ```bash
   mkdocs serve
   ```

4. **View in browser**
   Open [http://127.0.0.1:8000](http://127.0.0.1:8000)

## 📝 Adding Content

1. **Create new pages** in the `docs/` directory
2. **Update navigation** in `mkdocs.yml`
3. **Commit and push** - GitHub Actions will automatically deploy!

### Directory Structure

```
docs/
├── index.md                    # Home page
├── cybersecurity/
│   ├── playbooks.md           # Incident response playbooks
│   ├── defensive.md           # Blue team tactics
│   └── tools.md               # Security tools
├── programming/
│   ├── python.md              # Python scripts & notes
│   ├── powershell.md          # PowerShell scripts
│   └── bash.md                # Bash scripting
└── projects/
    └── index.md               # Project documentation
```

## 🛠️ Built With

- [MkDocs](https://www.mkdocs.org/) - Static site generator
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) - Beautiful theme
- [GitHub Pages](https://pages.github.com/) - Hosting
- [GitHub Actions](https://github.com/features/actions) - CI/CD

## 📦 MkDocs Commands

| Command | Description |
|---------|-------------|
| `mkdocs serve` | Start local development server |
| `mkdocs build` | Build static site |
| `mkdocs gh-deploy` | Manual deploy to GitHub Pages |

## 🔄 Deployment

The site automatically deploys to GitHub Pages when you push to the `main` branch. The GitHub Actions workflow:

1. Checks out the code
2. Installs dependencies
3. Builds the MkDocs site
4. Deploys to GitHub Pages

## 📄 License

This project is open source and available for educational purposes.

## 🤝 Contributing

Feel free to submit issues or pull requests to improve the content!

## 📧 Contact

For questions or suggestions, please open an issue on GitHub.

---

**Note**: This is a living document and will be continuously updated with new content and improvements.

