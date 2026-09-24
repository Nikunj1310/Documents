**Expected to know:** [[HTML(Hyper-Text Makeup Language)#HTML Elements]]
**Topics Covered:** 
**Tags:**

There are various elements, but a few basic ones are listed below:

---
# `<!doctype >`
This is a preamble rather than the element itself.
In the mists of time, when HTML was young (around 1991/92), doctypes were meant to act as links to a set of rules that the HTML page had to follow to be considered good HTML, which could mean automatic error checking and other useful things. 
However, these days, they don't do much and are basically just needed to make sure your document behaves correctly. 

That's all for now.

---
# `<html></html>`

The [`<html>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/html) element wraps all the content on the entire page and is sometimes known as the **root element**. 
It also includes the `lang` [attribute](https://developer.mozilla.org/en-US/docs/Glossary/Attribute)(i.e., the language attribute of the web page), which sets the primary language of the document.

## lang attribute

The `lang` attribute in HTML specifies the natural language of the content within an element. 
Its primary purpose is to help browsers, search engines, and assistive technologies (like screen readers) correctly interpret, pronounce, and present the text on a webpage.

---
# `<head></head>`

The [`<head>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/head) element acts as a container for all the stuff you want to include on the HTML page that _isn't_ the content you are showing to your page's viewers. 

This includes things like [keywords](https://developer.mozilla.org/en-US/docs/Glossary/Keyword) and a page description that you want to appear in search results, [CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS) to style the content, character set declarations, and more.

A few head elements are:
## `<title></title>`
This is used inside the `<head>` of the `html` script.
The [`<title>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/title) element sets the title of your page, which is the title that appears in the browser tab the page is loaded in. 
It is also used to describe the page when you bookmark/favorite it.

---
# `<body></body>`

The [`<body>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/body) element contains _all_ the content that you want to show to web users when they visit your page, whether that's text, images, videos, games, playable audio tracks, or whatever else. 

At the moment(in the `practice1.html` file) it only contains a single `<img>` element, but we'll add more content later on.

A few important body elements are:
## Heading Elements

```html
<!DOCTYPE html>
<html>
  <head>
    <title>HTML Heading Elements Example</title>
  </head>
  <body>
    <h1>This is heading 1.</h1>
    <h2>This is heading 2.</h2>
    <h3>This is heading 3.</h3>
    <h4>This is heading 4.</h4>
    <h5>This is heading 5.</h5>
    <h6>This is heading 6.</h6>
  </body>
</html>
```

The `<h1>` element is an HTML tag used to define the most important heading on a webpage. It represents the highest level of section heading, with `<h2>` to `<h6>` providing progressively lower levels of subheadings.

The text inside an `<h1>` tag usually describes the main topic or purpose of the page, helping both users and search engines quickly understand what the page is about.

The output page looks like this:
![[Pasted image 20250512235255.png]]

## Paragraph element

`<p>` element is used to form paragraph in an web page via html.
The `<p>` element in HTML defines a paragraph. 

It is a fundamental block-level element used to group and separate blocks of text, making web content more readable and structured. 

Browsers automatically add space (a blank line or margin) before and after each `<p>` element, visually separating paragraphs from one another.

---

# `<meta>`

What does meta mean? 
	It means something describing that something itself, for example, a movie about making movies, a game about games, a book about books, etc.
**In gaming:** "Meta" can refer to the "most effective tactics available" or the "metagame," which is using knowledge about the game (often from outside the game itself) to gain an advantage

The `<meta>` element in HTML is used to provide metadata-information about the data-of a web page. 
Metadata-information refers to information that describes, explains, or provides context about other data, rather than being the main content itself. 

In other words, metadata is "data about data"-it summarizes, categorizes, or gives details about the actual data, making it easier to find, use, and manage.

Metadata is not displayed on the page itself but is machine-readable and used by browsers, search engines, and other web services to understand and process the page content.
`<meta>` tags are always placed inside the `<head>` section of an HTML document.
## Common Types of Meta Tags

| Meta Attribute       | Example Usage                                                            | Purpose                                             |
| -------------------- | ------------------------------------------------------------------------ | --------------------------------------------------- |
| charset              | `<meta charset="UTF-8">`                                                 | Defines the character encoding                      |
| name="description"   | `<meta name="description" content="A brief summary of the page.">`       | Describes the page content for search engines       |
| name="keywords"      | `<meta name="keywords" content="HTML, CSS, JavaScript">`                 | Lists keywords relevant to the page                 |
| name="author"        | `<meta name="author" content="John Doe">`                                | Specifies the author of the page                    |
| name="viewport"      | `<meta name="viewport" content="width=device-width, initial-scale=1.0">` | Controls layout on mobile devices                   |
| http-equiv="refresh" | `<meta http-equiv="refresh" content="30">`                               | Refreshes the page every 30 seconds                 |
| name="robots"        | `<meta name="robots" content="noindex, nofollow">`                       | Tells search engines how to index or crawl the page |
**What is `name` in a `<meta>` Tag?**
	- The `name` attribute in a `<meta>` tag specifies the type or category of metadata being provided about the HTML document. 
	- It works together with the `content` attribute to form a name-value pair that describes information about the page, such as its description, author, keywords, or viewport settings
	- The name tags called "description", "author", and "keyword" are used by the other entities to get to know about the site itself.

In our example, we have used: 
### **`<meta charset = "UTF-8">`

This element sets the character set your document should use to [UTF-8](https://developer.mozilla.org/en-US/docs/Glossary/UTF-8), which includes most characters from the vast majority of written languages. 

Essentially, it can now handle any textual content you might put on it. There is no reason not to set this, and it can help avoid some problems later on.

### **`<meta name = "viewport" content="width=device-width">`
This [viewport element](https://developer.mozilla.org/en-US/docs/Web/CSS/CSSOM_view/Viewport_concepts#mobile_viewports) ensures the page renders at the width of the browser [viewport](Viewport.md), preventing mobile browsers from rendering pages wider than the viewport and then shrinking them down.

---
# Image Embedding

Images are important part of a site as it makes it more visual. Our example also uses images by using `<img>` element:
```html
<img src="" alt="My test image" />
```

This embeds an image into our page in the position it appears. It does this via the `src` (source) attribute, which contains the path to the image file we want to embed.

We have also included an `alt` (alternative) attribute. In the [`alt` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#authoring_meaningful_alternate_descriptions), you specify descriptive text for users who cannot see the image, possibly because of the following reasons:

1. They are visually impaired. Users with significant visual impairments often use tools called screen readers to read out the alt text to them.
2. Something has gone wrong, causing the image not to display. If the `src` attribute does not contain a valid path to an image, the alt text will be displayed instead.

Now time to provide it some image. There are multiple ways of providing an HTML code with image:

| Method              | Example                                                                                                | Description                                                                                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Relative Path       | `src="images/pic.jpg"`,`src="../image/picture.jpg"`,etc,.                                              | Relative to current HTML file, looks into the folder where the html file is stored.                                                                                   |
| Root-Relative Path  | `src="/images/pic.jpg"`                                                                                | From website root,i.e., irrespective of HTML file's location, starts from the root location of domain.                                                                |
| Absolute Path (URL) | `src="https://example.com/images/pic.jpg"`                                                             | Full web address (external or internal server)                                                                                                                        |
| Local Absolute Path | `src="file:///C:/images/pic.jpg"`,``<img src="/home/rubber_duck/Pictures/picture.jpg" alt="Picture">`` | Full local file path (for local testing only)This uses the full path on your computer. It works locally but **will not work when the page is hosted on a web server** |

---
# Some more basic elements

## **`<link>`

The `<link>` element in HTML is used to connect your web page to other resources that help it work or look better. Unlike clickable links you see on web pages, the `<link>` element works behind the scenes and is not visible to people visiting your site.

### What Does `<link>` Do?
- It tells the browser about files your page needs, like stylesheets (for colors and layout), icons (the little image in the browser tab), or other resources.
- It is always placed inside the `<head>` section at the top of your HTML file.
### The attributes
Some basic attributes that are required are:
#### **`rel` Attribute**
- The `rel` attribute is required and specifies the relationship between the current document and the linked resource.
- Examples: `"stylesheet"` (for CSS), `"icon"` (for favicon,i.e., small icon on browser tab), `"next"` (for navigation), etc.
- Without `rel`, the browser does not know how to use the linked resource.
#### **`href` Attribute**
- The `href` attribute specifies the location (URL) of the external resource you want to link to, such as a CSS file or an icon image.
- Example: `href="styles.css"` links to a CSS file named "`styles.css`"(Note: Path format is similar to what we use in `src` attribute in `img` element).
- Without `href`, the browser does not know where to find the resource

For example:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Project</title>
    <link rel="stylesheet" href="Css/main.css">
</head>
<body>
    
</body>
</html>
```

So here, we used `link` element to tell which `stylesheet` to use for the CSS of the site.