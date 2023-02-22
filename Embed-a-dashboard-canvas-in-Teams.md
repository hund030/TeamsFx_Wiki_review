> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

The dashboard tab template from Teams Toolkit enables you to quickly get started with embedding a canvas containing multiple cards that provide an overview of data or content in Microsoft Teams while worrying less about dashboard layout.

## In this tutorial, you will learn

Basic Concepts:
 * [What are Teams tabs](https://learn.microsoft.com/microsoftteams/platform/tabs/what-are-tabs)
 * [App design guidelines for Tab](https://learn.microsoft.com/microsoftteams/platform/tabs/design/tabs)
 * [Fluent UI Library](https://react.fluentui.dev/?path=/docs/concepts-introduction--page) and [Fluent UI React Charting Examples](https://fluentuipr.z22.web.core.windows.net/heads/master/react-charting/demo/index.html#/)

Get started with Teams Toolkit:
 * [How to create a new dashboard tab](#create-a-new-dashboard-project)
 * [How to understand the dashboard tab project](#take-a-tour-of-your-app-source-code)
 * [How to understand the widget abstraction](#widget-abstraction)
 * [How to understand the dashboard abstraction](#dashboard-abstraction)

Customize the scaffolded app template:
 * [How to add a widget](#how-to-add-a-new-widget)
 * [How to add a dashboard](#how-to-add-a-new-dashboard)
 * [How to customize the widget](#customize-the-widget)
 * [How to include a data loader](#how-to-include-a-data-loader)
 * [How to handle empty state](#how-to-handle-empty-state)
 * [How to refresh data as scheduled](#how-to-refresh-data-as-scheduled)
 * [How to use Microsoft Graph Toolkit as widget content](#how-to-use-microsoft-graph-toolkit-as-widget-content)
 * [How to embed Power BI to Dashboard](#how-to-embed-power-bi-to-dashboard)
 * [How to customize the dashboard layout](#customize-the-dashboard-layout)
 * [How to add a Graph API call](#how-to-add-a-new-graph-api-call)

## Create a new dashboard project

### In Visual Studio Code

1. From Teams Toolkit side bar click `Create a new app` or select `Teams: Create a new app` from the command palette.

![image](https://user-images.githubusercontent.com/107838226/220565583-7e6145f1-6f07-4072-b343-b9f76cc63162.png)

2. Select `Create a new Teams app`.

![image](https://user-images.githubusercontent.com/107838226/220566003-54ab7c82-e402-422b-b905-26605e112495.png)

3. Select `Dashboard tab` from Scenario-based Teams app section.

![image](https://user-images.githubusercontent.com/11220663/205845275-40b69e76-6bda-42a5-af7e-0ac74a7357ca.png)

4. Select programming language.

![image](https://user-images.githubusercontent.com/11220663/205845342-ce213cdb-a42a-4f6a-a27d-4a12dc3cb829.png)

5. Select a workspace folder.

![image](https://user-images.githubusercontent.com/11220663/205845434-f44ca178-f481-4674-bb9a-80324e6459df.png)

6. Enter an application name and then press enter.

![image](https://user-images.githubusercontent.com/11220663/205845475-a718966e-8a2b-4300-b454-a569362df67c.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.

* If you prefer non-interactive mode, enter all required parameters in one command. 

`teamsfx new --interactive false --capabilities "dashboard-tab" --programming-language "typescript" --folder "./" --app-name dashboard-cli-001`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. after running this template and you will see a tab app loaded as below:

![dashboard](https://user-images.githubusercontent.com/107838226/212262814-37a2bf02-2801-458f-97f0-af26ea04553c.png)


This app also supported teams different themes, including dark theme and high contrast theme.

|           Dark theme           |     High contrast theme      |
| :----------------------------: | :--------------------------: |
| <img src="https://user-images.githubusercontent.com/107838226/212263051-6e212830-4299-4901-b0f2-d6c6e035aabb.png" /> | <img src="https://user-images.githubusercontent.com/107838226/212263105-450f106f-fee5-40d3-93a6-b7147455fc9d.png" /> |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Take a tour of your app source code

This section walks through the generated code. The project folder contains the following:

| Folder       | Contents                                            |
|--------------|-----------------------------------------------------|
| `.vscode`    | VSCode files for debugging                          |
| `appPackage` | Templates for the Teams application manifest        |
| `env`        | Environment files                                   |
| `infra`      | Templates for provisioning Azure resources          |
| `src`        | The source code for the dashboard Teams application |

The following files provide the business logic for the dashboard tab. These files can be updated to fit your business logic requirements. The default implementation provides a starting point to help you get started.

| File                                       | Contents                                          |
| ------------------------------------------ | ------------------------------------------------- |
| `src/data/ListData.json`                   | The data provided to the list widget              |
| `src/models/listModel.tsx`                 | Data model for the list widget                    |
| `src/services/listService.tsx`             | A data retrive implementation for the list widget |
| `src/views/dashboards/SampleDashboard.tsx` | A sample dashboard layout implementation          |
| `src/views/lib/Dashboard.styles.ts`        | The dashbaord style file                          |
| `src/views/lib/Dashboard.tsx`              | An base class that defines the dashboard          |
| `src/views/lib/Widget.styles.ts`           | The widgt style file                              |
| `src/views/lib/Widget.tsx`                 | An abstract class that defines the widget         |
| `src/views/styles/ChartWidget.styles.ts`   | The chart widget style file                       |
| `src/views/styles/ListWidget.styles.ts`    | The list widget style file                        |
| `src/views/widgets/ChartWidget.tsx`        | A widget implementation that can display a chart  |
| `src/views/widgets/ListWidget.tsx`         | A widget implementation that can display a list   |

The following files are project-related files. You generally will not need to customize these files.

| File                               | Contents                                                     |
|------------------------------------|--------------------------------------------------------------|
| `src/App.tsx`                      | Application route                                            |
| `src/index.tsx`                    | Application entry point                                      |
| `src/index.css`                    | The style of this application                                |
| `src/internal/addNewScopes.ts`     | Implementation of adding new scope                           |
| `src/internal/context.ts`          | TeamsFx Context                                              |
| `src/internal/login.ts`            | Implementation of login                                      |
| `src/internal/singletonContext.ts` | Implementation of the TeamsUserCredential instance singleton |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Widget abstraction

To simplify the development of a widget, the TeamsFx provides a `Widget` class 
for developers to inherit to quickly implement a widget that meets their needs without pay too much attention to how to implement the widget layout.

Below is the definition of the Widget class.

```ts
export abstract class Widget<T> extends Component<any, { data?: T | void }> {
  constructor(props: any) {
    super(props);
    this.state = {
      data: undefined,
    };
  }

  /**
   * This method is invoked immediately after a component is mounted.
   * It's a good place to fetch data from server.
   */
  async componentDidMount() {
    this.setState({ data: await this.getData() });
  }

  /**
   * Define your widget layout, you can edit the code here to customize your widget.
   */
  render() {
    return (
      <div style={widgetStyles()}>
        {this.headerContent() && (
          <div style={headerStyles()}>{this.headerContent()}</div>
        )}
        {this.bodyContent() && <div>{this.bodyContent()}</div>}
        {this.footerContent() && <div>{this.footerContent()}</div>}
      </div>
    );
  }

  /**
   * Get data required by the widget, you can get data from a api call or static data stored in a file. Override this method according to your needs.
   * @returns data for the widget
   */
  protected async getData(): Promise<T> {
    return new Promise<T>(() => {});
  }

  /**
   * Override this method to customize the widget header.
   * @returns JSX component for the widget body
   */
  protected headerContent(): JSX.Element | undefined {
    return undefined;
  }

  /**
   * Override this method to customize the widget body.
   * @returns JSX component for the widget body
   */
  protected bodyContent(): JSX.Element | undefined {
    return undefined;
  }

  /**
   * Override this method to customize the widget footer.
   * @returns react node for the widget footer
   */
  protected footerContent(): JSX.Element | undefined {
    return undefined;
  }
}
```

| Methods               | Function                                                                                                                                                                             | Recommend to override |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| `constructor()`       | Assigns the initial `this.state` and call the constructor of the super class `React.Component`.                                                                                     | NO                    |
| `componentDidMount()` | This method will be invoked after a component is mounted and assigns a value to the `data` property of the state by calling the `getData()` method.                                 | NO                    |
| `render()`            | This method will be called each time an update happens, the dashboard default layout is defined in this method.                                                                      | NO                    |
| `getData()`           | This method can be used to get the data needed by the widget, and the value returned by this method will be set to `this.state.data`.                                                | YES                   |
| `headerContent()`     | This method is used to define what the widget header will look like. You can choose to override this method to define the header of your widget, if not, the widget will not have a header. | YES                   |
| `bodyContent()`       | This method is used to define what the widget body will look like. You can choose to override this method to define the body of your widget, if not, the widget will not have a body.     | YES                   |
| `footerContent()`     | This method is used to define what the widget footer will look like. You can choose to override this method to define the footer of your widget, if not, the widget will not have a footer. | YES                   |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Dashboard abstraction
In order to easily define and adjust the layout of the dashboard, the TeamsFx provides a `Dashboard` class 
for developers. You can inherit this `Dashboard` class to quickly implement your own dashboard.

Below is the definition of the Dashboard class.

```ts
interface IDashboardState {
  isMobile?: boolean;
  observer?: ResizeObserver;
}

/**
 * The dashboard class which is the base class for all dashboard components.
 */
export class Dashboard extends Component<any, IDashboardState> {
  private ref: React.RefObject<HTMLDivElement>;

  /**
   * Constructor for the dashboard class. Initializes the dashboard state.
   * @param props The properties for the dashboard.
   */
  constructor(props: any) {
    super(props);
    this.state = {
      isMobile: undefined,
      observer: undefined,
    };
    this.ref = React.createRef<HTMLDivElement>();
  }

  /**
   * This method is invoked immediately after a component is mounted. It's a good place to fetch data from server.
   */
  componentDidMount(): void {
    // Observe the dashboard div for resize events
    const observer = new ResizeObserver((entries) => {
      for (let entry of entries) {
        if (entry.target === this.ref.current) {
          const { width } = entry.contentRect;
          this.setState({ isMobile: width < 600 });
        }
      }
    });
    observer.observe(this.ref.current!);
  }

  /**
   * This method is invoked immediately when a component will be unmounted. It's a good place to clean up the resources.
   */
  componentWillUnmount(): void {
    // Unobserve the dashboard div for resize events
    if (this.state.observer && this.ref.current) {
      this.state.observer.unobserve(this.ref.current);
    }
  }

  /**
   * Define the dashboard default layout, you can edit the code here to customize your dashboard layout.
   */
  render() {
    return (
      <>
        <div
          ref={this.ref}
          style={dashboardStyles(
            this.state.isMobile,
            this.rowHeights(),
            this.columnWidths()
          )}
        >
          {this.dashboardLayout()}
        </div>
      </>
    );
  }

  /**
   * Implement this method to define the row heights of the dashboard.
   * For example, if you want to have 3 rows, and the height of the first row is 100px, the height of the second row is 200px, and the height of the third row is 300px, you can return "100px 200px 300px".
   * @returns The row heights of the dashboard.
   */
  protected rowHeights(): string | undefined {
    return undefined;
  }

  /**
   * Implement this method to define the column widths of the dashboard.
   * For example, if you want to have 3 columns, and each column occupies 1/3 of the full width, you can return "1fr 1fr 1fr".
   * @returns The column widths of the dashboard.
   */
  protected columnWidths(): string | undefined {
    return undefined;
  }

  /**
   * Implement this method to define the dashboard layout.
   */
  protected dashboardLayout(): JSX.Element | undefined {
    return undefined;
  }
}
```

In the Dashboard class, the TeamsFx provided some basic layouts and customizable methods. The Dashboard is actually still a react component, and the TeamsFx provide some basic implementations of functions based on the lifecycle of react components, such as implementing a basic render logic based on the Grid layout, adding an observer to automatically adapt to mobile devices.


| Methods                  | Function                                                                                                       | Recommend to override |
|--------------------------|----------------------------------------------------------------------------------------------------------------|-----------------------|
| `constructor()`          | Initializes the dashboard state and variables                                                                  | NO                    |
| `componentDidMount()`    | This method will be invoked after a component is mounted                                                       | NO                    |
| `componentWillUnmount()` | This method will be invoked when a component will be unmounted                                                 | NO                    |
| `render()`               | This method will be called each time an update happens, we defined the dashboard default layout in this method | NO                    |
| `rowHeights()`           | Customize the height of each row of the dashboard                                                              | YES                   |
| `columnWidths()`         | Customize how many columns the dashboard has at most and the width of each column                              | YES                   |
| `dashboardLayout()`      | Define the widgets layout in dashboard                                                                         | YES                   |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new widget

You can use the following steps to add a new widget to the dashboard:

1. [Step 1: Define a data model](#step-1-define-a-data-model)
2. [Step 2: Create a data retrive service](#step-2-create-a-data-retrive-service)
3. [Step 3: Create a widget file](#step-3-create-a-widget-file)
4. [Step 4: Add the widget to the dashboard](#step-4-add-the-widget-to-the-dashboard)

### Step 1: Define a data model

Define a data model based on the business scenario and put it in `src/models` folder. The widget model defined according to the data you want to display in the widget. Here's a sample data model:

```typescript
export interface SampleModel {
  content: string;
}
```

### Step 2: Create a data retrive service

Simply, you can create a service that returns dummy data. We recommend that you put data files in the `src/data` folder and put data retrive services in the `src/services` folder.

Here's a sample json file that contains dummy data:

```json
{
  "content": "Hello world!"
}
```

Here's a dummy data retrive service:

```typescript
import { SampleModel } from "../models/sampleModel";
import SampleData from "../data/SampleData.json";

export const getSampleData = (): SampleModel => SampleData;
```

> Note: You can also implement a service to retrieve data from the backend service or from the Microsoft Graph API.

### Step 3: Create a widget file

Create a widget file in `src/views/widgets` folder. Extend the `src/views/lib/Widget` class. The following table shows the methods that you can override to customize your widget.

| Methods           | Function                                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `getData()`       | This method is used to get the data for the widget. You can implement it to get data from the backend service or from the Microsoft Graph API |
| `headerContent()` | Customize the content of the widget header                                                                                                    |
| `bodyContent()`   | Customize the content of the widget body                                                                                                      |
| `footerContent()` | Customize the content of the widget footer                                                                                                    |

> All methods are optional. If you do not override any method, the default widget layout will be used.

Here's a sample widget implementation:

```tsx
import { Button, Text } from "@fluentui/react-components";
import { Widget } from "../lib/Widget";
import { SampleModel } from "../../models/sampleModel";
import { getSampleData } from "../../services/sampleService";

export class SampleWidget extends Widget<SampleModel> {
  async getData(): Promise<SampleModel> {
    return getSampleData();
  }

  headerContent(): JSX.Element | undefined {
    return <Text>Sample Widget</Text>;
  }

  bodyContent(): JSX.Element | undefined {
    return <div>{this.state.data?.content}</div>;
  }

  footerContent(): JSX.Element | undefined {
    return (
      <Button
        appearance="primary"
        size="medium"
        style={{ width: "fit-content" }}
        onClick={() => {}}
      >
        View Details
      </Button>
    );
  }
}
```

### Step 4: Add the widget to the dashboard

1. Go to `src/views/dashboards/SampleDashboard.tsx`, if you want to create a new dashboard, please refer to [How to add a new dashboard](#how-to-add-a-new-dashboard).
2. Update your `dashboardLayout()` method to add the widget to the dashboard:

```tsx
protected dashboardLayout(): void | JSX.Element {
  return (
    <>
      <ListWidget />
      <ChartWidget />
      <SampleWidget />
    </>
  );
}
```

> Note: If you want put your widget in a column, you can use the [`oneColumn()`](src/views/lib/Dashboard.styles.ts#L32) method to define the column layout. Here is an example:

```tsx
protected dashboardLayout(): void | JSX.Element {
  return (
    <>
      <ListWidget />
      <div style={oneColumn()}>
        <ChartWidget />
        <SampleWidget />
      </div>
    </>
  );
}
```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new dashboard

You can use the following steps to add a new dashboard layout:

1. [Step 1: Create a dashboard class](#step-1-create-a-dashboard-class)
2. [Step 2: Override methods to customize dashboard layout](#step-2-override-methods-to-customize-dashboard-layout)
3. [Step 3: Add a route for the new dashboard](#step-3-add-a-route-for-the-new-dashboard)
4. [Step 4: Modify manifest to add a new dashboard tab](#step-4-modify-manifest-to-add-a-new-dashboard-tab)

### Step 1: Create a dashboard class

Create a file with the extension `.tsx` for your dashboard in the `src/views/dashboards` directory. For example, `YourDashboard.tsx`. Then, create a class that extends the `src/views/lib/Dashboard` class.

```tsx
export default class YourDashboard extends Dashboard {}
```

### Step 2: Override methods to customize dashboard layout

Dashboard class provides some methods that you can override to customize the dashboard layout. The following table lists the methods that you can override.

| Methods             | Function                                                                          |
| ------------------- | --------------------------------------------------------------------------------- |
| `rowHeights()`      | Customize the height of each row of the dashboard                                 |
| `columnWidths()`    | Customize how many columns the dashboard has at most and the width of each column |
| `dashboardLayout()` | Define widgets layout                                                             |

Here is an example to customize the dashboard layout.

```tsx
export default class YourDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "500px";
  }

  protected columnWidths(): string | undefined {
    return "4fr 6fr";
  }

  protected dashboardLayout(): void | JSX.Element {
    return (
      <>
        <SampleWidget />
        <div style={oneColumn("6fr 4fr")}>
          <SampleWidget />
          <SampleWidget />
        </div>
      </>
    );
  }
}
```

> Note: All methods are optional. If you do not override any method, the default dashboard layout will be used.

### Step 3: Add a route for the new dashboard

Open the `src/App.tsx` file, and add a route for the new dashboard. Here is an example:

```tsx
import YourDashboard from "./views/dashboards/YourDashboard";

export default function App() {
  ...
  <Route exact path="/yourdashboard" component={YourDashboard} />
  ...
}
```

### Step 4: Modify manifest to add a new dashboard tab

Open the `appPackage/manifest.json` file and add a new dashboard tab under the `staticTabs`. Here is an example:

```json
{
  "entityId": "index1",
  "name": "Your Dashboard",  
  "contentUrl": "${{TAB_ENDPOINT}}/index.html#/tab",
  "websiteUrl": "${{TAB_ENDPOINT}}/index.html#/tab",
  "scopes": ["personal"]
}
```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Customize the Widget
Since the widget class provides some methods that can be overridden to customize the widget, you can customize the widget by overriding these methods.

- Override `headerContent()`、`bodyContent()`、`footerContent()` to customize the widget.

```ts
export class NewsWidget extends Widget<void> {

    headerContent(): JSX.Element | undefined {
        return (
            <div style={headerContentStyle()}>
                <News28Regular />
                <Text style={headerTextStyle()}>Your News</Text>
                <Button icon={<MoreHorizontal32Regular />} appearance="transparent" />
            </div>
        );
    }

    bodyContent(): JSX.Element | undefined {
        return (
            <div style={contentLayoutStyle()}>
                <Image src="image.svg" style={imageStyle()} />
                <Text style={titleStyle()}>Lorem Ipsum Dolor</Text>
                <Text style={descStyle()}>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim, elementum sed</Text>
            </div>
        );
    }

    footerContent(): JSX.Element | undefined {
        return (
            <Button
                appearance="transparent"
                icon={<ArrowRight16Filled />}
                iconPosition="after"
                size="small"
                style={footerButtonStyle()}
                onClick={() => { }} // navigate to detailed page
            >
                View details
            </Button>
        );
    }
}
```

![image](https://user-images.githubusercontent.com/107838226/209327297-bbcc62fd-9ec1-4a4e-88af-f408085634aa.png)


- Just override `bodyContent()`、`footerContent()` to customize the widget.

```ts
export class NewsWidget extends Widget<void> {

    bodyContent(): JSX.Element | undefined {
        return (
            <div style={contentLayoutStyle()}>
                <Image src="image.svg" style={imageStyle()} />
                <Text style={titleStyle()}>Lorem Ipsum Dolor</Text>
                <Text style={descStyle()}>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim, elementum sed</Text>
            </div>
        );
    }

    footerContent(): JSX.Element | undefined {
        return (
            <Button
                appearance="transparent"
                icon={<ArrowRight16Filled />}
                iconPosition="after"
                size="small"
                style={footerButtonStyle()}
                onClick={() => { }} // navigate to detailed page
            >
                View details
            </Button>
        );
    }
}
```

![image](https://user-images.githubusercontent.com/107838226/209326796-69fa92a6-0582-4ddb-bb1c-8383ac805433.png)


- Just override `bodyContent()` to customize the widget.

```ts
export class NewsWidget extends Widget<void> {

    bodyContent(): JSX.Element | undefined {
        return (
            <div style={contentLayoutStyle()}>
                <Image src="image.svg" style={imageStyle()} />
                <Text style={titleStyle()}>Lorem Ipsum Dolor</Text>
                <Text style={descStyle()}>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim, elementum sed</Text>
            </div>
        );
    }

}
```

![image](https://user-images.githubusercontent.com/107838226/209326667-e88e000f-04c5-48e6-a976-b6e40237a9ad.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>



## How to include a data loader

If you want to include a data loader to your widget before the widget is loaded, you can add a property to the state of the widget to indicate that the data loader is loading. This property can be used to show a loading indicator to the user. 

The following steps show how to add a property to the state of the ListWidget and how to use it to show a loading spinner while the data is loading.

### Step 1: Define a state type
Define a state type including a property named `loading` that indicates whether the data is loading.

```ts
interface ListWidgetState {
  data: ListModel[];
  loading?: boolean;
}
```

### Step 2: Add a data loader

Modify the `bodyContent` method to show a loading spinner if data is loading.

```ts
bodyContent(): JSX.Element | undefined {
  return (
    <>
      {this.state.loading !== false ? (
        <div style={{ display: "grid", justifyContent: "center", height: "100%" }}>
          <Spinner label="Loading..." labelPosition="below" />
        </div>
      ) : (
        <div style={bodyContentStyle()}>
          ...
        </div>
      )}
    </>
  );
}
```

### Step 3: Hide the footer button if data is loading

```ts
footerContent(): JSX.Element | undefined {
  if (this.state.loading === false) {
    return (
      <Button
        ...
      </Button>
    );
  }
}
```

### Step 4: Update the state reference

Update the state reference in the widget file to use the new state type and update the state in the `getData` method to set the `loading` property to `false` after the data is loaded.

```ts
...
export class ListWidget extends Widget<ListWidgetState> {
  
  async getData(): Promise<ListWidgetState> {
    return { data: await getListData(), loading: false };
  }
...
```

Now, the loading spinner is shown while the data is loading. When the data is loaded, the loading spinner is hidden, and the list data and footer button are shown.

![5xcq3-utv28](https://user-images.githubusercontent.com/107838226/212793039-90aa6f7e-28fd-4650-bb64-f9c3859d260b.gif)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to handle empty state

You can display a specific content in your widget when the data is empty. To do so, you need to modify the `bodyContent` method in your widget file to adopt different states of the data. The following example shows how to display an empty image when the data of ListWidget is empty:

```ts
bodyContent(): JSX.Element | undefined {
    let hasData = this.state.data && this.state.data.length > 0;
    return (
      <div style={bodyContentStyle()}>
        {hasData ? (
          <>
            {this.state.data?.map((t: ListModel) => {
              ...
            })}
          </>
        ) : (
          <div
            style={{
              display: "grid",
              gap: "1rem",
              justifyContent: "center",
              alignContent: "center",
            }}
          >
            <Image src="empty-default.svg" height="150px" />
            <Text align="center">No data</Text>
          </div>
        )}
      </div>
    );
  }
```

And you can use a similar approach to remove the footer content of your widget when the data is empty. 

```ts
footerContent(): JSX.Element | undefined {
    let hasData = this.state.data && this.state.data.length > 0;
    if (hasData) {
      return (
        <Button
          ...
        </Button>
      );
    }
  }
```

Your list widget will look like this when the data is empty:
![image](https://user-images.githubusercontent.com/107838226/212546123-33b8943e-094c-4de5-9f23-61868ee4e51b.png)


<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to refresh data as scheduled

The following example shows how to display real-time data in a widget. The widget displays the current time and updates every second.

```tsx
import { Widget } from "../lib/Widget";

interface IRefreshWidgetState {
  data: string;
}

export class RefreshWidget extends Widget<IRefreshWidgetState> {
  bodyContent(): JSX.Element | undefined {
    return <>{this.state.data}</>;
  }

  async componentDidMount() {
    setInterval(() => {
      this.setState({ data: new Date().toLocaleTimeString() });
    }, 1000);
  }
}
```

You can modify `setInterval` method to call your own function to refresh data, like this: `setInterval(() => yourGetDataFunction(), 1000)`.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to use Microsoft Graph Toolkit as widget content

Microsoft Graph Toolkit is a set of reusables, framework-agnostic web components and helpers for accessing and working with Microsoft Graph. You can use the Microsoft Graph Toolkit with any web framework or without a framework at all. 

You can follow the steps below to use Microsoft Graph Toolkit as your widget content.

### Step 1: Add SSO feature to your Teams app

Microsoft Teams provides single sign-on (SSO) function for an app to obtain signed in Teams user token to access Microsoft Graph. For how to add SSO feature to your Teams app, please refer to [this document](https://aka.ms/teamsfx-add-sso-new).

### Step 2: Install required npm packages

Run the following command in the root directory of your project to install the required npm packages:

```bash
npm install @microsoft/mgt-react @microsoft/mgt-element @microsoft/mgt-teamsfx-provider
```

### Step 3: Add a new Graph Toolkit widget

Create a new widget file in your project `src/views/widgets` folder. For example, `GraphyWidget.tsx`. In this widget, we will guide users to consent our app to access Microsoft Graph and then show the user's todo list by using Microsoft Graph Toolkit. The following code is an example of using `Todo` component from Microsoft Graph Toolkit in widget.

```tsx
import { Providers, ProviderState, Todo } from "@microsoft/mgt-react";
import { TeamsFxProvider } from "@microsoft/mgt-teamsfx-provider";

import { loginAction } from "../../internal/login";
import { TeamsUserCredentialContext } from "../../internal/singletonContext";
import { Widget } from "../lib/Widget";

interface IGraphWidgetState {
  needLogin: boolean;
}

export class GraphWidget extends Widget<IGraphWidgetState> {
  protected bodyContent(): JSX.Element | undefined {
    return <div>{this.state.needLogin === false && <Todo />}</div>;
  }

  async componentDidMount() {
    super.componentDidMount();

    // Initialize TeamsFx provider
    const provider = new TeamsFxProvider(TeamsUserCredentialContext.getInstance().getCredential(), [
      "Tasks.ReadWrite",
    ]);
    Providers.globalProvider = provider;

    // Check if user is signed in
    if (await this.checkIsConsentNeeded()) {
      await loginAction(["Tasks.ReadWrite"]);
    }

    // Update signed in state
    Providers.globalProvider.setState(ProviderState.SignedIn);
    this.setState({ needLogin: false });
  }

  /**
   * Check if user needs to consent
   * @returns true if user needs to consent
   */
  async checkIsConsentNeeded() {
    let needConsent = false;
    try {
      await TeamsUserCredentialContext.getInstance().getCredential().getToken(["Tasks.ReadWrite"]);
    } catch (error) {
      needConsent = true;
    }
    return needConsent;
  }
}
```

You also can use other Microsoft Graph Toolkit components in your widget. For more information about Microsoft Graph Toolkit components, please refer to [this document](https://learn.microsoft.com/en-us/graph/toolkit/overview).

### Step 4: Add the widget to dashboard layout

Include the new widget in your dashboard file.

```tsx
...
export default class YourDashboard extends Dashboard {
  ...
  protected dashboardLayout(): undefined | JSX.Element {
    return (
      <>
        <GraphWiget />
      </>
    );
  }
  ...
}
```

Now, launching or refreshing your Teams app, you will see the new widget using Microsoft Graph Toolkit after you finish login to consent our app to access Microsoft Graph.

![image](https://user-images.githubusercontent.com/107838226/213122528-f0823063-b5b6-4cbf-9f2c-0198e6d72958.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to embed Power BI to Dashboard

For how to embed Power BI item to the Dashboard, you can refer to [this document](https://learn.microsoft.com/en-us/javascript/api/overview/powerbi/powerbi-client-react).

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Customize the Dashboard Layout
The TeamsFx provided some convenient methods for defining and modifying the layout of the dashboard.

- Three widgets in a row with a height of 350px, occupying 20%, 60% and 20% of the width respectively.

```ts
export default class SampleDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "350px";
  }

  protected columnWidths(): string | undefined {
    return "2fr 6fr 2fr";
  }

  protected dashboardLayout(): undefined | JSX.Element {
    return (
      <>
        <ListWidget />
        <ChartWidget />
        <NewsWidget />
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209525799-9ee5b7ab-6263-4a1f-89e6-b33eee6d31dc.png)

- There are two widgets in a row with widths of 600px and 1100px, the height of the first line is the maximum height of its content, and the height of the second line is 400px.
```ts
export default class SampleDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "max-content 400px";
  }

  protected columnWidths(): string | undefined {
    return "600px 1100px";
  }

  protected dashboardLayout(): undefined | JSX.Element {
    return (
      <>
        <ListWidget />
        <ChartWidget />
        <NewsWidget />
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209532741-0ad90649-f904-4dae-af25-940e7f874bf2.png)

- Arrange two widgets in a column.
```ts
import { oneColumn } from '../lib/Dashboard.styles';
export default class SampleDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "max-content";
  }

  protected columnWidths(): string | undefined {
    return "4fr 6fr";
  }

  protected dashboardLayout(): undefined | JSX.Element {
    return (
      <>
        <NewsWidget />
        <div style={oneColumn()}>
          <ListWidget />
          <ChartWidget />
          
        </div>
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209540272-094c871c-cdf0-45a4-b131-b1548e182e6f.png)


- Customize the height of widgets in a row. The following code can achieve a height of 400px for the `ListWidget` and a height of 350px for the `ChartWidget`.

```ts
import { oneColumn } from '../lib/Dashboard.styles';
export default class SampleDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "max-content";
  }

  protected columnWidths(): string | undefined {
    return "4fr 6fr";
  }

  protected dashboardLayout(): undefined | JSX.Element {
    return (
      <>
        <NewsWidget />
        <div style={oneColumn("400px 350px")}>
          <ListWidget />
          <ChartWidget />
        </div>
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209540029-1a888753-a03c-44e2-bd93-897fef077489.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new Graph API call

### Add SSO First

Before you add your logic of calling a Graph API, you should enable your dashboard project to use SSO. For how to add an SSO to your project you can refer to [this document](https://aka.ms/teamsfx-add-sso-new);

Now you have already added SSO files to your project, and you can call Graph APIs. There are two types of Graph APIs, one will be called from the front-end(most of APIs, use delegated permissions), the other will be called from the back-end(sendActivityNotification, e.g., use application permissions). You can refer to [this tutorial](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-beta) to check permission types of the Graph APIs you want to call.

### Call graph api from the front-end(use delegated permissions)

If you want to call a Graph API from the front-end tab, you can refer to the following steps.

1. [Step 1: Consent delegated permissions first](#step-1-consent-delegated-permissions-first)
2. [Step 2: Create a graph client by adding the scope related to the Graph API you want to call](#step-2-create-a-graph-client-by-adding-the-scope-related-to-the-graph-api-you-want-to-call)
3. [Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end](#step-3-call-the-graph-api-and-parse-the-response-into-a-certain-model-which-will-be-used-by-front-end)

#### Step 1: Consent delegated permissions first

You can call [`addNewPermissionScope(scopes: string[])`](https://github.com/OfficeDev/TeamsFx/blob/dev/templates/tab/ts/dashboard/src/internal/addNewScopes.ts) from `tab/ts/dashboard/src/internal/addNewScopes.ts` to consent the scopes of permissions you want to add. And the consented status will be preserved in a global context [`TeamsUserCredentialContext`](https://github.com/OfficeDev/TeamsFx/blob/dev/templates/tab/ts/dashboard/src/internal/singletonContext.ts).

You can refer to [the Graph API V1.0](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-1.0) to get the `scope name of the permission` related to the Graph API you want to call.

#### Step 2: Create a graph client by adding the scope related to the Graph API you want to call

You can refer to the following code snippet:

```ts
let credential: TeamsUserCredential;  
credential = TeamsUserCredentialContext.getInstance().getCredential();
const graphClient: Client = createMicrosoftGraphClientWithCredential(credential, scope);
```

#### Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end

You can refer to the following code snippet:

```ts
try {
  const graphApiResult = await graphClient.api("<GRAPH_API_PATH>").get();
  // Parse the graphApiResult into a Model you defined, used by the front-end.
} catch (e) {}
```

### Call graph api from the back-end(use application permissions)

If you want to call a Graph API from the back-end, you can refer to the following steps.

1. [Step 1: Consent application permissions first](#step-1-consent-application-permissions-first)
2. [Step 2: Add an Azure Function](#step-2-add-an-azure-function)
3. [Step 3: Add your logic in Azure Function](#step-4-add-your-logic-in-azure-function)
4. [Step 4: Call the Azure Function from the front-end](#step-5-call-the-azure-function-from-the-front-end)

#### Step 1: Consent application permissions first

Go to [Azure portal](https://portal.azure.com/) > Click `Azure Active Directory` > Click `App registrations` in the side bar > Click your Dashboard app > Click `API permissions` in the side bar > Click `+Add a permission` > Choose `Microsoft Graph` > Choose `Application permissions` > Find the permissions you need > Click `Add permissions` button in the bottom > Click `✔Grant admin consent for XXX` and then click `Yes` button to finish the admin consent

#### Step 2: Add an Azure Function

For how to add an Azure Function to your project you can refer to [this doc](https://aka.ms/teamsfx-add-azure-function).

#### Step 3: Add your logic in Azure Function

In the `index.jsx`/`index.ts` under the folder named in step 2, you can add your logic which contains back-end graph api calling with application permissions. You can refer to the following code snippet.

```ts
/**
 * This function handles requests from teamsfx client.
 * The HTTP request should contain an SSO token queried from Teams in the header.
 * Before trigger this function, teamsfx binding would process the SSO token and generate teamsfx configuration.
 *
 * You should initializes the teamsfx SDK with the configuration and calls these APIs.
 *
 * The response contains multiple message blocks constructed into a JSON object, including:
 * - An echo of the request body.
 * - The display name encoded in the SSO token.
 * - Current user's Microsoft 365 profile if the user has consented.
 *
 * @param {Context} context - The Azure Functions context object.
 * @param {HttpRequest} req - The HTTP request.
 * @param {teamsfxContext} TeamsfxContext - The context generated by teamsfx binding.
 */
export default async function run(
  context: Context,
  req: HttpRequest,
  teamsfxContext: TeamsfxContext
): Promise<Response> {
  context.log("HTTP trigger function processed a request.");

  // Initialize response.
  const res: Response = {
    status: 200,
    body: {},
  };

  // Your logic here.

  return res;
}
```
#### Step 4: Call the Azure Function from the front-end

Call the Azure Function by function name. You can refer to the following code snippet to call the Azure Function.

```ts
const functionName = process.env.REACT_APP_FUNC_NAME || "myFunc";
export let taskName: string;

export async function callFunction(params?: string) {
  taskName = params || "";
  const credential = TeamsUserCredentialContext.getInstance().getCredential();
  if (!credential) {
    throw new Error("TeamsFx SDK is not initialized.");
  }
  try {
    const apiBaseUrl = process.env.REACT_APP_FUNC_ENDPOINT + "/api/";    
    const apiClient = createApiClient(
      apiBaseUrl,
      new BearerTokenAuthProvider(async () => (await credential.getToken(""))!.token)
    );
    const response = await apiClient.get(functionName);
    return response.data;
  } catch (err: unknown) {
    ...
  }
}
```

Refer to [this sample](https://github.com/OfficeDev/TeamsFx-Samples/blob/dev/hello-world-tab-with-backend/tabs/src/components/sample/AzureFunctions.tsx) for some helps. And you can read [this doc](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference?tabs=blob) for more details.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>
