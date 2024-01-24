This blueprint acts as a tutorial for creating your first Mendix Pluggable Widget. The end result will be a simple textbox that can modify a string attribute.
# Install tools
 - Install the LTS version of [Node.js](https://nodejs.org/en/download/).
 - Install Yeoman (to do this, you can either use the built-in Visual Studio Code terminal, Windows Powershell, or Windows Command Prompt):
	 - `npm install -g yo`
 - Install the Mendix Pluggable Widget Generator:
	 - `npm install -g @mendix/generator-widget`
 - Install an IDE of choice (recommended: [Microsoft Visual Studio Code](https://code.visualstudio.com/))

# Creating widget scaffolding
The Pluggable Widget Generator is the quickest way to start developing a widget as it builds the recommended folder structure and files for you.

 - Navigate to your project's app root directory. You can navigate to this directory by opening it in the Windows file explorer, or by opening your project in Mendix Studio Pro and selecting App -> Show app directory in Explorer. In your project's app root directory, create a new folder called `CustomWidgets` and open it in Visual Studio Code, Windows Powershell, or Windows Command Prompt.
 - Run the generator:
	 - `yo @mendix/widget <Widget_Name>`
 - Fill in the information requested; the following should be completed as specified:
	 - **What is the name of your widget?**: <Widget_Name>
  	 - **Enter a description for your widget**: <Widget_Description>
	 - **Organization name**: Finaps
	 - **Add a copyright**: *Default*
	 - **Add a license**: *Default*
	 - **Initial version**: 1.0.0
	 - **Author**: <Your_Name>
 	 - **Mendix project path**: <Project_Root_Directory>
	 - **Which programming language do you want to use for the widget?**: Typescript
	 - **Which type of components do you want to use?**: Class Components
	 - **Which type of widget are you developing?**: For web and hybrid mobile apps
	 - **Which template do you want to use for the widget?**: Empty widget
	 - **Add unit tests for the widget?**: No *(We will create these ourselves)*
	 - **Add End-to-end tests for the widget?**: No

# Building the widget
 - Open the newly created folder (`<MendixApp>/CustomWidgets/<Widget_Name>`) in your IDE. All file references will now be relative to this path. 
 - Remove the file `src/components/HelloWorldSample.tsx`.
 - Remove the `import { HelloWorldSample } from "./components/HelloWorldSample";` line from `src/<Widget_Name>.editorPreview.tsx` and `src/<Widget_Name>.tsx`.
### 1. Editor preview 
 - Open `src/<Widget_Name>.editorPreview.tsx`. This file tells Mendix how to render the preview of your widget in the modeler when in Design view.
	 - The class `preview` should contain a render() function which returns a `ReactNode` as below. This can behave in any way you see fit, however as Design view is rarely used, simple text will suffice.
```tsx
render(): ReactNode {
	return <p>My widget</p>
}
```
### 2. Attributes 
 - Open `src/<Widget_Name>.xml`. This is the widget definition file which Studio Pro uses to interpret the widget's capabilities.
	- The generator will have created a default `sampleText` property which you should remove. The to-be-deleted property looks like this:
```xml
	<property key="sampleText" type="string" required="false">
		<caption>Default value</caption>
                <description>Sample text input</description>
	</property>
```
- Create a new property group as follows by placing it between the existing `<propertygroup>` open and close tags:
```xml
<propertyGroup caption="Data source">
	<property key="textAttribute" type="attribute" required="true">
		<caption>Text</caption>
		<description>A string attribute that the text box will read/write</description>
		<attributeTypes>
			<attributeType name="String"/>
		</attributeTypes>
	</property>
</propertyGroup>
```
 - This will give the widget a new property which can be selected in the widget's properties screen in Studio Pro.
### 3. Components
 - Create a new file `components/TextInput.tsx` which will be the component displayed on the page. A component does not interact with APIs and can be re-used in any React application. Add the following code:
```tsx
import { Component, ReactNode, createElement } from "react";

export interface InputProps {
    value: string;
}

export class TextInput extends Component<InputProps> {
    render(): ReactNode {
        return <input type="text" value={this.props.value} />;
    }
}
```
 - The interface `InputProps` defines the properties of the React components — the value is passed to the component and it will render a HTML input element with the given value.
 - The component `TextInput` is a class extending `Component` and should be exported to be used in other components.
 - The `render` function is the only required function in a component, and it will return the expected DOM for the browser
### 4. Containers
 - The generator already created a container component `<Widget_Name>.tsx`. This receives the properties from the Mendix runtime, and passes them on to the display component created above. 
 - Open the container's file, and import our newly created component:
```tsx
import { TextInput } from "./components/TextInput";
```
 - Modify the render function as follows:
```tsx
render(): ReactNode {
	const value = this.props.textAttribute.value || "";
	return <TextInput value={value} />;
}
```
 - `this.props.textAttribute`, created in [[#2. Attributes]], will contain the data stored in the attribute. When the data is changed, it will cause an update of the component and the new data will be displayed in the text box.
 - The `render` function will now return our TextBox input, passing along the text attribute.
### 5. Using the widget
- You can ignore the red squiggly lines underneath any code for now, these will disappear as a result of the next step.
- Run `npm run build` to compile the widget.
- In Mendix, press F4 to refresh the project folder.
- In the toolbox, you will see your newly created widget. You can add it into a page inside a dataview. However, you will not yet be able to enter text into the widget, and the data will not be persisted. We will fix this in the next step.
### 6. Events
 - At this point, the value from the attribute can be displayed and updated using a standard Mendix text input, but you can not yet change the value directly from within your widget. We will fix this by handling updates.
 - Create a function that will update the attribute and pass it to the TextInput component. Do this by opening `src/<Widget_Name>.tsx` and replacing the code within the class with the following:
```tsx
private readonly onUpdateHandle = this.onUpdate.bind(this);
render(): ReactNode {
	const value = this.props.textAttribute.value || "";
        return <TextInput
            value={value}
            style={this.props.style}
            className={this.props.class}
            tabIndex={this.props.tabIndex}
            onUpdate={this.onUpdateHandle}
        />;
    }
private onUpdate(value: string): void {
this.props.textAttribute.setValue(value);
}
```
- In `src/components/TextInput.tsx`, the change events of the input need to be handled and the new value needs to be passed to the `onUpdate` function of the container component. To accomplish this, replace the existing code with the following:
```tsx
import { CSSProperties, ChangeEvent, Component, ReactNode, createElement } from "react";
import classNames from "classnames";
export interface InputProps {
    value: string;
    className?: string;
    index?: number;
    style?: CSSProperties;
    tabIndex?: number;
    onUpdate?: (value: string) => void;
}
export class TextInput extends Component<InputProps> {
    private readonly handleChange = this.onChange.bind(this);
    render(): ReactNode {
        const className = classNames("form-control", this.props.className);
        return <input
            type="text"
            className={className}
            style={this.props.style}
            value={this.props.value}
            tabIndex={this.props.tabIndex}
            onChange={this.handleChange}
        />;
    }
    private onChange(event: ChangeEvent<HTMLInputElement>): void {
        if (this.props.onUpdate) {
            this.props.onUpdate(event.target.value);
        }
    }
}
```
- The input’s value is set by the `this.props.value`. This property is not changed directly; the update function will use the setValue to trigger a re-render with the updated property. For more information on rendering, read through the following React documentation: [React Render and Commit](https://react.dev/learn/render-and-commit).
- It is now possible to enter text input in your custom widget, but it will still not persist the input at this point.
- As we want to be able to persist data entered into the text box, we need to tell Mendix to save changes to the database. This can be achieved with events.
 - Similar to [[#2. Attributes]], open `src/<Widget_Name>.xml` and change the XML between the top and bottom `<properties>` tags to the following:
```xml
<properties>
        <propertyGroup caption="General">
            <propertyGroup caption="Data source">
                <property key="textAttribute" type="attribute" required="true" onChange="onChangeAction">
                    <caption>Text</caption>
                    <description>A string attribute that the text box will read/write</description>
                    <attributeTypes>
                        <attributeType name="String" />
                    </attributeTypes>
                </property>
            </propertyGroup>
        </propertyGroup>
        <propertyGroup caption="Events">
            <property key="onChangeAction" type="action" required="false">
                <caption>On change</caption>
                <description/>
            </property>
        </propertyGroup>
    </properties>
```
 - Use `npm run build` once again to update the widget, and when done press F4 in Studio Pro.
 - Right click the widget, and select Update widget.
 - You will be able to select an event which will happen when onChange is fired. For now set it to **Save changes**.
 - We now need to trigger this event by changing the value of our props. Open the container component (`src/<Widget_Name>.tsx`) created in [[#4. Containers]], and modify the `render` function as follows:
```tsx
return (
            <TextInput
                value={value}
                style={this.props.style}
                className={this.props.class}
                tabIndex={this.props.tabIndex}
                onUpdate={this.onUpdateHandle}
                onLeave={this.onLeaveHandle}
            />
        );
```
- Define the `onLeave` function inside the class as follows:
```tsx
private onLeave(value: string, isChanged: boolean): void {
	if (!isChanged) {
		return;
	}
	this.props.textAttribute.setValue(value);
}
```
- Also, add the following line inside the class to bind the `onLeave` function as follows:
```tsx
private readonly onLeaveHandle = this.onLeave.bind(this);
```
- Our display component (`src/components/TextInput.tsx`), created in [[#3. Components]] should be updated to call the `onLeave` callback. Open the file and add modify as follows:
```tsx
export interface InputProps {
    value: string;
    className?: string;
    index?: number;
    style?: CSSProperties;
    tabIndex?: number;
    onUpdate?: (value: string) => void;
    onLeave?: (value: string, changed: boolean) => void;
}
interface InputState {
    editedValue?: string;
}
export class TextInput extends Component<InputProps, InputState> {
    private readonly handleChange = this.onChange.bind(this);
    private readonly onBlurHandle = this.onBlur.bind(this);
    readonly state: InputState = { editedValue: undefined };
    componentDidUpdate(prevProps: InputProps): void {
        if (this.props.value !== prevProps.value) {
            this.setState({ editedValue: undefined });
        }
    }
    render(): ReactNode {
        const className = classNames("form-control", this.props.className);
        return (
            <input
                type="text"
                className={className}
                style={this.props.style}
                value={this.getCurrentValue()}
                tabIndex={this.props.tabIndex}
                onChange={this.handleChange}
                onBlur={this.onBlurHandle}
            />
        );
    }

    private getCurrentValue(): string {
        return this.state.editedValue !== undefined ? this.state.editedValue : this.props.value;
    }

    private onChange(event: ChangeEvent<HTMLInputElement>): void {
        this.setState({ editedValue: event.target.value });
    }

    private onBlur(): void {
        const inputValue = this.props.value;
        const currentValue = this.getCurrentValue();
        if (this.props.onLeave) {
            this.props.onLeave(currentValue, currentValue !== inputValue);
        }
        this.setState({ editedValue: undefined });
    }
}

```
 - We have made the following changes:
	 1. Added the `onLeave` callback to the `InputProps`.
	 2. Add `InputState` which will store the current contents of the text box.
	 3. Created an `onBlur` callback which is fired when the user exits the text box. It compares the value stored in the props to the current value. If this is different, it will trigger onLeave in the container component.
- Rebuild the widget once again and return to Studio Pro. Run your app and you will find that changes in the text box are now persisted.
# Questions
If you have questions or doubts after reading this document, or struggled with a particular step, reach out to sam.farhan@finaps.nl.
