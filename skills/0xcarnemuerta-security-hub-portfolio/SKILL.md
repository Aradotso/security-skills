---
name: 0xcarnemuerta-security-hub-portfolio
description: Browser-based security portfolio and technical blog covering AI security, data protection, ML threats, and secure engineering practices
triggers:
  - "set up the 0xcarnemuerta security portfolio locally"
  - "how do I customize the security hub portfolio"
  - "serve the security portfolio website"
  - "modify the AI security blog content"
  - "configure the 0xcarnemuerta portfolio site"
  - "add new security articles to the portfolio"
  - "deploy the security hub to GitHub Pages"
  - "update security blog navigation"
---

# 0xcarnemuerta Security Hub Portfolio

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

The 0xcarnemuerta Security Hub is a static HTML/CSS/JavaScript portfolio website focused on security research, data protection, AI/ML security threats, and secure engineering practices. It serves as both a personal portfolio and a technical blog platform for sharing security insights, particularly around prompt injection, token authentication, and AI-specific attack surfaces.

This is a web-based project with no backend dependencies—just static files served through a web server or GitHub Pages.

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/reedjordanvrb4237/0xcarnemuerta-security-hub.git
cd 0xcarnemuerta-security-hub
```

## Local Development

### Serving the Site Locally

Use Python's built-in HTTP server (most common):

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/` in your browser.

Alternative with Node.js `http-server`:

```bash
npx http-server -p 8000
```

Alternative with PHP:

```bash
php -S localhost:8000
```

### Directory Structure

```
0xcarnemuerta-security-hub/
├── index.html              # Main landing page
├── portfolio/              # Portfolio projects
├── blog/                   # Technical blog posts
├── assets/                 # Images, CSS, JS
│   ├── css/
│   ├── js/
│   └── images/
├── LICENSE
└── README.md
```

## Key Usage Patterns

### Adding a New Blog Post

Create a new HTML file in the `blog/` directory:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prompt Injection Vulnerabilities in LLM Applications - 0xcarnemuerta</title>
    <link rel="stylesheet" href="../assets/css/style.css">
</head>
<body>
    <header>
        <nav>
            <a href="../index.html">Home</a>
            <a href="../portfolio/">Portfolio</a>
            <a href="../blog/">Blog</a>
        </nav>
    </header>
    
    <main>
        <article>
            <h1>Prompt Injection Vulnerabilities in LLM Applications</h1>
            <time datetime="2026-08-03">August 3, 2026</time>
            
            <section>
                <h2>Introduction</h2>
                <p>Prompt injection represents a critical security concern...</p>
            </section>
            
            <section>
                <h2>Attack Vectors</h2>
                <pre><code class="language-python">
# Example vulnerable code
user_input = request.form['query']
prompt = f"Translate the following to Spanish: {user_input}"
response = llm.complete(prompt)
                </code></pre>
            </section>
            
            <section>
                <h2>Mitigation Strategies</h2>
                <ul>
                    <li>Input sanitization and validation</li>
                    <li>Prompt templates with strict boundaries</li>
                    <li>Output filtering and monitoring</li>
                </ul>
            </section>
        </article>
    </main>
    
    <footer>
        <p>&copy; 2026 0xcarnemuerta Security Hub</p>
    </footer>
    
    <script src="../assets/js/main.js"></script>
</body>
</html>
```

### Adding a Portfolio Project

Create a portfolio entry showcasing security work:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Secure ML Pipeline - 0xcarnemuerta Portfolio</title>
    <link rel="stylesheet" href="../assets/css/style.css">
</head>
<body>
    <header>
        <nav>
            <a href="../index.html">Home</a>
            <a href="../portfolio/">Portfolio</a>
            <a href="../blog/">Blog</a>
        </nav>
    </header>
    
    <main>
        <div class="portfolio-item">
            <h1>Secure ML Pipeline Architecture</h1>
            
            <div class="project-meta">
                <span class="tech-stack">Python | TensorFlow | Kubernetes</span>
                <span class="date">2026</span>
            </div>
            
            <section class="project-overview">
                <h2>Overview</h2>
                <p>End-to-end secure machine learning pipeline implementing data validation, 
                model versioning, and inference monitoring...</p>
            </section>
            
            <section class="technical-details">
                <h2>Security Features</h2>
                <ul>
                    <li>Encrypted data storage and transmission</li>
                    <li>Model signature verification</li>
                    <li>Access control with JWT tokens</li>
                    <li>Audit logging for all predictions</li>
                </ul>
            </section>
            
            <section class="code-sample">
                <h2>Implementation</h2>
                <pre><code class="language-python">
import os
from cryptography.fernet import Fernet

class SecureModelLoader:
    def __init__(self):
        self.encryption_key = os.getenv('MODEL_ENCRYPTION_KEY')
        self.cipher = Fernet(self.encryption_key.encode())
    
    def load_model(self, model_path, signature):
        """Load and verify encrypted model"""
        with open(model_path, 'rb') as f:
            encrypted_data = f.read()
        
        # Verify signature before decryption
        if not self.verify_signature(encrypted_data, signature):
            raise SecurityError("Model signature verification failed")
        
        decrypted_model = self.cipher.decrypt(encrypted_data)
        return pickle.loads(decrypted_model)
                </code></pre>
            </section>
            
            <div class="project-links">
                <a href="https://github.com/reedjordanvrb4237/secure-ml-pipeline" 
                   class="btn">View on GitHub</a>
            </div>
        </div>
    </main>
</body>
</html>
```

### Customizing Styling

Edit `assets/css/style.css` to modify the site appearance:

```css
/* Security-focused dark theme example */
:root {
    --primary-bg: #0d1117;
    --secondary-bg: #161b22;
    --accent-color: #58a6ff;
    --danger-color: #f85149;
    --success-color: #3fb950;
    --text-primary: #c9d1d9;
    --text-secondary: #8b949e;
}

body {
    background-color: var(--primary-bg);
    color: var(--text-primary);
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    line-height: 1.6;
    margin: 0;
    padding: 0;
}

header {
    background-color: var(--secondary-bg);
    padding: 1rem 2rem;
    border-bottom: 1px solid var(--accent-color);
}

nav a {
    color: var(--text-primary);
    text-decoration: none;
    margin-right: 2rem;
    transition: color 0.2s;
}

nav a:hover {
    color: var(--accent-color);
}

pre code {
    background-color: var(--secondary-bg);
    border-left: 3px solid var(--accent-color);
    display: block;
    padding: 1rem;
    overflow-x: auto;
}

.vulnerability {
    border-left: 3px solid var(--danger-color);
    padding-left: 1rem;
    margin: 1rem 0;
}

.mitigation {
    border-left: 3px solid var(--success-color);
    padding-left: 1rem;
    margin: 1rem 0;
}
```

## Configuration

### GitHub Pages Deployment

1. Go to repository Settings → Pages
2. Set Source to "Deploy from a branch"
3. Select branch: `main`, folder: `/ (root)`
4. Save and wait for deployment

The site will be available at: `https://reedjordanvrb4237.github.io/0xcarnemuerta-security-hub/`

### Custom Domain (Optional)

Add a `CNAME` file to the repository root:

```
security.yourdomain.com
```

Then configure your DNS provider:

```
Type: CNAME
Host: security
Value: reedjordanvrb4237.github.io
```

### Navigation Updates

Update navigation links across all pages when adding new sections. Create a shared navigation component in `assets/js/main.js`:

```javascript
// Dynamic navigation injection
document.addEventListener('DOMContentLoaded', function() {
    const navHTML = `
        <nav class="main-nav">
            <a href="/index.html">Home</a>
            <a href="/portfolio/">Portfolio</a>
            <a href="/blog/">Blog</a>
            <a href="/resources/">Resources</a>
            <a href="/contact/">Contact</a>
        </nav>
    `;
    
    const header = document.querySelector('header');
    if (header && !header.querySelector('nav')) {
        header.innerHTML = navHTML + header.innerHTML;
    }
});
```

## Common Patterns

### Security Blog Post Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Security analysis and technical discussion">
    <title>Article Title - 0xcarnemuerta</title>
    <link rel="stylesheet" href="../assets/css/style.css">
    <link rel="stylesheet" href="../assets/css/prism.css">
</head>
<body>
    <header></header>
    
    <main class="blog-post">
        <article>
            <header class="post-header">
                <h1>Article Title</h1>
                <div class="post-meta">
                    <time datetime="2026-08-03">August 3, 2026</time>
                    <span class="tags">
                        <a href="/blog/tags/ai-security">AI Security</a>
                        <a href="/blog/tags/prompt-injection">Prompt Injection</a>
                    </span>
                </div>
            </header>
            
            <div class="post-content">
                <!-- Content here -->
            </div>
            
            <footer class="post-footer">
                <div class="share-buttons">
                    <!-- Social sharing -->
                </div>
            </footer>
        </article>
    </main>
    
    <script src="../assets/js/main.js"></script>
    <script src="../assets/js/prism.js"></script>
</body>
</html>
```

### Code Example with Syntax Highlighting

Include Prism.js for syntax highlighting:

```html
<!-- In <head> -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css">

<!-- Before </body> -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>

<!-- Usage -->
<pre><code class="language-python">
import jwt
import os
from datetime import datetime, timedelta

def generate_token(user_id):
    """Generate secure JWT token"""
    secret = os.getenv('JWT_SECRET_KEY')
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=1),
        'iat': datetime.utcnow()
    }
    return jwt.encode(payload, secret, algorithm='HS256')
</code></pre>
```

### Blog Index Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Security Blog - 0xcarnemuerta</title>
    <link rel="stylesheet" href="../assets/css/style.css">
</head>
<body>
    <header></header>
    
    <main class="blog-index">
        <h1>Security Research & Technical Blog</h1>
        
        <div class="blog-posts">
            <article class="post-preview">
                <h2><a href="./prompt-injection-vulnerabilities.html">
                    Prompt Injection Vulnerabilities in LLM Applications
                </a></h2>
                <time datetime="2026-08-03">August 3, 2026</time>
                <p>Deep dive into prompt injection attack vectors and mitigation strategies...</p>
                <a href="./prompt-injection-vulnerabilities.html" class="read-more">Read more →</a>
            </article>
            
            <article class="post-preview">
                <h2><a href="./token-authentication-best-practices.html">
                    Token Authentication Best Practices
                </a></h2>
                <time datetime="2026-08-01">August 1, 2026</time>
                <p>Comprehensive guide to implementing secure token-based authentication...</p>
                <a href="./token-authentication-best-practices.html" class="read-more">Read more →</a>
            </article>
        </div>
    </main>
    
    <script src="../assets/js/main.js"></script>
</body>
</html>
```

## Troubleshooting

### Site Not Loading Locally

**Problem**: Opening `index.html` directly shows broken styles/links.

**Solution**: Always use a local web server instead of `file://` protocol:

```bash
# Use Python server
python3 -m http.server 8000

# Or Node http-server
npx http-server -p 8000

# Or PHP
php -S localhost:8000
```

### Relative Path Issues

**Problem**: Navigation breaks when moving between sections.

**Solution**: Use root-relative paths consistently:

```html
<!-- Bad - breaks in subdirectories -->
<link rel="stylesheet" href="assets/css/style.css">

<!-- Good - works from any depth -->
<link rel="stylesheet" href="/assets/css/style.css">

<!-- Alternative - explicit relative paths -->
<link rel="stylesheet" href="../assets/css/style.css">
```

### GitHub Pages 404 Errors

**Problem**: Pages not found after deployment.

**Solution**: 
1. Ensure all file names are lowercase
2. Check that index.html exists in each directory
3. Verify paths don't have trailing slashes where not needed
4. Wait 2-3 minutes after pushing for Pages to rebuild

### Missing Syntax Highlighting

**Problem**: Code blocks show as plain text.

**Solution**: Include Prism.js and use correct class names:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>

<pre><code class="language-python">
# Your code here
</code></pre>
```

### Mobile Responsiveness Issues

**Problem**: Site doesn't display well on mobile devices.

**Solution**: Add responsive meta tag and media queries:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

```css
@media (max-width: 768px) {
    nav a {
        display: block;
        margin: 0.5rem 0;
    }
    
    pre code {
        font-size: 0.875rem;
    }
}
```

## Best Practices

1. **Keep it Static**: No databases or server-side processing needed—security through simplicity
2. **Version Control**: Commit regularly with descriptive messages about content changes
3. **Content Organization**: Separate blog posts, portfolio items, and resources into dedicated directories
4. **SEO Friendly**: Include meta descriptions, semantic HTML, and descriptive titles
5. **Security Focus**: Use HTTPS, implement CSP headers via GitHub Pages, avoid inline scripts where possible
6. **Accessibility**: Use semantic HTML, alt text for images, ARIA labels where appropriate
