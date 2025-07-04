# Zenn Content Management Project

## Project Overview

This is a **Zenn content management project** using the official Zenn CLI for creating and managing technical articles and books. Zenn is a Japanese technical content platform similar to Medium or Dev.to.

### Project Type
- **Content Management System**: Technical articles and books
- **Platform**: Zenn (Japanese technical content platform)
- **Language**: Japanese technical writing
- **Content Types**: Articles and Books (multi-chapter publications)

## Key Development Commands

### Core Zenn CLI Commands
```bash
# Preview content in browser (main development command)
npx zenn preview

# Create new article
npx zenn new:article

# Create new book
npx zenn new:book

# List all articles
npx zenn list:articles

# List all books
npx zenn list:books

# Show CLI version
npx zenn --version

# Show help
npx zenn --help
```

### Content Quality Commands
```bash
# Run textlint on articles (Japanese technical writing linting)
npx textlint articles/**

# Install dependencies
npm install

# No build process - content is managed via Zenn CLI
```

### Browser Sync (Available but not actively used)
```bash
# browser-sync is installed but no npm scripts are configured
```

## Project Structure

```
/Users/kenji/Projects/zenn/
├── README.md                 # Basic project info with Zenn links
├── package.json             # Dependencies and project config
├── .textlintrc             # Japanese technical writing linting rules
├── .gitignore              # Git ignore rules
├── articles/               # Individual technical articles
│   ├── *.md               # Article files (with frontmatter)
├── books/                  # Multi-chapter books
│   ├── */                 # Each book in its own directory
│   │   ├── config.yaml    # Book configuration
│   │   ├── *.md          # Chapter files
│   │   └── cover.png     # Optional book cover
├── images/                # Static assets for articles/books
│   ├── *.png, *.gif      # General images
│   └── books/            # Book-specific images
└── .github/
    └── workflows/
        └── reviewdog.yml  # Automated textlint review on PRs
```

## Content Architecture

### Articles Structure
- **Format**: Markdown with YAML frontmatter
- **Naming**: Random hash-based filenames (e.g., `03f09d74584666.md`)
- **Frontmatter**:
  ```yaml
  ---
  title: "Article Title"
  emoji: "💬"
  type: "tech"  # tech: technical article / idea: idea article
  topics: ["javascript", "react"]
  published: false  # true when ready to publish
  ---
  ```

### Books Structure
- **Format**: Multi-file structure with YAML config
- **Directory**: Each book has its own directory
- **Config** (`config.yaml`):
  ```yaml
  title: "Book Title"
  summary: "Book description"
  topics: ["figma", "frontend", "デザインシステム"]
  published: true
  price: 0  # 0 for free, number for paid
  chapters:
    - chapter1
    - chapter2
    # ...
  ```
- **Chapters**: Individual `.md` files for each chapter

## Important Patterns and Conventions

### Content Management
1. **Article Creation**: Use `npx zenn new:article` to generate proper filename structure
2. **Book Creation**: Use `npx zenn new:book` to generate proper directory structure
3. **Preview**: Always use `npx zenn preview` for local development
4. **Publishing**: Set `published: true` in frontmatter when ready

### Quality Assurance
1. **Textlint**: Japanese technical writing standards enforced
   - `preset-ja-technical-writing`: Technical writing rules
   - `preset-jtf-style`: Japanese typography standards
   - `preset-ja-spacing`: Proper spacing rules
2. **Automated Review**: GitHub Actions runs textlint on PRs
3. **Content Review**: Uses reviewdog for PR-based content review

### Asset Management
1. **Images**: Store in `/images/` directory
2. **Book Assets**: Use `/images/books/` for book-specific images
3. **Covers**: Book covers go in the book directory as `cover.png`

### Git Workflow
- **Branch Strategy**: Feature branches (currently on `add-astro-tech-book`)
- **Main Branch**: `main`
- **Review Process**: PRs with automated textlint checking
- **Status**: Clean working directory

## Development Workflow

### Starting Development
1. Run `npx zenn preview` to start local preview server
2. Navigate to the provided localhost URL
3. Make changes to articles/books
4. Preview updates in real-time

### Creating New Content
```bash
# For articles
npx zenn new:article
# Edit the generated .md file
# Set published: true when ready

# For books
npx zenn new:book
# Edit config.yaml for book metadata
# Create chapter files
# Set published: true when ready
```

### Quality Control
```bash
# Before committing, run textlint
npx textlint articles/**

# The CI will also run textlint on PRs
```

## Key Dependencies

### Production Dependencies
- **zenn-cli** (^0.1.144): Official Zenn CLI for content management
- **browser-sync** (^2.26.14): Development server (available but not used)

### Development Dependencies
- **textlint** (13.3.3): Japanese text linting
- **textlint-rule-preset-ja-spacing** (2.3.0): Japanese spacing rules
- **textlint-rule-preset-ja-technical-writing** (8.0.0): Technical writing rules
- **textlint-rule-preset-jtf-style** (2.3.13): Japanese typography standards

## Content Topics

Based on existing content, this project covers:
- **Frontend Development**: React, JavaScript, design systems
- **Accessibility**: Web accessibility (a11y) documentation
- **Design Systems**: Figma, UI/UX design
- **Product Design**: Design process and methodology
- **Technical Writing**: Japanese technical documentation

## Notes for Development

1. **Language**: Content is primarily in Japanese
2. **Platform**: Content is designed for publication on Zenn.dev
3. **Format**: Strict adherence to Zenn's markdown and frontmatter standards
4. **Quality**: High emphasis on Japanese technical writing quality via textlint
5. **Review**: Automated content review process via GitHub Actions
6. **Structure**: Well-organized content with clear separation between articles and books