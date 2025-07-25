# Drink Blog - Jekyll Personal Blog

A personal blog built with Jekyll 4.x, featuring game development content, technical articles, and project showcases.

## 🚀 Quick Start

### Prerequisites

- **Ruby**: 3.2.0 or higher
- **Node.js**: 22.x (specified in package.json)
- **Bundler**: Latest version

### Local Development

1. **Install dependencies:**
   ```bash
   # Install Ruby gems
   bundle install
   
   # Install Node.js dependencies
   npm install
   ```

2. **Start development server:**
   ```bash
   # Standard serve with live reload
   npm run serve
   
   # Or serve with drafts
   npm run serve-drafts
   ```

3. **Build for production:**
   ```bash
   npm run build
   ```

## 📦 Deployment

### Vercel Deployment

This project is optimized for Vercel deployment with the following configurations:

- **Node.js Version**: 22.x (latest LTS)
- **Ruby Version**: 3.2.0
- **Build Command**: `bundle install && bundle exec jekyll build`
- **Output Directory**: `_site`

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/import/project?template=https://github.com/Drinkwang/jekyll2)

### Manual Deployment

```bash
# Build the site
npm run build

# Deploy the _site directory to your hosting provider
```

## 🛠️ Development Scripts

- `npm run serve` - Start development server with live reload
- `npm run serve-drafts` - Start server including draft posts
- `npm run build` - Build site for production
- `npm run clean` - Clean build artifacts
- `npm run install-deps` - Install Ruby dependencies
- `npm run update-deps` - Update Ruby dependencies

## 🔧 Configuration

### Jekyll Configuration

Key settings in `_config.yml`:
- **Markdown**: Kramdown with GFM input
- **Highlighter**: Rouge
- **Plugins**: Pagination, Feed, Sitemap, SEO
- **Pagination**: 6 posts per page

### Dependencies

**Ruby Gems:**
- Jekyll 4.3.x
- Minima theme 2.5.x
- Jekyll plugins (feed, sitemap, seo-tag, paginate)
- Webrick (Ruby 3.0+ compatibility)

**Node.js:**
- Grunt for asset processing
- Various build tools

## 🎮 Features

- **Game Showcase**: Dedicated section for game projects
- **Technical Blog**: Articles about Unity, Godot, and web development
- **Responsive Design**: Mobile-friendly layout
- **SEO Optimized**: Meta tags, sitemap, and structured data
- **Comments**: Gitalk integration
- **Analytics**: Google Analytics support

## 🐛 Troubleshooting

### Common Issues

1. **Ruby Version Conflicts:**
   ```bash
   # Check Ruby version
   ruby --version
   
   # Use rbenv or rvm to switch versions
   rbenv install 3.2.0
   rbenv local 3.2.0
   ```

2. **Bundle Install Errors:**
   ```bash
   # Clear bundle cache
   bundle clean --force
   
   # Reinstall gems
   bundle install
   ```

3. **Node.js Version Issues:**
   ```bash
   # Check Node version
   node --version
   
   # Use nvm to switch versions
   nvm install 22
   nvm use 22
   ```

### Deployment Issues

- **Vercel Build Failures**: Ensure Node.js 22.x is specified in package.json engines
- **Ruby Dependency Conflicts**: Delete Gemfile.lock and run `bundle install`
- **Asset Loading Issues**: Check baseurl configuration in _config.yml

## 📝 Content Management

### Adding Posts

Create new posts in `_posts/` directory with format:
```
YYYY-MM-DD-title.md
```

### Adding Games

Update the `games` array in `_config.yml` with new game entries.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally
5. Submit a pull request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 🔗 Links

- **Live Site**: [drinker.site](http://drinker.site)
- **GitHub**: [Drinkwang/jekyll2](https://github.com/Drinkwang/jekyll2)
- **Author**: [Drinkwang](https://github.com/Drinkwang)