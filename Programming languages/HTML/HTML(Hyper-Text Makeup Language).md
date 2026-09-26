**HTML** (HyperText Markup Language) is the most basic building block of the Web. It defines the meaning and structure of web content.
It is a _markup language_ consisting of a series of **[[#HTML Elements]]** used to wrap (or enclose) text content to define its structure and cause it to behave in a certain way.

Other technologies besides HTML are generally used to describe a web page's appearance/presentation ([CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)) or functionality/behavior ([JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)).

"Hypertext" refers to links that connect web pages to one another, either within a single website or between websites. 
Links are a fundamental aspect of the Web. By uploading content to the Internet and linking it to pages created by other people, you become an active participant in the World Wide Web.

# HTML page default look

# HTML page structuring

# HTML Elements

HTML uses `markup` to annotate text, images, and other content for display in a Web Browser.
These `Markups` include special `elements` like:
- `<head>`
- `<title>`
- `<body>`
- `<header>`
- `<footer>`
- `<article>`
- etc,.

An HTML element is set off from other text in a document by "tags", which consist of the element name surrounded by `<` and `>`. 
The name of an element inside a tag is case-insensitive. That is, it can be written in uppercase, lowercase, or a mixture. 

For example, the `<title>` tag can be written as `<Title>`, `<TITLE>`, or in any other way. However, the convention and recommended practice is to write tags in lowercase.

Each element in HTML has attribute. **Attributes in HTML** are special words added to HTML elements to provide extra information, modify their behavior, or change their appearance. 
They are always included in the opening tag of an element and usually come as name/value pairs, such as `name="value"`.

Using an element is simple, you start an element by using `<element_name>` and close it by using `<\element_name>`.

# Using elements and attributes in HTML to make websites!
Each HTML element has attributes that can be used with it for custom edits and details about how that element is supposed to work in that particular section of the page.

***(add what are attributes and how they work, and types of attributes)***

## Basics of HTML code:
So for example, in the practice file 1, we have used various elements:
![[Pasted image 20250512230126.png]]
We have used a variety of elements and their functions would be [](Basic%20HTML%20Elements%20used%20in%20Practice1.md).

---
## Head Elements 

Now lets have a look at `practice2.html`, where we tweak around with the head of the webpage and work behind the screen:(create a project that uses link element to add icons and explains link. Also explain where is all the code is stored which is being used to run html, what compiles it and how does it know different names in meta, and how does it know diff ref in link, what if I change the ref name and all, explain that in this project, also include the compilation thing in the start of this file.)

---
## Text Basics

Now lets have a look at `Practice3.html` that formats the body of our webpage:

`<hr>` is called horizontal rule, no closing tag and adds a line in our page. Is used in `<body>` of the html code of course.

We can create linebreak using `<br>` element in HTML, which can be used in-line.

We can emphasize on something via `<em></em>` element, what is done on emphasis depends on the style/CSS of the code, but by default, the text is converted into *italics*. If we want to really emphasize, then we can use `<strong></strong>` element. By default, this makes the content **bold**. Both of them are in-line elements, i.e., can be used in the line.

Now when it comes to adding whitespaces, or symbols like < or >, we cannot do that directly due to some reasons, like whitespaces are ignored and <  and >  are considered as the part of the script, so for these king of things, we have something called HTML Entities.
- `&nbsp;` is for whitespace
- `&lt;` for <
- `&gt;` for >
- `&copy;` for copyright symbol
- and many more!
They are similar to what is used in LaTeX.

We can also **add comment inside our code**, using `< ! - - You comment here - - >`.

We can also add a footer.

---
## [[Lists in HTML]]

Time to use lists, as a lot of web content is made of lists.There two types of lists in HTML:
- Ordered Lists:
	- **Ordered lists** are for lists where the order of the items does matter, such as a list of cooking instructions in a recipe. These are wrapped in an [`<ol>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol) element.
- Unordered lists:
	- **Unordered lists** are for lists where the order of the items doesn't matter, such as a shopping list. These are wrapped in a [`<ul>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul) element.
- Each item inside the lists is put inside an [`<li>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li) (list item) element.
## Description List in HTML

A **description list** in HTML is used to display a list of terms and their corresponding descriptions, making it ideal for glossaries, metadata, FAQs, or any situation where you need to pair names with values or explanations.

### Structure and Core Tags

A description list uses three main tags:

- `<dl>`: Defines the start and end of the description list.
- `<dt>`: Stands for "description term" and is used for the term or name.
- `<dd>`: Stands for "description definition" and is used for the description or value associated with the term.
```html
<dl>
  <dt>Coffee</dt>
  <dd>Black hot drink</dd>
  <dt>Milk</dt>
  <dd>White cold drink</dd>
</dl>
```

---
## Links in HTML
Time to link stuff up using links in HTML. What we are going to do here is create links that will help us link other stuff on the we with what we have created.

Consider the given HTML page as a reference:
`Create a page that has anchor link, Maybe create somthing that mentions all game developing companies or smt like that.`
### Anchor Tag in HTML
The **anchor tag** (`<a>`) is a fundamental HTML element used to create hyperlinks. These links allow users to navigate between web pages, jump to specific sections within a page, download files, send emails, and more.

**Basic Syntax**
```html
<a href="URL">Link Text</a>
```
- `href`: Specifies the destination URL or location.
- The text (or content) between `<a>` and `</a>` is what users see and click.
#### Key Attributes

|Attribute|Description|Example Value|
|---|---|---|
|`href`|The URL or path to the resource (required for a working link)|"https://..."|
|`target`|Where to open the link (`_blank` for new tab, `_self` for same tab, etc.)|"_blank"|
|`rel`|Describes the relationship between the current page and the link (e.g., `nofollow`, `noopener`)|"noopener noreferrer"|
|`download`|Prompts the browser to download the linked file instead of navigating to it|(no value needed)|
|`title`|Tooltip text shown on hover|"More info"|
|`hreflang`|Specifies the language of the linked document|"en", "fr"|
|`id`|Used for creating bookmarks or linking to specific sections|"top"|

#### Examples

**External Link:**
```html
<a href="https://www.google.com" target="_blank" rel="noopener">Google</a>
```
**Internal Link:**
```html
<a href="/about.html">About Us</a>
```
**Section Link (Jump Link):**
```html
<a href="#contact">Contact Section</a> <!-- Later in the page --> <section id="contact">...</section>
```
For jumping back to top, use only `#`.
**Email Link:**
```html
<a href="mailto:info@example.com">Email Us</a>
```
**Download Link:**
```html
<a href="/files/report.pdf" download>Download Report</a>
```

**Summary:**  

The `<a>` (anchor) tag is the primary way to create hyperlinks in HTML. Its most important attribute is `href`, which defines the link’s destination. 
It supports several other attributes for controlling link behavior, improving accessibility, and optimizing for search engines. 
Anchor tags are the building blocks of web navigation, connecting pages, resources, and sections across the internet.

---
## [[Adding Images]]
We can add images using `<img>` tag in html. 

---
## Sematic Elements in HTML
**Semantic elements** in HTML are tags that clearly describe their meaning and the type of content they contain, both to the browser and to developers.

Unlike non-semantic elements like [[div]] and [[span]], which provide no information about their content, semantic elements make the structure and purpose of a web page more obvious.

The code written in Practice.html covers this.

Below are some of the most widely used semantic elements in HTML5, along with their typical purposes:

| Element          | Purpose/Description                                                               |
| ---------------- | --------------------------------------------------------------------------------- |
| [[Programming languages/HTML/header]]     |  Defines introductory content or navigation links at the top of a page or section |
| [[nav]]        | Contains navigation links (main menus, tables of contents, etc.)                  |
| `<main>`         | Specifies the main content unique to the page                                     |
| [[section]]    | Represents a thematic grouping of content, typically with a heading               |
| `<article>`      | Defines independent, self-contained content (e.g., blog post, news article)       |
| `<aside>`        | Contains content tangentially related to the main content (e.g., sidebars)        |
| `<footer>`       | Defines a footer for a page or section (e.g., copyright, contact info)            |
| `<figure>`       | Specifies self-contained content, like images, diagrams, or code listings         |
| `<figcaption>`   | Provides a caption for a `<figure>` element                                       |
| `<details>`      | Defines additional details that the user can view or hide                         |
| `<summary>`      | Provides a summary or heading for a `<details>` element                           |
| `<mark>`         | Highlights or marks text for reference                                            |
| `<time>`         | Represents a specific period in time                                              |
| [[blockquote]] | Used to represent quotes in HTML                                                  |

---
## Creating Tables

This would probably be the part of CSS but here, we are covering just the basics of it. 

Create a HTML page for tables related to employees of an organisation.

Tables in HTML5 are used to organize and display data in a structured, grid-like format consisting of rows and columns. They are essential for presenting tabular data such as schedules, financial information, or any content that benefits from a clear, organized layout.

### **Basic Structure of an HTML Table**

An HTML table is created using the `<table>` element. Inside the table, rows are defined with `<tr>` (table row), and each row contains cells, which are either `<td>` (table data) for regular cells or `<th>` (table header) for header cells.

**Example:**
```html
<table>   
	<tr>    
		<th>Name</th>    
		<th>Age</th>    
		<th>City</th>  
	</tr>  
	<tr>    
		<td>Emil</td>    
		<td>16</td>    
		<td>Berlin</td>  
	</tr>  
	<tr>    
		<td>Tobias</td>    
		<td>14</td>    
		<td>London</td>  
	</tr> 
</table>
```

- `<table>`: Defines the table.
- `<tr>`: Defines a table row.
- `<th>`: Defines a header cell (bold and centered by default).
- `<td>`: Defines a standard data cell.

### **Additional Table Elements**

HTML5 introduces several elements to add semantic meaning and structure to tables:

| Tag          | Purpose                                         |
| ------------ | ----------------------------------------------- |
| `<caption>`  | Adds a title or description to the table        |
| `<thead>`    | Groups the header content (rows)                |
| `<tbody>`    | Groups the main body content (rows)             |
| `<tfoot>`    | Groups the footer content (rows)                |
| `<colgroup>` | Groups columns for formatting                   |
| `<col>`      | Specifies column properties within `<colgroup>` |

**Example with Semantic Elements:**
```html
<table>   
	<caption>Student Information</caption>  
	<thead>    
		<tr>      
			<th>Name</th>      
			<th>Age</th>      
			<th>City</th>    
		</tr>  
	</thead>  
	<tbody>    
		<tr>      
			<td>Emil</td>      
			<td>16</td>      
			<td>Berlin</td>    
		</tr>    
		<tr>      
			<td>Tobias</td>      
			<td>14</td>      
			<td>London</td>    
		</tr>  
	</tbody>  
	<tfoot>    
		<tr>      
			<td colspan="3">End of List</td>    
		</tr>  
	</tfoot> 
</table>
```
- `<caption>` gives the table a title.
- `<thead>`, `<tbody>`, and `<tfoot>` help organize and semantically group rows.
### **Common Table Attributes**

- `colspan`: Makes a cell span multiple columns.
- `rowspan`: Makes a cell span multiple rows.
- `border`, `width`, etc.: Control appearance (though CSS is preferred for styling).

**Example:**

```html
<tr>   
	<th colspan="2">Header 1</th>  
	<th rowspan="2">Header 2</th> 
</tr>
```
This merges cells for more complex layouts.

### **Content Inside Table Cells**

Table cells can contain not just text, but also images, links, lists, forms, and even other tables.

---
## Forms and Input in HTML

HTML forms are a fundamental part of web development, enabling websites to collect user input for various purposes, such as registration, login, search, or feedback. 
Forms are interactive sections of a web page that contain fields and controls for users to enter or select data, which is then typically sent to a server for processing

### Creating an HTML Form from Scratch
To create a complete HTML form, you use the `<form>` element as the container and combine it with various other elements to gather and organize user input. Below is a step-by-step example of a basic registration form, followed by explanations of each element used.

---
#### **Example: Simple Registration Form**

```xml
<form action="/submit_registration" method="post">   
	<h2>Registration Form</h2>     
	
	<label for="username">Username:</label>  
	<input type="text" id="username" name="username" required>  
	<br><br>     
	
	<label for="email">Email:</label>  
	<input type="email" id="email" name="email" required>  
	<br><br>     
	
	<label for="password">Password:</label>  
	<input type="password" id="password" name="password" required>  
	<br><br>     
	
	<label for="gender">Gender:</label>  
	<select id="gender" name="gender">    
		
		<option value="">--Select--</option>    
		<option value="male">Male</option>    
		<option value="female">Female</option>    
		<option value="other">Other</option>  
		
	</select>  
	<br><br>     
	
	<label>    
		<input type="checkbox" name="terms" required>    
		I agree to the terms and conditions  
	</label>  
	<br><br>     
	
	<input type="submit" value="Register"> </form>`
```

---
## Elements used in creating a form

| Element                | Purpose                                                                                                                                                                                                                                                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [[form]]             | The main container for all form elements. Attributes:<br>- `action`: URL where form data will be sent after submission.<br>- `method`: HTTP method (`get` or `post`) used to send data.                                                                                                                                     |
| [[h2]]               | A heading for the form, improves structure and accessibility.                                                                                                                                                                                                                                                               |
| [[label]]            | Describes each input field, improves accessibility. The `for` attribute links it to the corresponding input by `id`.                                                                                                                                                                                                        |
| [[input]]            | Creates various types of user input fields. <br>Common types include:<br>- `text`: Single-line text input.<br>- `email`: Validates input as an email address.<br>- `password`: Hides the input text.<br>- `checkbox`: Allows selection of an option (e.g., agreeing to terms).<br>- `submit`: Button to send the form data. |
| [[Programming languages/HTML/tags/select]]           | Creates a dropdown menu. Contains `<option>` elements for each choice.                                                                                                                                                                                                                                                      |
| [[option]]           | Represents each item in a dropdown menu.                                                                                                                                                                                                                                                                                    |
| [[br]]               | Line break for better visual separation (optional for layout).                                                                                                                                                                                                                                                              |
| [[required attribute]] | Makes the field mandatory; form cannot be submitted if not filled.                                                                                                                                                                                                                                                          |

---

## **How It Works**

- The form starts with `<form action="/submit_registration" method="post">`, meaning when the user submits the form, the data is sent to `/submit_registration` using the POST method.
- Each `<label>` describes an input field, and the `for` attribute connects it to the corresponding input’s `id`.
- `<input type="text">`, `<input type="email">`, and `<input type="password">` collect the username, email, and password.
- `<select>` and `<option>` allow the user to choose their gender from a dropdown list.
- `<input type="checkbox">` is used for agreeing to terms and conditions.
- `<input type="submit">` creates the button that submits the form.

---
# Time to learn by making
HTML has a lot of things, and not everything can be covered like this, cannot have multiple documentation of same stuff, so it is time to learn stuff by making them, and we will use [w3schools](https://www.w3schools.com/tags/default.asp) for things that are new!!!

## Project 1

We created a simple to-do task site, where we embeded a map, used some new features and learnt about how sematic elements are arranged and used.

The Final look of the site is:

The HTML for it looks like this:
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Project</title>
    <link rel="stylesheet" href="Css/main.css">
</head>
<body>
   <div class="navbar"> <!--This is for the navigation bas-->
        <div class="container"> <!--This one is another container created to keep the navigation bar in center-->
            <!--
            A tip to create a container with a class, use '.' 
            followed by class name you want to give without any spaces 
            in between
            -->
            <a href="https://google.com">Remember<span>That</span></a>
            <!--This is for the logo, as usually logo is used for ppl to 
            access the homepage directly-->
            
            <!--
            Now this is a menu button for phone version of this site
            -->
            <img id="mobile.cta" class="mobile-menu" src="images/menu.svg" alt="">
            
            
            <nav>
            <!--Using nav tag for navigation bar elements like homepage and all, recogmmended-->
                <img id="mobile-exit" class="mobile-menu-exit" src="images/exit.svg" alt="close any navigation bar"> <!--Whenever we open a navigation, ppl should have something to close it as well-->
                <ul class="PrimaryNav">
                    <li class="current"><a href="#">Home</a></li>
                    <li><a href="#">Features</a></li>
                    <li><a href="#">Pricing</a></li>
                    <!--Tip, use shift+alt+ctrl+(downArrow) key to replicate stuff-->
                </ul>
                <ul class="SecondaryNav">
                    <li><a href="#">Contact</a></li>
                    <li class="Go-premium-cta"><a href="#">Go Premium</a></li>
                    <!--Tip, Give class to all elements to apply different css to, helps a lot-->
                </ul>
            </nav>

        </div>
   </div> 

   <section class="hero">
        <div class="container"> <!--This is the container for the heading section-->
            <div class="left-col"> <!--For all the text and buttons-->
                <p class="subhead">
                    It's Nitty &amp; Gritty
                </p>
                <h1>A Taks App that doesn't suck</h1>

                <div class="hero-cta"> <!--The buttons below the main heading text-->
                    <a href="#" class="primary-cta">Try for free</a>
                    <a href="#" class="watch-video-cta">
                        <img src="images/watch.svg" alt="Watch a video">
                        Watch a video
                    </a>
                </div>
            </div>
            <img src="images/illustration.svg" alt="Image for the graphics of this hero section">
        </div>
   </section>

   <section class="features-section">
        <div class="container">
            <ul>
                <li>Unlimited Tasks</li>
                <li>SMS Task Reminders</li>
                <li>Confetti Explosions upon Task Completions</li>
                <li>Social Media Announcements</li>
                <li>Real Time Collaboration</li>
                <li>Other awesome features</li>
            </ul>

            <img src="images/holding-phone.jpg" alt="Man holding phont photo from front">
        </div>
   </section>

   <section class="testimonial-section">
        <div class="container">
            <ul>
                <li>

                    <img src="images/person.jpg" alt="Person">
                    <blockquote>"This has made my life so organised, simple and fun!"</blockquote>
                    <cite>-Jane Doe</cite>
                </li>
                
                <li>

                    <img src="images/person.jpg" alt="Person">
                    <blockquote>"This has made my life so organised, simple and fun!"</blockquote>
                    <cite>-Jane Doe</cite>
                </li>
                
                <li>

                    <img src="images/person.jpg" alt="Person">
                    <blockquote>"This has made my life so organised, simple and fun!"</blockquote>
                    <cite>-Jane Doe</cite>
                </li>
                
            </ul>
        </div>
   </section>
   <section class="contact-section">
        <div class="container">
                <div class="contact-left">
                    <h2>Contact</h2>
                    <form action="">
                        <label for="name">Name</label>
                        <input type="text" name="name" id="name" placeholder="Enter your name">
                            <br> <br>
                        <label for="email">E-mail</label>
                        <input type="text" name="email" id="email" placeholder="Enter your email">
                            <br> <br>
                        <label for="message">Message</label>
                        <textarea name="message" id="message" rows="10" cols="30">
                        </textarea>

                        <input type="submit" class="send-message-cta" value="Send Message">

                    </form>
                </div>
                <div class="contact-right">
                    <!--Time to add a map-->
                    <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d7037738.296334398!2d-29.730330792252243!3d64.49145954571817!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x48d22b52a3eb6043%3A0x6f8a0434e5c1459a!2sIceland!5e0!3m2!1sen!2sin!4v1748410180981!5m2!1sen!2sin" width="600" height="450" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
                </div>
        </div>
   </section>
</body>
</html>
```

 We used `iframe` to embed a map, and we used `textarea` for message sent.

## Project 2
### For Navigation bar:
It is enclosed under a [[Programming languages/HTML/header]] called `nav-container`.
[[nav]] element is used for the overall navigation bar
- A space is given for the Netflix logo first
- Two division are created:
	- `leftcol`
	- `rightcol`
- After that, for left col, space is given for each element in that col using [[Lists in HTML#1. **Unordered List (`<ul>`)**]]
- Similarly for right col, we have another list, but a nested list is created for the dropdown menu.