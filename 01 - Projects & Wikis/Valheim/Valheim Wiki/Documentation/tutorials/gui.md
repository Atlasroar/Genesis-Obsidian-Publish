# GUI

To add custom GUI elements to the game it is necessary to add the prefabs or generate the GUI components in code respecting the [Unity UI guidelines](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/index.html).

## CustomGUI event and GameObjects

Valheim creates new clones of the whole menu and ingame GUI everytime the scene changes from start to main and vice versa. So if you dont want to create and draw on your own canvas, you have to add your custom stuff to the right path on every scene change, too. Additionally, Valheim implemented a scaling feature for high resolution display (4K output or high DPI screens) and a pixel correction component called the PixelFix. These concepts can be accessed easily through Jötunn. You can subscribe to the event [GUIManager.OnCustomGUIAvailable](xref:Jotunn.Managers.GUIManager.OnCustomGUIAvailable) which gets called everytime the scene changed and Jötunn has created and resolved the current GUI in the scene (previously called [GUIManager.OnPixelFixCreated](xref:Jotunn.Managers.GUIManager.OnPixelFixCreated) which is deprecated now). In that event call you can load / create your custom GUI components and add them in the transform hierarchy under either the [GUIManager.CustomGUIBack](xref:Jotunn.Managers.GUIManager.CustomGUIBack) or the [GUIManager.CustomGUIFront](xref:Jotunn.Managers.GUIManager.CustomGUIFront) GameObject (the latter being equivalent to the former [GUIManager.PixelFix](xref:Jotunn.Managers.GUIManager.PixelFix) GameObject which is now deprecated). The first resides before Valheim's own GUI elements in the transform hierarchy which means the GUI elements added to this GameObject will be drawn *before* the vanilla GUI and therefore appear *behind* of it. The latter is drawn *after* Valheim's GUI elements and therefore your custom GUI will be in *front* of Valheim's GUI.

## Block input for GUI

When drawing GUI elements Valheim does not stop interpreting the Mouse/Keyboard inputs in any way per default. So if you want to interact with your GUI, you would have to interrupt the receiving of input for the player, camera, etc. Jötunn provides a shortcut to enable or disable player and camera input as the convenient method [GUIManager.BlockInput(bool)](xref:Jotunn.Managers.GUIManager.BlockInput(System.Boolean)). When passing `true`, all input to the player is intercepted and the mouse is released from the camera so you can actually click on your custom GUI elements. You **must** call the method passing `false` again to release the input yourself upon closing your GUI components. Jötunn does not handle the release automatically.

## Determine a headless instance

A dedicated server running without GUI is commonly referred to as a headless server. Valheim provides methods in ZNet to determine if the current instance is a dedicated server, a "local" server (aka local game that others can connect to) or a client to another server. Jötunn also provides shortcuts to these via [ZNetExtension](xref:Jotunn.ZNetExtension). Problem is that both approaches require ZNet to be instantiated which is not the case on your mods Awake(). If you need that information early on, the GUIManager provides a method for that: [GUIManager.IsHeadless()](xref:Jotunn.Managers.GUIManager.IsHeadless) returns true if the current game instance is a dedicated/headless server without relying on ZNet. Jötunn also does not register GUI or Input hooks in that case to save on unnecessarily allocated resources.

## Valheim style GUI elements

The [GUIManager](xref:Jotunn.Managers.GUIManager) provides useful methods to create buttons, text element and more at runtime using the original Valheim assets to create a seamless look for your custom GUI components.

If you dont want to create new GameObjects from scratch, there is also the possibility to apply Jötunn style to your existing GUI controls. Check out the [API docs](xref:Jotunn.Managers.GUIManager) to learn more.

### ColorPicker and GradientPicker

![ColorPicker and GradientPicker](../images/data/colorgradientpicker.png)

Jötunn uses a custom made ColorPicker for its ModSettings dialogue which is also available for mods to use. Additionaly there is also a GradientPicker available.

ColorPickerExample:
```cs
GUIManager.Instance.CreateColorPicker(
    new Vector2(0.5f, 0.5f), new Vector2(0.5f, 0.5f), new Vector2(0.5f, 0.5f),
    r.sharedMaterial.color, // Initial selected color in the picker
    "Choose your poison",   // Caption of the picker window
    SetColor,               // Callback delegate when the color in the picker changes
    ColorChosen,            // Callback delegate when the window is closed
    true                    // Whether or not the alpha channel should be editable
);
```

GradientPicker example:
```cs
GUIManager.Instance.CreateGradientPicker(
    new Vector2(0.5f, 0.5f), new Vector2(0.5f, 0.5f), new Vector2(0, 0),
    new Gradient(),  // Initial gradient being used
    "Gradiwut?",     // Caption of the GradientPicker window
    SetGradient,     // Callback delegate when the gradient changes
    GradientFinished // Callback delegate when thw window is closed
);
```

A more explanatory example can be found in our [example mod](https://github.com/Valheim-Modding/JotunnModExample).

### Wood panels

![Woodpanel](../images/data/woodpanel.png)

Woodpanels, nicely usable as containers for other gui elements. Can automatically add the draggable Component to the panel object (default: true).

Example:
```cs
var panel = GUIManager.Instance.CreateWoodpanel(
    parent: GUIManager.CustomGUIFront.transform, 
    anchorMin: new Vector2(0.5f, 0.5f), 
    anchorMax: new Vector2(0.5f, 0.5f), 
    position: new Vector2(0f, 0f), 
    width: 400f,
    height: 300f,
    draggable: true);
```

### Text elements

![Text Element](../images/data/text-element.png)

To create a text element, provide text, the parent's transform, min and max anchors, the position and it's size (width and height). You can also let Jötunn add a ContentSizeFitter Component so the text element will have its bounds automatically adjusted to its content.

Example:
```cs
var textObject = GUIManager.Instance.CreateText(
    text: "Jötunn, the Valheim Lib",
    parent: TestPanel.transform,
    anchorMin: new Vector2(0.5f, 1f),
    anchorMax: new Vector2(0.5f, 1f),
    position: new Vector2(0f, -100f),
    font: GUIManager.Instance.AveriaSerifBold,
    fontSize: 30,
    color: GUIManager.Instance.ValheimOrange,
    outline: true,
    outlineColor: Color.black,
    width: 350f,
    height: 40f,
    addContentSizeFitter: false);
```

### Buttons

![GUI Button](../images/data/test-button.png)

To create buttons, provide text, the parent's transform, min and max anchors, the position and it's size (width and height).

Example:
```cs
var buttonObject = GUIManager.Instance.CreateButton(
    text: "A Test Button",
    parent: TestPanel.transform,
    anchorMin: new Vector2(0.5f, 0.5f),
    anchorMax: new Vector2(0.5f, 0.5f),
    position: new Vector2(0, -250f),
    width: 250f,
    height: 60f);
```

### Input Fields

![Input Field](../images/data/test-input.png) ![Input Field with text](../images/data/test-input-filled.png)

To create an input field, provide text, the parent's transform, min and max anchors and the position. Optional parameters are the content type (using Unity's builtin types), a placeholder text (can be null), font size (default 16) and the field's actual size (width and height).

Example:
```cs
GUIManager.Instance.CreateInputField(
    parent: TestPanel.transform,
    anchorMin: new Vector2(0.5f, 0.5f),
    anchorMax: new Vector2(0.5f, 0.5f),
    position: new Vector2(250f, -250f),
    contentType: InputField.ContentType.Standard,
    placeholderText: "input...",
    fontSize: 16,
    width: 160f,
    height: 30f);
```

### Checkboxes

![Checkbox](../images/data/checkbox.png)

Example:
```cs
var checkbox = GUIManager.Instance.CreateToggle(
    parent: GUIManager.CustomGUIFront.transform,
    width: 40f,
    height: 40f);
```

### Dropdown Fields

![Input Field](../images/data/test-dropdown.png)

To create a dropdown field, provide the parent's transform, min and max anchors, the position and optionally font size (default 16) and the dropdown's actual size (width and height). After creating the object through the GUIManager you need to provide selectable values. For that you get the Dropdown component of the GameObject and feed your options with the AddOptions() method.

Example:
```cs
var dropdownObject = GUIManager.Instance.CreateDropDown(
    parent: TestPanel.transform,
    anchorMin: new Vector2(0.5f, 0.5f),
    anchorMax: new Vector2(0.5f, 0.5f),
    position: new Vector2(-250f, -250f),
    fontSize: 16,
    width: 100f,
    height: 30f);
dropdownObject.GetComponent<Dropdown>().AddOptions(new List<string>
{
    "bla", "blubb", "börks", "blarp", "harhar"
});
```

### Getting sprites

Gets sprites from the UIAtlas or IconAtlas by name. You find a list of the sprite names [here](../data/gui/sprite-list.md).

```cs
var sprite = GUIManager.Instance.GetSprite("text_field");
```

### Instance properties

The [GUIManager](xref:Jotunn.Managers.GUIManager) also comes with some useful instance properties for your custom assets to resemble the vanilla Valheim style.

- Font AveriaSerif
- Font AveriaSerifBold (the default Valheim font)
- Color ValheimOrange
- ColorBlock ValheimScrollbarHandleColorBlock
- ColorBlock ValheimToggleColorBlock
- ColorBlock ValheimButtonColorBlock

## Custom GUI Components

Jötunn provides helper Components for mods to use when handling with custom GUI.

### Draggable windows

[DragWindowCntrl](xref:Jotunn.GUI.DragWindowCntrl) is a simple Unity Component to make GUI elements draggable with the mouse. Does respect the window limits. To apply it to your own GameObject, simply call the static factory method:

```cs
// Add the Jötunn draggable Component to the panel
DragWindowCntrl.ApplyDragWindowCntrl(TestPanel);
```

## Example

In our [example mod](https://github.com/Valheim-Modding/JotunnModExample) we use a [custom button](inputs.md) to toggle a simple, draggable panel with the Jötunn provided GUI controls on it. The button also gets a listener added to close the panel again. Also the input is blocked for the player and camera while the panel is active so we can actually use the mouse and cant control the player any more.

```cs
// Toggle our test panel with button
private void TogglePanel()
{
    // Create the panel if it does not exist
    if (!TestPanel)
    {
        if (GUIManager.Instance == null)
        {
            Logger.LogError("GUIManager instance is null");
            return;
        }

        if (!GUIManager.CustomGUIFront)
        {
            Logger.LogError("GUIManager CustomGUI is null");
            return;
        }

        // Create the panel object
        TestPanel = GUIManager.Instance.CreateWoodpanel(
            parent: GUIManager.CustomGUIFront.transform,
            anchorMin: new Vector2(0.5f, 0.5f),
            anchorMax: new Vector2(0.5f, 0.5f),
            position: new Vector2(0, 0),
            width: 850,
            height: 600,
            draggable: false);
        TestPanel.SetActive(false);

        // Add the Jötunn draggable Component to the panel
        // Note: This is normally automatically added when using CreateWoodpanel()
        DragWindowCntrl.ApplyDragWindowCntrl(TestPanel);

        // Create the text object
        GUIManager.Instance.CreateText(
            text: "Jötunn, the Valheim Lib",
            parent: TestPanel.transform,
            anchorMin: new Vector2(0.5f, 1f),
            anchorMax: new Vector2(0.5f, 1f),
            position: new Vector2(0f, -50f),
            font: GUIManager.Instance.AveriaSerifBold,
            fontSize: 30,
            color: GUIManager.Instance.ValheimOrange,
            outline: true,
            outlineColor: Color.black,
            width: 350f,
            height: 40f,
            addContentSizeFitter: false);

        // Create the button object
        GameObject buttonObject = GUIManager.Instance.CreateButton(
            text: "A Test Button - long dong schlongsen text",
            parent: TestPanel.transform,
            anchorMin: new Vector2(0.5f, 0.5f),
            anchorMax: new Vector2(0.5f, 0.5f),
            position: new Vector2(0, -250f),
            width: 250f,
            height: 60f);
        buttonObject.SetActive(true);

        // Add a listener to the button to close the panel again
        Button button = buttonObject.GetComponent<Button>();
        button.onClick.AddListener(TogglePanel);

        // Create a dropdown
        var dropdownObject = GUIManager.Instance.CreateDropDown(
            parent: TestPanel.transform,
            anchorMin: new Vector2(0.5f, 0.5f),
            anchorMax: new Vector2(0.5f, 0.5f),
            position: new Vector2(-250f, -250f),
            fontSize: 16,
            width: 100f,
            height: 30f);
        dropdownObject.GetComponent<Dropdown>().AddOptions(new List<string>
        {
            "bla", "blubb", "börks", "blarp", "harhar"
        });

        // Create an input field
        GUIManager.Instance.CreateInputField(
            parent: TestPanel.transform,
            anchorMin: new Vector2(0.5f, 0.5f),
            anchorMax: new Vector2(0.5f, 0.5f),
            position: new Vector2(250f, -250f),
            contentType: InputField.ContentType.Standard,
            placeholderText: "input...",
            fontSize: 16,
            width: 160f,
            height: 30f);
    }

    // Switch the current state
    bool state = !TestPanel.activeSelf;

    // Set the active state of the panel
    TestPanel.SetActive(state);

    // Toggle input for the player and camera while displaying the GUI
    GUIManager.BlockInput(state);
}
```

![Test Panel Example](../images/data/test-complete.png)