This blueprint acts as a tutorial for creating your first Mendix Pluggable Widget. The end result will be a simple textbox that can modify a string attribute.
# Install tools
 - Install the LTS version of [Node.js](https://nodejs.org/en/download/).
 - Install Yeoman:
	 - `npm install -g yo`
 - Install the Mendix Pluggable Widget Generator:
	 - `npm install -g @mendix/generator-widget`
 - Install an IDE of choice (recommended: [Microsoft Visual Studio Code](https://code.visualstudio.com/))

# Creating widget scaffolding
The Pluggable Widget Generator is the quickest way to start developing a widget as it builds the recommended folder structure and files for you.

 - Navigate to your project's app directory, create a new folder `CustomWidgets` and open in Powershell.
 - Run the generator:
	 - `yo @mendix/widget <Widget_Name>`
 - Fill in the information requested; the following should be completed as specified:
	 - **Programming language**: Typescript
	 - **Which type of components do you want to use**: Class components
	 - **Widget type**: For web mobile apps
	 - **Widget template**: Empty widget
	 - **Unit tests**: No *(We will create these ourselves)*
	 - **End-to-end tests**: No

# Building the widget
 - Open the newly created folder (`<MendixApp>/CustomWidgets/<Widget_Name>`) in your IDE. All file references will now be relative to this path. 
 - Remove the file `src/components/HelloWorldSample.tsx`.
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
	- The generator will have created a default `sampleText` property which you should remove.
	- Create a new property as follows:
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
- Run `npm run build` to compile the widget.
- In Mendix, press F4 to refresh the project folder.
- In the toolbox, you will see your newly created widget. You can add it into a page inside a dataview. You will be able to enter text into the widget, however the data will not be persisted. We will fix this in the next step.
### 6. Events
 - As we want to be able to persist data entered into the text box, we need to tell Mendix to save changes to the database. This can be achieved with events.
 - Similar to [[#2. Attributes]], open `src/<Widget_Name>.xml` and add the following:
```xml
<propertyGroup caption="Events">
    <property key="onChangeAction" type="action" required="false">
        <caption>On change</caption>
        <description/>
    </property>
</propertyGroup>
```
 - Use `npm run build` once again to update the widget, and when done press F4 in Studio Pro.
 - Right click the widget, and select Update widget.
 - You will be able to select an event which will happen when onChange is fired. For now set it to **Save changes**.
 - We now need to trigger this event by changing the value of our props. Open the container component created in [[#4. Containers]], and modify the `render` function as follows:
```tsx
return <TextInput
	value={value}
	onLeave={this.onLeaveHandle}
/>
```
- Define the `onLeaveHandle` function inside the class as follows:
```tsx
private onLeave(value: string, isChanged: boolean): void {
	if (!isChanged) {
		return;
	}
	this.props.textAttribute.setValue(value);
}
```
- Our display component, created in [[#3. Components]] should be updated to call the `onLeave` callback. Open the file and add modify as follows:
```tsx
export interface InputProps {
    value: string;
    onLeave?: (value: string, changed: boolean) => void;
}
interface InputState {
    editedValue?: string;
}
export class TextInput extends Component<InputProps, InputState> {
    private readonly onBlurHandle = this.onBlur.bind(this);
    readonly state: InputState = { editedValue: undefined };
    
    componentDidUpdate(prevProps: InputProps): void {
        if (this.props.value !== prevProps.value) {
            this.setState({ editedValue: undefined });
        }
    }
    
    render(): ReactNode {
        return <input
            type="text"
            value={this.getCurrentValue()}
            onBlur={this.onBlurHandle}
        />;
    }
    
    private getCurrentValue(): string {
        return this.state.editedValue !== undefined
            ? this.state.editedValue
            : this.props.value;
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
