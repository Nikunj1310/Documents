# What is the use?
The `<section>` tag is a **semantic HTML5 element** used to define a distinct, standalone section of content within a web page. It is intended for grouping related content that shares a common theme or purpose, such as chapters, thematic groupings, or major parts of a document.

Here, the highlighted section is what we would call a section:
![[Pasted image 20250527204824.png]]

It is the first thing a user usually sees.

## When to use?
- **Semantic Structure:** The `<section>` element gives meaning to the content it wraps, helping browsers, search engines, and assistive technologies understand the structure of your page.
- **Use Case:** Use `<section>` to divide your page into logical parts, such as "Introduction," "Features," or "Contact Us," especially when each part could have its own heading
- **Standalone Thematic Grouping:** Use `<section>` for content that forms a distinct part of your document, such as a chapter, tab, or feature area.
- **With Headings:** Each `<section>` should generally have its own heading to describe its purpose.
- **Not for Generic Containers:** If you only need a container for styling or scripting and not for meaning, use `<div>` instead.
## Basic Syntax
A typical `<section>` includes a heading (`<h1>–<h6>`) to describe its content:

```xml
<section>   
	<h2>Our Services</h2>  
	<p>
		We offer web development, design, and SEO optimization.
	</p> 
</section>
```
This groups the heading and paragraph as a single, meaningful unit.

---
# Attributes used?
It uses the following attributes:
- [[class attribute]]