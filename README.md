# TECHFOR by Suriya Sonphu
## Knowledge sharing blog about technology and life 
[![Deploy TECHFOR site to Pages](https://github.com/suriyasonp/myblogs/actions/workflows/gh-pages.yml/badge.svg?branch=main)](https://github.com/suriyasonp/myblogs/actions/workflows/gh-pages.yml)

This is a personal blog project built with [Hugo](https://gohugo.io/), using the Hugo Stack theme. It features posts, categories, tags, and custom layouts for sharing articles and resources.

## Features

- Fast static site generation
- Organized content by categories and tags
- Customizable layouts and themes
- Easy deployment to any static hosting provider

## Getting Started

### Prerequisites

- [Hugo](https://gohugo.io/getting-started/installing/) installed (extended version recommended)

### Installation

Clone the repository:

```bash
git clone https://github.com/suriyasonp/myblogs.git
cd myblogs
```

### Preview Locally

Start the Hugo development server:

```bash
hugo server -D
```

Visit [http://localhost:1313](http://localhost:1313) in your browser to preview the site. Use `Ctrl+C` to stop the server.

## Project Structure

- `content/` — Blog posts and pages
- `themes/` — Hugo theme files
- `config/` — Site configuration
- `static/` — Static assets (images, icons, etc.)
- `layouts/` — Custom templates and partials

## Creating a New Post

```bash
hugo new post/my-new-post/index.md
```

Edit the new file in `content/post/my-new-post/index.md` and add your content.

## Deployment

After building the site, deploy the `public/` folder to your preferred static hosting (e.g., GitHub Pages, Netlify, Vercel).

```bash
hugo
```

## Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you would like to change.

## License

### Code and Templates
The codebase, including Hugo templates, configurations, and theme files, is licensed under the MIT License. You are free to use, modify, and distribute the code as long as you include the original copyright notice.

### Content of Posts
The content of posts, including text, images, and other media, is licensed under the Creative Commons Attribution-NonCommercial-NoDerivatives (CC BY-NC-ND) license. You may share the content but cannot use it for commercial purposes or create derivative works.

For more details:
- MIT License: [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)
- CC BY-NC-ND License: [https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)

## Change History

### Post Additions

- **2025-12-05**: Added "Fail Fast: When 'Failure' is the Cheapest Cost of Building a Great Product" post.
- **2025-11-26**: Added "Skill Opens Doors, Relationships Carry You Far: Building Sustainable Success Together" post.
- **2025-11-23**: Added "Great Ideas Start with Your Customers" post.
- **2025-08-08**: Added "Fullstack Vue + Tailwind + .NET Modular Monolith" post.
- **2025-07-27**: Added "Modular Monolith: Architecture that combines the benefits of Monolith and Microservices" post.
- **2025-07-20**: Added "Transform Your Idea into a Hero App, Faster: A 3-Phase Journey" post.
- **2025-05-28**: Added "Markdown Syntax Guide" post.
- **2025-05-04**: Added "Mad Unicorn CliftonStrengths" post.
- **2025-05-04**: Added "The Human Algorithm" post.
- **2025-06-04**: Added "GitHub Copilot" post.
- **2025-06-04**: Added "Pride Month" post.
- **2025-07-11**: Added "WindRecorder Personal Memory Search" post.
- **2025-07-13**: Added "Future Skills 2030" post.
- **2025-07-13**: Added "Gemini CLI Developer Productivity" post.

### Bug Fixes

- **2025-07-13**: Fixed language persistence issue in the sidebar template.
- **2025-07-13**: Fixed English menu navigation issue by using `relLangURL` for multilingual URL handling.
- **2025-07-13**: Resolved template syntax errors in `left.html` causing Hugo server rebuild failures.
- **2025-07-13**: Updated image formats across multiple posts to use WebP for improved performance.
- **2025-07-13**: Removed 'Hello World' and 'How to Setup Hugo' posts.
- **2025-07-13**: Removed 'จิตกับธรรมชาติ เชื่อมโยงกันอย่างไร?' post.
- **2025-07-13**: Updated site language settings to Thai, including metadata and descriptions.
- **2025-07-13**: Updated site branding and removed 'The Human Algorithm' post.
