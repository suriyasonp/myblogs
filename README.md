# TECHFOR by Suriya Sonphu

A bilingual (Thai/English) personal blog built with Hugo, focusing on technology and life experiences.

## 🌟 Features

- **Bilingual Support**: Thai (default) and English content
- **Hugo Stack Theme**: Modern, responsive design
- **SEO Optimized**: Meta tags, Open Graph, Twitter Cards
- **Google Analytics**: Integrated analytics tracking
- **Auto-deployment**: GitHub Actions for seamless publishing
- **RSS/JSON Feeds**: Multiple output formats
- **Search Functionality**: Built-in search capabilities
- **Categories & Tags**: Organized content taxonomy

## 🚀 Quick Start

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (v0.147.5 or later)
- [Git](https://git-scm.com/)

### Local Development

1. **Clone the repository**

   ```bash
   git clone <your-repo-url>
   cd myblogs
   ```

2. **Initialize Hugo theme submodule** (if not already done)

   ```bash
   git submodule update --init --recursive
   ```

3. **Run the development server**

   ```bash
   hugo server -D
   ```

   The site will be available at `http://localhost:1313`

4. **Build for production**

   ```bash
   hugo --gc --minify
   ```

### Development Commands

- **Start server with drafts**: `hugo server -D`
- **Start server on all interfaces**: `hugo server -D --bind 0.0.0.0`
- **Create new post**: `hugo new post/my-new-post/index.md`
- **Create new page**: `hugo new page/my-page/index.md`

## 📁 Project Structure

```text
myblogs/
├── archetypes/          # Content templates
├── assets/              # Asset files (JS, CSS, images)
├── config/              # Configuration files
│   └── _default/
│       └── menu.toml    # Menu configurations
├── content/             # Blog content
│   ├── categories/      # Category pages
│   ├── page/           # Static pages (about, search, etc.)
│   └── post/           # Blog posts
├── data/               # Data files
├── i18n/               # Internationalization files
├── layouts/            # Custom templates
├── public/             # Generated site (git ignored)
├── resources/          # Hugo resources cache
├── static/             # Static assets
├── themes/             # Hugo themes
│   └── hugo-theme-stack/
├── hugo.toml           # Main configuration
└── README.md
```

## ⚙️ Configuration

The main configuration is in `hugo.toml`:

- **Base URL**: `https://suriyasonphu.com/`
- **Default Language**: Thai (`th`)
- **Theme**: hugo-theme-stack
- **Google Analytics**: Configured
- **Social**: Twitter integration

### Bilingual Setup

The site supports both Thai and English:

- Thai: Default language (`/`)
- English: Available at `/en/`

## 🚀 Deployment

### GitHub Actions (Automatic)

The site automatically deploys to GitHub Pages when you push to the `main` branch.

#### Workflow Features

- **Hugo Version**: 0.147.5 Extended
- **Build Environment**: Ubuntu Latest
- **Deployment Target**: GitHub Pages
- **Trigger**: Push to main branch or manual dispatch

#### Setup GitHub Pages

1. Go to your repository settings
2. Navigate to "Pages" section
3. Set source to "GitHub Actions"
4. The workflow will handle the rest

### Manual Deployment

If you prefer manual deployment:

1. **Build the site**

   ```bash
   hugo --gc --minify
   ```

2. **Deploy the `public/` folder** to your hosting provider

## 📝 Content Management

### Creating New Posts

```bash
# Create a new blog post
hugo new post/my-post-title/index.md

# Create a new post in Thai
hugo new post/my-thai-post/index.th.md

# Create a new post in English
hugo new post/my-english-post/index.en.md
```

### Front Matter Template

```yaml
---
title: "Your Post Title"
date: 2025-01-15T10:00:00+07:00
draft: false
categories: ["Technology"]
tags: ["hugo", "blog", "tutorial"]
description: "Brief description of your post"
---

Your content here...
```

### Multilingual Content

For bilingual posts, create both versions:

- `content/post/my-post/index.th.md` (Thai)
- `content/post/my-post/index.en.md` (English)

## 🎨 Customization

### Theme Configuration

The Hugo Stack theme is configured in `hugo.toml` with:

- Custom avatar and sidebar settings
- Color scheme toggle
- Widget configuration
- Social media integration

### Custom Layouts

Add custom templates in the `layouts/` directory to override theme defaults.

### Assets

- Favicons: `static/assets/favicon/`
- Images: `static/assets/images/`
- Custom CSS: `assets/scss/`

## 🔧 Development Tips

1. **Live Reload**: Hugo server automatically reloads on file changes
2. **Draft Posts**: Use `draft: true` in front matter and `-D` flag to preview
3. **Future Posts**: Use future dates and `--buildFuture` to preview
4. **Performance**: Use `hugo --gc --minify` for optimized builds

## 📊 Analytics & SEO

- **Google Analytics**: G-REDTWM8K0W
- **SEO**: Configured meta tags, Open Graph, Twitter Cards
- **Sitemap**: Auto-generated XML sitemap
- **RSS**: Available feeds for all sections

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with `hugo server -D`
5. Submit a pull request

## 📄 License

This project is open source. Please check the license file for details.

## 🆘 Troubleshooting

### Common Issues

1. **Theme not loading**: Ensure submodules are initialized

   ```bash
   git submodule update --init --recursive
   ```

2. **Build failures**: Check Hugo version compatibility

   ```bash
   hugo version
   ```

3. **Missing content**: Verify content files have correct front matter

### Support

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Stack Theme](https://github.com/CaiJimmy/hugo-theme-stack)
- [Hugo Community](https://discourse.gohugo.io/)

---

**Author**: Suriya Sonphu  
**Website**: [suriyasonphu.com](https://suriyasonphu.com/)  