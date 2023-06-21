# GetStyle API

## Overview
The Transmit SDK GetStyle API enables the customization of the appearance of screens during the execution of Transmit flows. Specifically, at runtime, the API retrieves styles from a JSON stylesheet and applies them to the screen's UI elements, similar to applying styles from a CSS stylesheet. The JSON stylesheet is stored on the client side and can be customized to reflect the client brand colors, ensuring that the Transmit flow screen remains consistent with the client's website visual identity. 

## Tags

Both the API query and the JSON stylesheet uses _tags_ to identify each UI element. A tag may consist of a single tag or a group of tags, with each tag representing a specific characteristic or feature of the UI element, such as its type, context of use, state, and others. The larger the tag set is, the more specific both the UI element usage and the corresponding rule's case of application are. The example above shows the possible structure of tags:

| UI element                       | Tag                              |
| -------------------------------- | -------------------------------- |
| H1 title                         | [`title`]                      |
| Authentication screens’ H1 title | [`title`, `password-screen`]     |
| Save button                      | [`button`, `save-button`]        |
| Save button (hover)              | [`button`, `save-button`, `state`] |

## JSON stylesheet 

### Structure

The JSON stylesheet consists of a collection of _style rules_, and styling _values_. 

_Style rules_ are flexible structures that can be designed to respond to specific contexts of use. The more specific the context to use a style is, the more complex the style rule becomes. For instance, a style rule may target the hover state of a validation button on a specific screen.

Each style rule includes:
- a _rule_ that identifies UI element types, characteristics, or features using _tags_ (see [`tags` field](#fields)) and thus determines the conditions for its application.
- a set of _properties_ that either identify formatting styles (see [`properties` field](#fields)), or refer to styling _values_.

_Values_, are a centralized collection of  styling properties intended to be applied by reference and reused in multiple contexts. For example, `values` properties may identify the brand identity colors to be used as the  main color palette for the website's backgrounds, page titles, menu items, an mode. Centralizing formatting styles within the `values` object ensures consistency and easy maintenance of the stylesheet. 

```
styles
    style
        rule
            tags
        properties
    style
        rule
            tags
        properties
    ...
    values
        styling property 1
        styling property 2
        ...
```
The JSON stylesheet is stored on the client side and then customizable by the client.

### Fields

| Field                | Nested in            | Description                                             | Data type | Mandatory |
| -------------------- | -------------------- | ------------------------------------------------------- | --------- | --------- |
| `styles `          | -                   | Collection of style rules                                  | array     | YES       |
| `rule`             | `styles `          | Style rule that defines the condition to apply the rule (see `tags`), and the rule's style properties (see `properties`)  | object    | YES       |
| `tags `            | `rule `            | Tag that specifies the UI elements to style                                     | array     | YES       |
| `properties`       | `rule `            | Formatting properties to style the UI elements. It can either specify a CSS formatting property o reference a styling variable (see `values`).          | object    | YES       |
| `values `          | -                  | Collection of styling variables that can be referenced by the properties of style rules         | object    | YES       |

## Query processing
At query time, the GetStyle API queries the JSON stylesheet with the tags that identify the screen’s UI elements. The SDK scans then the JSON stylesheet in cascade order to find style rules that contribute to formatting the UI elements. 

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
