# What is the use?
The `<label>` tag in HTML is used to define a caption or description for form elements, such as [[<input>]], [[<textarea>]], [[<select>]], and others. Its primary purpose is to improve accessibility and usability by clearly associating text with a specific form control.

It has an attribute called `for` which connects it to the input field it is describing. **Form Control** is what the label element is describing.

## When to use?
Whenever we want to create an input field, this is what user sees and this is what describes the user that what should be entered in the input field.

## Basic Syntax
There are two main ways to associate a label with a form control:
### 1. **Using the `for` Attribute (Explicit Association)**
The `for` attribute in the `<label>` tag must match the `id` of the form control:

```xml
<label for="email">
	Email Address:
</label> 

<input type="email" id="email" name="email">`
```
- When the label is clicked, the input with the matching `id` is focused.

## 2. **Wrapping the Control (Implicit Association)**
You can nest the form control inside the label:

```xml
<label>   
	Email Address:  
	<input type="email" name="email"> 
</label>`
```
- This method automatically links the label to the input without needing `for` and `id`

---
# Attributes used?
The attributes used by it are:
- [[for attribute]]
- [[form attribute]]
