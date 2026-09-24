# What is the use?
The `<form>` element in HTML is the foundational container for creating interactive forms that collect and submit user input to a server or process it within the browser. It can include various controls like text fields, checkboxes, radio buttons, and buttons.

## When to use?
Whenever you want to create a form?!

## Basic Syntax
The syntax is:
```HTML
<form action="">
	<!--Form elements here-->
</form>
```
There are some compulsory attributes that has to be defined whenever we create an HTML form.

| Attribute                                                                    | Value                                                                                                                   | Description                                                                                             |
| ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [accept-charset](https://www.w3schools.com/tags/att_form_accept_charset.asp) | _character_set_                                                                                                         | Specifies the character encodings that are to be used for the form submission                           |
| [action](https://www.w3schools.com/tags/att_form_action.asp)                 | _URL_                                                                                                                   | Specifies where to send the form-data when a form is submitted                                          |
| [autocomplete](https://www.w3schools.com/tags/att_form_autocomplete.asp)     | on  <br>off                                                                                                             | Specifies whether a form should have autocomplete on or off                                             |
| [enctype](https://www.w3schools.com/tags/att_form_enctype.asp)               | application/x-www-form-urlencoded  <br>multipart/form-data  <br>text/plain                                              | Specifies how the form-data should be encoded when submitting it to the server (only for method="post") |
| [method](https://www.w3schools.com/tags/att_form_method.asp)                 | dialog  <br>get  <br>post                                                                                               | Specifies the HTTP method to use when sending form-data                                                 |
| [name](https://www.w3schools.com/tags/att_form_name.asp)                     | _text_                                                                                                                  | Specifies the name of a form                                                                            |
| [novalidate](https://www.w3schools.com/tags/att_form_novalidate.asp)         | novalidate                                                                                                              | Specifies that the form should not be validated when submitted                                          |
| [rel](https://www.w3schools.com/tags/att_form_rel.asp)                       | external  <br>help  <br>license  <br>next  <br>nofollow  <br>noopener  <br>noreferrer  <br>opener  <br>prev  <br>search | Specifies the relationship between a linked resource and the current document                           |
| [target](https://www.w3schools.com/tags/att_form_target.asp)                 | _blank  <br>_self  <br>_parent  <br>_top                                                                                | Specifies where to display the response that is received after submitting the form                      |

---
# Other elements used in a from:

Whenever we create a form, it has multiple things, like a `label` for each `input` type we are going to provide(like name, email, and stuff). 
Along with that we can also provide `dropdown menu` and `checkboxes` with a compulsory to fill or one box only like settings.

### Creating an input field
Input field is where user gives info,i.e., fills information.
Let's have a look at basic elements used to create an input field in a form:
	- [[<label>]]
	- [[<input>]]
