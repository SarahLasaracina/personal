# GetStyle API

## Overview
The Transmit SDK GetStyle API enables the customization of the appearance of screens during the execution of Transmit flows. Specifically, at runtime, the API retrieves styles from a JSON stylesheet and applies them to the screen's UI elements, similar to applying styles from a CSS stylesheet. The JSON stylesheet is stored on the client side and can be customized to reflect the client brand colors, ensuring that the Transmit flow screen remains consistent with the client's website visual identity. 

## Tags

Both the API query and the JSON stylesheet use _tags_ to identify each UI element. A tag may consist of a single tag or a group of tags, with each tag representing a specific characteristic or feature of the UI element, such as its type, context of use, state, and others. The larger the tag set is, the more specific both the UI element usage and the corresponding rule's case of application are. The example above shows the possible structure of tags:

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
- a set of _properties_ that either identify formatting styles (see [`properties` field](#fields)) or refer to styling _values_.

_Values_, are a centralized collection of styling properties intended to be applied by reference and reused in multiple contexts. For example, `values` properties may identify the brand identity colors to be used as the main color palette for the website's backgrounds, page titles, menu items, and more. Centralizing formatting styles within the `values` object ensures consistency and easy maintenance of the stylesheet. 

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
At query time, the GetStyle API queries the JSON stylesheet with the tags that identify the screen’s UI elements. The SDK then scans the JSON stylesheet in cascade order to find style rules that contribute to formatting the UI elements.

The applicability of style rules is evaluated based on tag matching criteria. 

For each UI element, multiple style rules may apply if they provide either unique or complementary properties.

### Tag matching criterion
The criterion used to evaluate the applicability of a style rule consists of searching for style rules whose tags partially or fully match the query tag. 

More complex tag sets, which refer to more specific UI elements, are ignored. 

When a matching tag is found, the corresponding rule is applicable.

**Example** 
In the following table, "A, B, C" are symbolic placeholders for tags.

| Query tag | Style rule tag | Matching |
|-----------|----------------|----------|
| `[A, B, C]` | `[A]` | YES |
| `[A, B, C]` | `[B]` | YES |
| `[A, B, C]` | `[A, B]` | YES |
| `[A, B, C]` | `[B, C]` | YES |
| `[A, B, C]` | `[A, B, C]` | YES |
| `[A, B, C]` | `[A, B, C, D]` | NO |



### Property application

To ensure complete formatting of the UI elements, all properties of applicable rules apply. If multiple rules qualify the same property, the properties from the last applicable rule in the stylesheet file take precedence and override the previous ones.

The same criterion applies to style rules that refer to styling properties from the `values` collection.


**Example** 
In the following table "A, B, C" are symbolic placeholders for tags and "1, 2, 3" are symbolic placeholders for properties.

| Query tag | Style rule | Query tag properties  |
|-----------|----------------|-----------------------| 
| `[A, B, C]` | `[A]` | `[1, 2]` |
| `[A, B, C]` | `[A, B]` | `[3, 4]` |
| `[A, B, C]` | `[B, C]` | `[5, 6]` |
| `[A, B, C]` | `[A, B, C]` | `[5, 6, 7]` |

The properties displayed at runtime are `1`, `2`, `3`, `4`, `5`, `6`, and `7`. 
Although properties `6` and `7` are qualified by both `[A, B, C]` and `[A, B, C]` style rules, `[A, B, C]`'s properties override `[B, C]`'s, as they're the last ones referenced in the stylesheet.

## In-context example

In the JSON stylesheet above, for an API query referencing the `["list-item", "hover"]` tag, multiple style rules apply:
- First rule applies as its `"list-item"` tag matches the query tag partially.
- Second rule applies as its `["list-item", "hover"]` tag matches the query tag totally.

As both styles reference the `BackgroundColor` property, the BackgroundColor style of the second rule takes precedence because it is the last one referenced in the stylesheet. 

As a result, the properties on screen are:
- First rule's `tintColor`
- Second rule's `BackgroundColor` (note that the property refers to the `primary-color` from the In this example, the `values` collection)

![stylehsheet img](stylesheetexample.JPG)
