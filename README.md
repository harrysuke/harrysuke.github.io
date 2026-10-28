# harrysuke.github.io

Welcome to my personal website and blog repository! This site is built using **Jekyll** and is automatically hosted via **GitHub Pages**. 

The foundation of this theme is based on open-source work by Mark Otto and is released under the MIT License.

## 📁 Repository Structure

*   `_includes/` - Reusable HTML components (headers, footers, sidebars).
*   `_layouts/` - Page templates (e.g., default, post, page).
*   `_posts/` - Blog posts written in Markdown format.
*   `public/` - Static assets like CSS, JavaScript, and images.
*   `_config.yml` - Main configuration file for the Jekyll site settings.
*   `index.html` - The homepage entry point.

## 🚀 Getting Started

To run this website locally on your machine for development and testing, follow these steps:

### Prerequisites

Make sure you have Ruby and Bundler installed on your system.

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com
   cd harrysuke.github.io
   ```

2. Install the required dependencies:
   ```bash
   bundle install
   ```

3. Start the local Jekyll development server:
   ```bash
   bundle exec jekyll serve
   ```

4. Open your browser and navigate to `http://localhost:4000`.

## ✍️ Writing a New Post

To add a new blog post, create a markdown file inside the `_posts` directory using the following naming convention:

```text
YYYY-MM-DD-title-of-your-post.md
```

Include the required front matter at the top of the file:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +0000
categories: blog
---
Your content goes here...
```

## 📜 License

This project is open-source software licensed under the **MIT License**. See the `LICENSE.md` file for more details.
