# MyBlogs

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
This project is licensed under the MIT License.
