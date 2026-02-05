---
name: playwright-automation
description: Browser automation using Playwright for web scraping, form filling, authentication flows, and screenshot/PDF generation. Use when user needs to automate browser interactions, extract data from websites, test web applications, or capture web pages. Triggers include mentions of "scrape", "automate browser", "fill form", "login to", "screenshot", "web automation", or working with websites programmatically.
---

# Playwright Automation

Automate browser interactions using Playwright. This skill provides patterns for common web automation tasks including data extraction, form submission, authentication, and page capture.

## Installation

Playwright requires installation before first use:

```bash
pip install playwright --break-system-packages
playwright install chromium
```

## Core Patterns

### Basic Page Navigation and Extraction

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto('https://example.com')
    
    # Extract text content
    content = page.locator('h1').text_content()
    
    # Extract multiple elements
    items = page.locator('.item').all_text_contents()
    
    browser.close()
```

### Form Filling

```python
# Fill input fields
page.fill('input[name="username"]', 'myuser')
page.fill('input[type="password"]', 'mypass')

# Click buttons
page.click('button[type="submit"]')

# Select from dropdown
page.select_option('select#country', 'US')

# Check checkboxes
page.check('input[type="checkbox"]')
```

### Authentication Patterns

For sites requiring login:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # Launch visible browser for manual login
    browser = p.chromium.launch(headless=False)
    context = browser.new_context()
    page = context.new_page()
    
    page.goto('https://example.com/login')
    
    # Option 1: Automated login if credentials available
    page.fill('input[name="email"]', 'user@example.com')
    page.fill('input[name="password"]', 'password')
    page.click('button[type="submit"]')
    page.wait_for_load_state('networkidle')
    
    # Option 2: Pause for manual login
    # Inform user: "Please log in manually in the browser window, then press Enter"
    # input("Press Enter after logging in...")
    
    # Save authentication state for reuse
    context.storage_state(path='auth.json')
    
    # Perform authenticated actions
    page.goto('https://example.com/protected-page')
    content = page.content()
    
    browser.close()

# Reuse saved authentication in future runs
with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(storage_state='auth.json')
    page = context.new_page()
    page.goto('https://example.com/protected-page')
    # Already logged in
```

### Wait Strategies

```python
# Wait for element to appear
page.wait_for_selector('.dynamic-content')

# Preferred for scraping: fast and reliable
page.wait_for_load_state('domcontentloaded')
page.wait_for_timeout(2000)  # short wait for JS rendering

# Wait for network to be idle
# WARNING: unreliable on ad-heavy/news sites — trackers and ads
# make continuous requests, causing networkidle to time out.
# Prefer domcontentloaded + short timeout for scraping.
page.wait_for_load_state('networkidle')

# Wait for specific timeout
page.wait_for_timeout(2000)  # 2 seconds

# Wait for navigation
with page.expect_navigation():
    page.click('a.next-page')
```

### Screenshot and PDF Generation

```python
# Full page screenshot
page.screenshot(path='screenshot.png', full_page=True)

# Element screenshot
element = page.locator('.chart')
element.screenshot(path='chart.png')

# PDF generation
page.pdf(path='page.pdf', format='A4')
```

### Handling Dynamic Content

```python
# Wait for AJAX content to load
page.goto('https://example.com')
page.wait_for_selector('.loaded-content')

# Scroll to trigger lazy loading
page.evaluate('window.scrollTo(0, document.body.scrollHeight)')
page.wait_for_timeout(1000)

# Handle infinite scroll
for _ in range(5):
    page.evaluate('window.scrollTo(0, document.body.scrollHeight)')
    page.wait_for_timeout(1000)
```

### Error Handling

```python
from playwright.sync_api import sync_playwright, TimeoutError

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    
    try:
        page.goto('https://example.com', timeout=10000)
        page.wait_for_selector('.content', timeout=5000)
        data = page.locator('.content').text_content()
    except TimeoutError:
        print("Page load or element wait timed out")
    except Exception as e:
        print(f"Error: {e}")
    finally:
        browser.close()
```

## Selector Strategies

Prefer selectors in this order:
1. Data attributes: `[data-testid="submit-btn"]`
2. IDs: `#submit-button`
3. Specific classes: `.submit-btn`
4. Text content: `text="Submit"`
5. XPath as last resort: `//button[@type="submit"]`

### Cookie/Consent Banners

Modern sites often have cookie consent banners or overlays that contain heading elements and buttons. These can interfere with content extraction (e.g., grabbing banner text instead of the actual headline).

```python
# Strategy 1: Dismiss the banner first
try:
    page.locator('button:has-text("Accept"), button:has-text("Reject"), button:has-text("Close")').first.click(timeout=3000)
except:
    pass  # No banner present

# Strategy 2: Scope selectors to main content area, skipping overlays
content = page.locator('main h1, article h1, #content h1').first.text_content()

# Strategy 3: Filter out banner-related text when iterating elements
for el in page.locator('h1, h2').all():
    text = el.text_content().strip()
    if text and 'cookie' not in text.lower() and 'opt out' not in text.lower():
        print(text)
        break
```

## Common Use Cases

### Web Scraping
Extract structured data from websites. Use appropriate wait strategies for dynamic content. Be respectful of rate limits and robots.txt.

### Form Automation
Fill and submit forms programmatically. Verify submission success by checking for confirmation messages or URL changes.

### Testing
Validate web application functionality. Use assertions to verify expected behavior.

### Page Monitoring
Capture screenshots or check for content changes at intervals.

## Best Practices

- Use `headless=False` during development to see what's happening
- Save authentication state to avoid repeated logins
- Use specific selectors to avoid breaking when page structure changes
- Add appropriate waits for dynamic content
- Handle errors gracefully with try/except blocks
- Close browser contexts properly to free resources
- Respect website terms of service and rate limits
