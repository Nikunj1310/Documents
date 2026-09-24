# What is the use?
The `<input>` element in HTML is one of the most versatile and essential elements for creating interactive forms. It allows users to enter data, select options, and interact with web applications in various ways.

## Basic Syntax
The `<input>` tag defines an input field where users can enter data. It is a self-closing (void) tag, meaning it does not require a closing tag.
```xml
<input type="text" name="username">
```

---
# Types of Input Element

The behavior and appearance of the `<input>` element are controlled by its `type` attribute. Here are some of the most commonly used types:

|Type|Description|Example Syntax|
|---|---|---|
|`text`|Single-line text input|`<input type="text" name="user">`|
|`password`|Password input (characters are hidden)|`<input type="password" name="pwd">`|
|`email`|Email address input (validates format)|`<input type="email" name="email">`|
|`number`|Numeric input|`<input type="number" name="age">`|
|`checkbox`|Checkbox input (on/off)|`<input type="checkbox" name="subscribe">`|
|`radio`|Radio button (select one in a group)|`<input type="radio" name="gender" value="m">`|
|`submit`|Submit button|`<input type="submit" value="Send">`|
|`reset`|Reset button (clears the form)|`<input type="reset" value="Clear">`|
|`file`|File upload input|`<input type="file" name="resume">`|
|`date`|Date picker|`<input type="date" name="birthday">`|
|`color`|Color picker|`<input type="color" name="favcolor">`|

---
# Attributes used?
- **[[type attribute]]**: Specifies the type of input (see above).
- **[[name attribute]]**: Name of the input (used as the key when submitting form data).
- **[[id attribute]]**: Unique identifier, often used with `<label>`.
- **[[value]]**: The initial value of the input.
- **[[placeholder]]**: Shows a hint or example inside the input when empty.
- **[[required]]**: Makes the field mandatory.
- **[[readonly]]**: Makes the field non-editable.
- **[[disabled]]**: Disables the field (user cannot interact).
- **[[min, max, step]]**: Used with numeric/date inputs to set allowed ranges.
- **[[checked]]**: Used with checkboxes/radios to set the default checked state.
- **[[autocomplete]]**: Controls browser autofill behavior.