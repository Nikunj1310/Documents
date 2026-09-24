# What is the use?
The `<blockquote>` tag is a semantic HTML element used to indicate that a section of text is a **long quotation from another source**. It is typically used for quoting paragraphs or larger blocks of text, as opposed to the `<q>` tag, which is meant for short, inline quotes.

## When to use?
- For quoting **multiple sentences, paragraphs, or extended text** from another source.
- For **customer testimonials**, **expert opinions**, or **highlighted quotations** in articles and documentation

## Basic Syntax
```xml
<blockquote>
  Quoted content goes here.
</blockquote>
```

## Key Features

- **Block-level element:** `<blockquote>` creates a new block, visually separating the quoted content from the rest of the page, usually by indentation.
- **Semantic meaning:** It tells browsers, search engines, and assistive technologies that the content is a quotation from another source.
- **Source attribution:** The optional `cite` attribute can be used to specify the URL of the source of the quote.
```xml
<blockquote cite="https://www.example.com/source">   
	The only limit to our realization of tomorrow is our doubts of today. 
</blockquote>
```
The `cite` attribute is not displayed on the page but can be accessed by search engines and assistive technologies.
- **Styling:** Browsers apply default indentation, but you can customize the appearance using CS

---
# Attributes used?
