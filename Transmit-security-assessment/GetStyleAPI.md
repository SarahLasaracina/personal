# GetStyle API

## Overview
The Transmit SDK GetStyle API enables the customization of the appearance of screens during the execution of Transmit flows. Specifically, at runtime, the API retrieves styles from a JSON stylesheet and applies them to the screen's UI elements, similar to applying styles from a CSS stylesheet. The JSON stylesheet is stored on the client side and can be customized to reflect the client brand colors, ensuring that the Transmit flow screen remains consistent with the client website's visual identity. 

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

### Tags

Both the API query and the JSON stylesheet use tags to identify each UI element. A tag may consist of one or multiple tag items, with each item representing a specific characteristic or feature of the UI element, such as its type, context of use, state, and others. The larger the group of tags is, the more specific both the UI element usage and the corresponding rule's case of application are. The example above shows the possible structure of tags:

| UI element                       | Tag                              |
| -------------------------------- | -------------------------------- |
| H1 title                         | [`title`]                      |
| Authentication screens’ H1 title | [`title`, `password-screen`]     |
| Save button                      | [`button`, `save-button`]        |
| Save button (hover)              | [`button`, `save-button`, `state`] |

### Query processing
At query time, the GetStyle API queries the JSON stylesheet with the tags that identify the screen’s UI elements. Then, the SDK scans the JSON stylesheet in a cascade order to find style rules matching the query’s tags. 

As a general rule, stylesheet scanning involves searching for any applicable style rule that contributes to formatting the UI elements on the screen. Hence, multiple style rules may participate in styling the same UI element. 

If multiple style rules reference the same single tag or any tag item of a tag group but specify complementary properties, all rules apply to styling the UI element. 

Instead, if multiple applicable style rules refer the same property, the last applicable rule of the stylesheet file overrides the previous ones. 

**Example**

In the stylesheet above, for an API query referencing the `["list-item", "hover"]` tag group, multiple style rules are applied.

- Second rule’s `tags` items match both tag items in the query, and it qualifies the `BackgroundColor` property.
- First rule’s`tags` item matches the "list-item" tag item, and it qualifies the `tintColor` property.

Since both properties qualify the "list-item" tag item, both rules are applied. Note that, although both syles reference the `BackgroundColor` property, the BackgroundColor style of the second rule takes precedence because it is the last one referenced in the stylesheet.

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

