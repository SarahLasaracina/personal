# GetStyle API

## Overview
The Transmit SDK GetStyle API enables the customization of the appearance of screens during the execution of Transmit flows. Specifically, at runtime, the API retrieves styles from a JSON stylesheet and applies them to the screen's UI elements, similar to applying styles from a CSS stylesheet. The JSON stylesheet is stored on the client side and can be customized to reflect the client brand colors, ensuring that the Transmit flow screen remains consistent with the client's website visual identity. 

## Tags

Both the API query and the JSON stylesheet use tags to identify each UI element. A tag may consist of one or multiple tag items, with each item representing a specific characteristic or feature of the UI element, such as its type, context of use, state, and others. The larger the tag set is, the more specific both the UI element usage and the corresponding rule's case of application are. The example above shows the possible structure of tags:

| UI element                       | Tag                              |
| -------------------------------- | -------------------------------- |
| H1 title                         | [`title`]                      |
| Authentication screens’ H1 title | [`title`, `password-screen`]     |
| Save button                      | [`button`, `save-button`]        |
| Save button (hover)              | [`button`, `save-button`, `state`] |

## JSON stylesheet 

### Structure

The JSON stylesheet consists of a list of style rules defining for each UI element (`tags` field) the related formatting styles (`properties` field). Each JSON stylesheet also provides predefined formatting styles used as fallback (`values`).
```
styles
    rule
        tags
        properties
    rule
        tags
        properties
    …
values
```
The JSON stylesheet is stored on the client side and then customizable by the client.

### Fields

| Field                | Nested in            | Description                                             | Data type | Mandatory |
| -------------------- | -------------------- | ------------------------------------------------------- | --------- | --------- |
| `styles `          | -                   | List of style rules                                     | array     | YES       |
| `rule`             | `styles `          | Style rule  | object    | YES       |
| `tags `            | `rule `            | UI element to style                                     | array     | YES       |
| `properties`       | `rule `            | Formatting properties to style the UI element           | object    | YES       |
| `backgroundColor ` | `properties `      | Background color property for the UI element            | object    | NO        |
| `hex `             | `backgroundColor ` | Hexadecimal color value                                 | string    | NO        |
| `ref `             | `backgroundColor ` | References a predefined value. See values               | string    | NO        |
| `tintColor `       | `properties `      | Tint color property for the UI element                  | string    | NO        |
| `imageUrl `        | `properties `      | URL or path to the image file related to the UI element | string    | NO        |
| `values `          | -                  | List of predefined values used as fallback           | object    | YES       |
| `primary-color `   | `values `          | Reference of the fallback color for any UI element      | object    | NO        |



## Query processing
At query time, the GetStyle API queries the JSON stylesheet with the tags that identify the screen’s UI elements. The SDK scans then the JSON stylesheet in cascade order to find style rules matching the query’s tags. As a general rule, the scanning process focuses on searching for applicable style rules that contribute to formatting the UI elements. 

As a priority, the scanning searches for style rules whose tag sets match the query tags. When a matching tag set is found, the corresponding rule applies.

> Note: a tag set may include either single or multiple tag items. 

To ensure complete formatting of the UI elements, the scanning also searches for rules that refer to each tag item in the query tag set. If these rules are found, and their properties complement the main rule, they also contribute to styling the UI element. As a result, multiple style rules may participate in styling the same UI element.

If conflicts arise at either the tag or property level, the last applicable rule in the stylesheet file takes precedence and overrides the previous ones.

**Example**

In the JSON stylesheet above, for an API query referencing the `["list-item", "hover"]` tag group, multiple style rules apply.

- Second rule’s `tags` items match both tag items in the query, and it qualifies the `BackgroundColor` property.
- First rule’s`tags` item matches the `"list-item"` tag item, and it qualifies the `tintColor` property.

Since both properties qualify the `"list-item"` tag item, both rules are applied. Note that although both styles reference the `BackgroundColor` property, the BackgroundColor style of the second rule takes precedence because it is the last one referenced in the stylesheet.

![stylehsheet img](stylesheetexample.JPG)
