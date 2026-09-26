## HTML Header: Definition and Usage
The `<header>` element in HTML is a semantic tag that represents a container for introductory content or a set of navigational links within a section of a webpage. It is typically used to group together elements such as headings (`<h1>`–`<h6>`), logos, icons, and authorship information at the top of a page or section.

**Key Points about `<header>`:**
- The `<header>` element is not limited to the top of the page; it can be used within any sectioning element, such as `<article>`, `<section>`, or `<aside>`, to introduce that section.
- It commonly contains:
    - One or more heading elements (`<h1>`–`<h6>`)
    - Logos or icons
    - Navigation links
    - Authorship or introductory information
    
- Multiple `<header>` elements can exist on a single page, but a `<header>` cannot be placed inside another `<header>`, `<footer>`, or `<address>` element.

**Example:**
```xml
<header>   
	<h1>Main Page Heading</h1>  
	<nav>    
		<a href="#about">About</a>    
		<a href="#services">Services</a>    
		<a href="#contact">Contact</a>  
	</nav> 
</header>
```
This example shows a typical page header containing a main heading and a navigation bar.

## Header Tags vs. Heading Tags

It is important to distinguish between the `<header>` element and heading tags (`<h1>`–`<h6>`):

|Tag|Purpose|
|---|---|
|`<header>`|Groups introductory or navigational content for a section|
|`<h1>`–`<h6>`|Define headings of varying importance within content|

- Heading tags (`<h1>`–`<h6>`) are used to define the structure and hierarchy of content, with `<h1>` being the most important and `<h6>` the least[2](https://www.w3schools.com/html/html_headings.asp)[5](https://www.w3schools.com/tags/tag_hn.asp)[7](https://www.searchenginejournal.com/on-page-seo/header-tags/).
    
- The `<header>` element often contains these heading tags but is not itself a heading[1](https://www.w3schools.com/tags/tag_header.asp)[4](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/header).
    

## Best Practices
- Use one `<h1>` per page for the main title, then use `<h2>`, `<h3>`, etc., for subheadings to maintain a clear hierarchy.
- Use `<header>` to wrap introductory content or navigation relevant to a section or the entire page.
- Do not use `<header>` within `<footer>`, `<address>`, or another `<header>`.

## Browser Support

The `<header>` element is supported by all modern browsers.
