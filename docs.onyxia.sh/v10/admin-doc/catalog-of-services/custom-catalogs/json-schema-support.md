---
description: This section describes JSON Schema support in the launcher.
hidden: true
icon: bagel
---

# JSON Schema Support

Onyxia uses JSON Schema to dynamically create its service launch interface, often referred to as the "launcher." By defining parameters and configurations in JSON Schema, Onyxia can automatically generate forms and interfaces that guide users through setting up and deploying services.

The JSON Schema draft Onyxia follows is largely based on [Draft 7](https://json-schema.org/specification-links.html#draft-7), but it only implements a subset of the specification. This means that while Onyxia’s schema supports many core features of Draft 7—like data types, required fields, and basic validations—it may not include every feature or validation option found in the full Draft 7 specification. This subset approach keeps the schema manageable and efficient for the specific needs of Onyxia's interface generation and deployment configurations. In the following section, you’ll also see that Onyxia adds additional semantic layers.

## **Summary**

* **String**: Supports plain text input
* **Number / Integer**: Allows numerical input.
* **Boolean**: Renders a toggle.
* **Array**: Supported for list-like inputs, often used for specifying multiple items (e.g., environments, tags). Onyxia provide a way to add or remove item.
  * **Items**: Onyxia supports homogenous arrays, where all items are expected to be of the same type.
* **Object**: Forms the basis for grouping multiple fields together.
  * **Properties**: Each property in an object renders as an individual input element within the launcher.

## String

#### Render

In Onyxia’s JSON Schema implementation, string elements include various `render` types to adjust the input style based on each field’s function, creating a more intuitive user experience. Here are the primary `render` types supported for string fields:

1.  **Dropdown Selection (`render: "list"`)**: Displays a dropdown menu for selecting from a set of predefined values, which is useful for fields like software versions or configurations.

    Example with schema validation :

    ```json
    {
      "type": "string",
      "enum": ["version1", "version2", "version3"],
      "default": "version1",
      "description": "Choose a software version"
    }
    ```

    Example without schema validation (usefull if your chart are reused in other context and you want people to specify other value):

    ```json
    {
      "type": "string",
      "render": "list",
      "listEnum": ["version1", "version2", "version3"], // this is onyxia specification
      "default": "version1",
      "description": "Choose a software version"
    }
    ```
2.  **Password Field (`render: "password"`)**: Provides a masked input field to secure sensitive data, such as passwords or API keys.

    ```json
    {
      "type": "string",
      "render": "password",
      "description": "Enter your API key"
    }
    ```
3.  **Multi-line Text (`render: "textArea"`)**: Creates a resizable, multi-line text box for longer text entries, such as configuration scripts or notes, enhancing readability and usability.

    ```json
    {
      "type": "string",
      "render": "textArea",
      "description": "Enter configuration details\n Thank you!"
    }
    ```
4.  **Slider (`render: "slider"`)**: For numeric inputs (stored as strings), `render: "slider"` allows users to select a value within a specified range using a slider, commonly used for resource allocation (e.g., CPU, memory). This includes additional attributes like `sliderMin`, `sliderMax`, `sliderStep`, and `sliderUnit` to configure the slider's behavior.

    ```json
    {
      "type": "string",
      "render": "slider",
      "sliderMin": 50,
      "sliderMax": 40000,
      "sliderStep": 50,
      "sliderUnit": "m",
      "description": "Set the CPU limit"
    }
    ```



Onyxia also define some extention to the JSON Schema standard in order to let you pre-fill some values levraging what we know about the user. &#x20;

{% content-ref url="onyxia-extension.md" %}
[onyxia-extension.md](onyxia-extension.md)
{% endcontent-ref %}
