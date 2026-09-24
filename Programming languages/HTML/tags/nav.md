# What is the use?
The `<nav>` tag is a semantic HTML5 element specifically designed to define a section of a webpage that contains navigation links. It helps organize menus and navigation areas, making web content more accessible and meaningful for both users and search engines.

**Key Points About `<nav>`:**
- **Semantic Meaning:** The `<nav>` tag clearly indicates that its content is a navigation section, which improves the structure and accessibility of a webpage.
- **Accessibility:** Assistive technologies (like screen readers) recognize `<nav>` as a navigation area, helping users with disabilities find and use site menus more easily.
- **SEO Benefits:** Search engines use semantic tags like `<nav>` to better understand site structure, potentially improving search rankings.
- **Not for All Links:** Only major navigation blocks (like main menus, sidebars, or footer navigation) should be wrapped in `<nav>`. Ordinary or contextual links (such as those within articles) should not be placed inside a `<nav>` tag

## Basic Syntax:
```xml
<nav>
  <a href="/home">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

## Different types of `<nav>`
![HTML5 nav Element](https://d2u1z1lopyfwlx.cloudfront.net/thumbnails/6bf503bd-0f2e-5833-be96-296a432b251e/ce74b9b5-ab13-5788-ab26-4ae18c234bac.jpg)

There are several common types of navigation (`nav`) structures used in web development, each serving different user interface needs. The `<nav>` tag can be styled and structured in various ways to provide intuitive navigation experiences. Here are the main types:

---

### Types of Navs Offered

| Navigation Type                  | Description & Use Case                                                                             | Example Structure                                                          |
| -------------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Horizontal Navigation Bar**    | Displays navigation links in a row, typically at the top of the page. Used for main site menus.    | `<nav><ul><li>Home</li><li>About</li></ul></nav>`                          |
| **Vertical Navigation Sidebar**  | Arranges links in a column, often used in sidebars for dashboards or admin panels.                 | `<nav><ul><li>Dashboard</li><li>Profile</li></ul></nav>`                   |
| **Breadcrumb Navigation**        | Shows a trail of links indicating the user's location within the site hierarchy.                   | `<nav><ul class="breadcrumb"><li>Home</li> > <li>Category</li></ul></nav>` |
| **Dropdown Navigation**          | Menus with expandable/collapsible submenus for categories and subcategories.                       | `<nav><ul><li>Services<ul><li>Web Design</li></ul></li></ul></nav>`        |
| **Responsive/Mobile Navigation** | Adapts to different screen sizes, often collapses into a hamburger menu on small devices.          | `<button>☰ Menu</button><nav id="navMenu">...</nav>`                       |
| **Footer Navigation**            | Navigation links placed at the bottom of the page, often for legal, policy, or social media links. | `<footer><nav><a href="/privacy">Privacy</a></nav></footer>`               |
| **In-Page Navigation**           | Links that jump to sections on the same page, useful for single-page sites or documentation.       | `<nav><a href="#section1">Section 1</a></nav>`                             |
|                                  |                                                                                                    |                                                                            |

---
## Examples

- **Horizontal Navigation Bar:**  
    Used for main site navigation, typically across the top of the page.
- **Vertical Navigation Sidebar:**  
    Common in admin panels or applications, where navigation stays on the side.
- **Breadcrumb Navigation:**  
    Shows the user's path and helps with orientation within the site.
- **Dropdown Navigation:**  
    Provides expandable submenus, useful for sites with many categories.
- **Responsive/Mobile Navigation:**  
    Adapts layout for mobile devices, often using a toggle button to show/hide links
- **Footer Navigation:**  
    Offers secondary navigation at the bottom of the page for legal or informational links.
- **In-Page Navigation:**
    Allows jumping to different sections of a single page, improving usability for long content.

---
These navigation types can be combined and customized using HTML and CSS to fit the needs of any website or application.

# Attributes used?
- **No Tag-Specific Attributes:**  
    `<nav>` does not have unique attributes, but it supports all global HTML attributes (like `id`, `class`, `style`, etc.) and event attributes.
- **Accessibility Enhancements:**  
    Use `aria-label` or `aria-labelledby` to describe the navigation area for screen readers:
```xml
<nav aria-label="Main Navigation">   
	<!-- navigation links --> 
</nav>
```

However, `nav` does use something called lists in HTML to be more organised.