---
title: 'User Interaction'
Weight: 900
---

## Interacting with a Sprotty Diagram
Sprotty offers multiple mouse and keyboard interactions by default listed in the following tables.
The `CTRL` key in the following is equal to the `CMD` key on Mac.

| Mouse       
| ----------------- | ---- |
| left-click:       | toggle selection |
| CTRL-click:       | add to/remove from selection |
| left-click : drag | move selection (when on selected element) |
| left-click : drag | pan viewport (when on diagram background) |
| mouse wheel:    | zoom |

| Touchpad
| --------------------- | --- |
|click                  | toggle selection |
|drag                   | move selection (when on selected element) |
|two-finger pan up/down | zoom |
|zoom                   | zoom |

| Keys
| ---------- | --- |
|CTRL-SHIFT-A| select all |
|CTRL-SHIFT-C| center selection, or if nothing is selected center the entire diagram |
|CTRL-SHIFT-E| export diagram to SVG |
|CTRL-SHIFT-F| zoom selection to fill the entire canvas, or if nothing is selected zoom the entire diagram |
|CTRL-Z      | undo |
|CTRL-SHIFT-Z| redo |

## Creating Custom Interaction

### Buttons and Button Handlers
The easiest way to create custom interactions is through buttons. Buttons in Sprotty work similarly to the other model elements. 
As in the other examples with other types of nodes, we first need to define the model and view class and define its type ID. in addition, we also need to define a button handler of type `IButtonHandler`.

```Typescript
const container = new ContainerModule((bind, unbind, isBound, rebind) => { 
        const context = { bind, unbind, isBound, rebind };
        configureModelElement(context, 'graph', SGraph, SGraphView);
        configureModelElement(context, 'button:custom', SButton, CustomButtonView);

        configureButtonHandler({bind, isBound}, 'button:custom', CustomButtonHandler);

        configureViewerOptions(context, {
            needsClientLayout: true,
            baseDiv: 'sprotty'
        });

    });
```

A button handler is a simple injectable class with a `buttonPressed(button: SButton): Action[]` method. 
returned actions are passed to the `ActionDispatcher` to be handled there. 
```Typescript
@injectable()
export class CustomButtonHandler implements IButtonHandler {
    buttonPressed(button: SButton): Action[] {
        alert('button on' + button.parent.id + ' pressed');
        return [];
    }
}
```

### Mouse and Keyboard Listeners
Sprotty also offers the ability to attach mouse and keyboard listeners by registering `MouseListener` or `KeyListener`.
This can be simply done by binding the custom listener to the respective listener type in your DI-container like this:
```Typescript
    bind(CustomMouseListener).toSelf().inSingletonScope();
    bind(TYPES.MouseListener).toService(CustomMouseListener);
```
These listeners are global, meaning they react to every interaction with the diagram. A very simple mouse Listener would look like this: 
```Typescript
export class CustomMouseListener extends MouseListener {
    doubleClick(target: SModelElement, event: MouseEvent): (Action | Promise<Action>)[] {
        alert('double clicked ' + target.id)
        return [];
    }
}
```
If only specific types of nodes are supposed to be interacted with through this custom listener, it's recommended to [create a custom feature](no-link-yet).

### Projection Bars
Another interactive Sprotty feature is the `ProjectedViewportView`.
This view automatically adds a vertical and horizontal scrollbar to our diagram view in which our current viewport is shown, like in the following image.

![projection bar example](/projection_bar_example.png)

To activate this feature, instead of using `SGraph` and `SGraphView` for our root element, as we did in the other examples, we use `ViewportRootElement` and `ProjectedViewportView`.
The scrollbars created through `ProjectedViewportView` can be styled via the `.sprotty-viewport` and `.sprotty-projection-bar` CSS classes. 


As we can see in the image, these scrollbars can contain projections of our nodes, which show their horizontal and vertical position. 
Double-clicking on a projection will center the view on the node the projection belongs to. 

To add projections we add the `Projectable` type to the nodes in our model schema like this:
```Typescript
 <SNode & Projectable>{
            id: 'svg',
            type: 'node:svg',
            text: '',
            projectionCssClasses: ['svg-projection']
}
```
This requires us to add the `projectionCssClasses` property to the node which the view can evaluate to style the projections. We need at least one class, otherwise, no projection will be created.
The projections are simple HTML `div` elements, which can be styled through the classes we set through `projectionCssClasses`. 

