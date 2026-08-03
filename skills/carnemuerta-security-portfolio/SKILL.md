---
name: carnemuerta-security-portfolio
description: Browser-based security portfolio and technical blog covering AI security, data protection, ML threats, and secure engineering practices
triggers:
  - "set up the 0xcarnemuerta security portfolio locally"
  - "how do I serve the security portfolio website"
  - "customize the carnemuerta portfolio content"
  - "deploy the security hub to GitHub Pages"
  - "modify the AI security blog posts"
  - "add new content to the security portfolio"
  - "configure the 0xcarnemuerta site navigation"
  - "troubleshoot the security portfolio website"
---

# 0xcarnemuerta Security Portfolio Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## What This Project Does

0xcarnemuerta Security Portfolio is a static HTML-based security portfolio and technical blog focused on AI security, data protection, machine-learning threats, and secure engineering. It serves as a browser-accessible showcase for security research and engineering work, covering topics like prompt injection, token authentication, ML threat surfaces, and secure data pipelines.

The project is designed to be served as a static website, either locally for development or via GitHub Pages for public access.

## Installation

Clone the repository to your local machine:

```bash
git clone https://github.com/reedjordanvrb4237/0xcarnemuerta-security-hub.git
cd 0xcarnemuerta-security-hub
```

No additional dependencies or package installations are required since this is a pure HTML/CSS/JS project.

## Serving the Portfolio Locally

### Using Python 3 (Built-in HTTP Server)

The simplest approach for local development:

```bash
# Serve on default port 8000
python3 -m http.server 8000

# Serve on a custom port
python3 -m http.server 3000
```

Then navigate to `http://localhost:8000/` in your browser.

### Using Node.js http-server

If you have Node.js installed:

```bash
# Install http-server globally (one-time)
npm install -g http-server

# Serve the current directory
http-server -p 8000

# With auto-reload on file changes
http-server -p 8000 -c-1
```

### Using PHP Built-in Server

```bash
php -S localhost:8000
```

### Using Live Server (VS Code Extension)

1. Install the "Live Server" extension in VS Code
2. Right-click on `index.html`
3. Select "Open with Live Server"

## Project Structure

Typical static portfolio structure:

```
0xcarnemuerta-security-hub/
├── index.html              # Main landing page
├── about.html             # About/bio page
├── portfolio.html         # Portfolio showcase
├── blog.html              # Blog listing page
├── posts/                 # Individual blog posts
│   ├── ai-security.html
│   ├── prompt-injection.html
│   └── token-auth.html
├── assets/
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   └── main.js
│   └── images/
└── LICENSE
```

## Customizing Content

### Modifying the Main Page

Edit `index.html` to update portfolio introduction:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>0xcarnemuerta Security Portfolio</title>
    <link rel="stylesheet" href="assets/css/styles.css">
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="index.html">Home</a></li>
                <li><a href="portfolio.html">Portfolio</a></li>
                <li><a href="blog.html">Blog</a></li>
                <li><a href="about.html">About</a></li>
            </ul>
        </nav>
    </header>
    
    <main>
        <section class="hero">
            <h1>0xcarnemuerta Security Portfolio</h1>
            <p>AI Security, Data Protection, and Secure Engineering</p>
        </section>
        
        <section class="featured-work">
            <h2>Featured Projects</h2>
            <!-- Add your security projects here -->
        </section>
    </main>
    
    <script src="assets/js/main.js"></script>
</body>
</html>
```

### Adding a New Blog Post

Create a new HTML file in the `posts/` directory:

```html
<!-- posts/ml-threat-modeling.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ML Threat Modeling - 0xcarnemuerta</title>
    <link rel="stylesheet" href="../assets/css/styles.css">
</head>
<body>
    <header>
        <nav>
            <a href="../index.html">← Back to Home</a>
        </nav>
    </header>
    
    <article>
        <h1>Machine Learning Threat Modeling</h1>
        <time datetime="2026-08-03">August 3, 2026</time>
        
        <section>
            <h2>Introduction</h2>
            <p>Understanding threat surfaces in ML systems...</p>
        </section>
        
        <section>
            <h2>Common Attack Vectors</h2>
            <ul>
                <li>Data poisoning</li>
                <li>Model extraction</li>
                <li>Adversarial examples</li>
                <li>Prompt injection</li>
            </ul>
        </section>
        
        <section>
            <h2>Mitigation Strategies</h2>
            <pre><code class="language-python">
# Example: Input validation for ML endpoints
def validate_input(user_input: str) -> bool:
    # Check for prompt injection patterns
    injection_patterns = [
        "ignore previous instructions",
        "disregard all above",
        "new instructions:"
    ]
    
    return not any(pattern in user_input.lower() 
                   for pattern in injection_patterns)
            </code></pre>
        </section>
    </article>
    
    <script src="../assets/js/main.js"></script>
</body>
</html>
```

### Updating Navigation

To add the new post to your blog listing (`blog.html`):

```html
<section class="blog-posts">
    <article class="post-preview">
        <h3><a href="posts/ml-threat-modeling.html">ML Threat Modeling</a></h3>
        <time datetime="2026-08-03">August 3, 2026</time>
        <p>Understanding threat surfaces in machine learning systems...</p>
    </article>
    
    <article class="post-preview">
        <h3><a href="posts/prompt-injection.html">Prompt Injection Attacks</a></h3>
        <time datetime="2026-07-30">July 30, 2026</time>
        <p>Exploring security risks in LLM-powered applications...</p>
    </article>
</section>
```

## Styling and Assets

### Custom CSS Example

Create or modify `assets/css/styles.css`:

```css
:root {
    --primary-color: #0a0e27;
    --accent-color: #00ff41;
    --text-color: #e0e0e0;
    --background: #0d1117;
}

body {
    font-family: 'Courier New', monospace;
    background-color: var(--background);
    color: var(--text-color);
    margin: 0;
    padding: 0;
    line-height: 1.6;
}

header nav {
    background: var(--primary-color);
    padding: 1rem;
}

header nav ul {
    list-style: none;
    display: flex;
    gap: 2rem;
    margin: 0;
    padding: 0;
}

header nav a {
    color: var(--accent-color);
    text-decoration: none;
    transition: opacity 0.3s;
}

header nav a:hover {
    opacity: 0.7;
}

.hero {
    text-align: center;
    padding: 4rem 2rem;
    border-bottom: 2px solid var(--accent-color);
}

pre code {
    display: block;
    background: var(--primary-color);
    padding: 1rem;
    overflow-x: auto;
    border-left: 3px solid var(--accent-color);
}
```

### JavaScript Enhancements

Add interactive features in `assets/js/main.js`:

```javascript
// Smooth scrolling for anchor links
document.querySelectorAll('a[href^="#"]').forEach(anchor => {
    anchor.addEventListener('click', function (e) {
        e.preventDefault();
        const target = document.querySelector(this.getAttribute('href'));
        if (target) {
            target.scrollIntoView({
                behavior: 'smooth',
                block: 'start'
            });
        }
    });
});

// Code syntax highlighting (if using a library)
document.addEventListener('DOMContentLoaded', function() {
    // Initialize syntax highlighting
    if (typeof hljs !== 'undefined') {
        hljs.highlightAll();
    }
});

// Dark mode toggle
const toggleDarkMode = () => {
    document.body.classList.toggle('light-mode');
    localStorage.setItem('theme', 
        document.body.classList.contains('light-mode') ? 'light' : 'dark'
    );
};

// Restore theme preference
if (localStorage.getItem('theme') === 'light') {
    document.body.classList.add('light-mode');
}
```

## Deploying to GitHub Pages

### Enable GitHub Pages

1. Push your repository to GitHub
2. Go to repository Settings → Pages
3. Select source branch (usually `main` or `gh-pages`)
4. Select folder (`/root` or `/docs`)
5. Save

Your site will be available at:
```
https://reedjordanvrb4237.github.io/0xcarnemuerta-security-hub/
```

### Custom Domain Setup

If you have a custom domain:

1. Create a `CNAME` file in your repository root:

```bash
echo "security.yourdomain.com" > CNAME
git add CNAME
git commit -m "Add custom domain"
git push
```

2. Configure DNS with your domain provider:
   - Add a CNAME record pointing to `reedjordanvrb4237.github.io`
   - Or add A records pointing to GitHub Pages IPs

## Common Patterns

### Security-Focused Content Structure

Organize security research posts with consistent structure:

```html
<article class="security-post">
    <header>
        <h1>Threat Name</h1>
        <div class="metadata">
            <span class="severity high">High Severity</span>
            <span class="category">AI Security</span>
            <time datetime="2026-08-03">August 3, 2026</time>
        </div>
    </header>
    
    <section class="threat-overview">
        <h2>Overview</h2>
        <!-- Threat description -->
    </section>
    
    <section class="attack-vectors">
        <h2>Attack Vectors</h2>
        <!-- How the attack works -->
    </section>
    
    <section class="code-examples">
        <h2>Proof of Concept</h2>
        <pre><code><!-- Demonstration code --></code></pre>
    </section>
    
    <section class="mitigations">
        <h2>Mitigations</h2>
        <!-- Defense strategies -->
    </section>
    
    <section class="references">
        <h2>References</h2>
        <!-- Links to research, CVEs, etc. -->
    </section>
</article>
```

### Portfolio Project Card

```html
<div class="portfolio-card">
    <h3>Secure ML Pipeline</h3>
    <div class="tags">
        <span class="tag">Python</span>
        <span class="tag">MLOps</span>
        <span class="tag">Security</span>
    </div>
    <p>End-to-end secure machine learning pipeline with data validation, 
       model authentication, and audit logging.</p>
    <div class="links">
        <a href="https://github.com/yourusername/project" target="_blank">
            View on GitHub
        </a>
        <a href="posts/ml-pipeline-security.html">Read More</a>
    </div>
</div>
```

## Troubleshooting

### Site Not Loading Locally

**Problem:** Opening `index.html` directly shows broken styles/scripts

**Solution:** Always use a web server, not `file://` protocol:

```bash
# Don't: file:///path/to/index.html
# Do:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Relative Path Issues

**Problem:** Navigation breaks when viewing different pages

**Solution:** Use relative paths consistently:

```html
<!-- In index.html -->
<link rel="stylesheet" href="assets/css/styles.css">

<!-- In posts/article.html -->
<link rel="stylesheet" href="../assets/css/styles.css">

<!-- Or use absolute paths from root -->
<link rel="stylesheet" href="/assets/css/styles.css">
```

### GitHub Pages 404 on Refresh

**Problem:** Refreshing sub-pages returns 404

**Solution:** Add a custom 404.html that redirects:

```html
<!-- 404.html -->
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="refresh" content="0; url=/0xcarnemuerta-security-hub/">
</head>
<body>
    <p>Redirecting...</p>
</body>
</html>
```

### Images Not Displaying

**Problem:** Images don't load after deployment

**Solution:** Check paths are correct for GitHub Pages subdirectory:

```html
<!-- Use relative paths -->
<img src="assets/images/diagram.png" alt="Architecture">

<!-- Or repository-relative -->
<img src="/0xcarnemuerta-security-hub/assets/images/diagram.png" alt="Architecture">
```

### CSS Not Updating

**Problem:** Changes to CSS not reflected in browser

**Solution:** Clear cache or use cache-busting:

```html
<link rel="stylesheet" href="assets/css/styles.css?v=2">
```

Or serve with proper headers during development:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
# Then hard refresh: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)
```

## Best Practices

1. **Keep HTML semantic**: Use proper heading hierarchy and semantic elements
2. **Optimize images**: Compress images before committing to reduce repository size
3. **Mobile-first design**: Ensure responsive design for all device sizes
4. **Accessibility**: Include alt text, ARIA labels, and proper contrast ratios
5. **Version assets**: Use query strings or versioned filenames for CSS/JS to bust cache
6. **Validate HTML**: Use W3C validator to catch markup errors
7. **Security headers**: If self-hosting, configure appropriate security headers
8. **Regular updates**: Keep content current and remove outdated security information
