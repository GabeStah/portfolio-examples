**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](https://github.com/GabeStah/portfolio-examples/#warning--copyright-notice) for more info.**

- [App Architecture](#app-architecture)
- [Widget Theme](#widget-theme)
  - [Adding a New Theme](#adding-a-new-theme)
- [Editing the Widget](#editing-the-widget)
- [How a Plugin Is Implemented](#how-a-plugin-is-implemented)
- [Widget Testing](#widget-testing)
  - [Local Widget + Local Test Site](#local-widget--local-test-site)
  - [Local Widget + Local API Server](#local-widget--local-api-server)
- [Using Inline SVGs](#using-inline-svgs)
- [Known Issues](#known-issues)

## App Architecture

- Typescript: Improved syntax and type checking.
- Preact: A smaller footprint version of React that provides 90% of the functionality.
- Redux: State management for use with Preact components.
- Redux-Saga: Handles background tasks / threading.
- SASS: Advanced styling.
- Webpack + Babel: General tooling to generate final build UMD output.

## Widget Theme

Widget theming provides an easy way to customize the look and feel of the widget with just a few palette color changes, or a total overhaul of element behavior and styling. Existing themes can be found in the [src/theme](src/theme) directory. The [base](src/theme/base) theme is the default Widget theme, based on design mockups.

### Adding a New Theme

1. Create a new directory in [src/theme](src/theme).
2. Copy an existing theme [index.ts](src/theme/base/index.ts) and [palette.ts](src/theme/base/pelette.ts) into the new directory.
3. Modify the new `palette.ts` `colors` constant as needed:

```ts
const colors = {
  background: '#fff',
  backgroundEnabled: '#FAFAFA',
  border: fade('#999999', 0.5),
  primary: '#297FCA',
  primaryDark: '#184570',
  primaryLight: '#e8f6ff',
  switchMain: '#fff',
  textSecondary: '#fff',
};
```

4. Add a new entry to the [ThemeTypes](src/theme/types.ts) enum to identify your theme.
5. Lastly, update the [index.tsx](src/index.tsx) theme section by importing your theme index file and adding it to the theme switch statement matching your new `ThemeTypes` entry.

Now invoke `ActionCreators.setTheme({ theme: ThemeTypes.NewThemeType })` anywhere you wish to dispatch an action to update the theme. This will disperse the set theme to all Widget components and rerender.

## Editing the Widget

The **WCASG Widget** is constructed out of individual plugins, each of which can be toggled independently. Plugins are designed to be as modular as possible, to make it easier to add more functionality/plugins in the future.

The [`Plugin`](src/types.ts#L90) interface defines the structure of a specific plugin:

- The `config` property contains a series of [`PluginProperty`](src/types.ts) props that define stateful, user-changeable plugin settings.
- The `tasks` property contains one or more [`IPluginActions`](src/state/redux/state.ts) which define executable functions used by the Plugin to manipulate the DOM.

## How a Plugin Is Implemented

The [`PluginActionStyle`](src/classes/plugin/action/style/index.ts) class is one action type example that allows for DOM element style manipulation. The [font-size](src/plugins/font-size/plugin.ts) Plugin makes simple use of the `PluginActionStyle` to manipulate the `font-size` style property:

```ts
const actionStyle = new PluginActionStyle({
  name: 'adjust-font-size-action',
  style: {
    name: 'font-size',
    manipulationType: ValueManipulationType.PercentageScaling,
  },
  query: new TextNodeType().types.join(', '),
});
```

The `query` defines what nodes are manipulated by the action. The `style.manipulationType` option defines how the style property of this action is altered, whether percentage scaling, absolute scaling, toggled, or direct input manipulation. In the case of the `font-size` Plugin we want to scale the `font-size` property of affected nodes, so we use percentage scaling.

Classes extending `PluginAction` initialize by performing some setup actions. For `PluginActionStyle` we don't want to lose a reference to the original value of the property that is being altered, so we start by adding a new Widget-specific data attribute to affected nodes that holds their base value:

```ts
export class PluginActionStyle
  extends PluginAction
  implements IPluginActionStyle {
  public style: IPluginActionStyleOptions;
  public domValueType: DOMValueType = DOMValueType.Style;

  constructor(params: IPluginActionStyleParams) {
    super(params);
    this.style = params.style;

    // Initialize by generating original data attributes for property
    this.addDataAttributeForStyles();
  }

  protected addDataAttributeForStyles(): void {
    Data.createOriginalDataAttribute({
      node: this.nodeList,
      name: this.style.name,
      type: this.domValueType,
    });
  }
}
```

The `IPluginAction` interface requires all implementers define an `enable()` and `disable()` method, which are invoked throughout the app to trigger actions when the associated Plugin is enabled/disabled. The `PluginActionStyle.enable()` method updates or removes style values on relevant nodes, depending on configuration.

As an example, the `font-size` Plugin defines an `onEnableOrChange()` generator function that can update the Plugin's state and trigger the `enable()` method on the `PluginActionStyle` instance defined earlier:

```ts
function* onEnableOrChange() {
  const state = yield select();
  const selectors = new Selectors(state);
  // Get latest state version.
  const plugin = selectors.getPlugin(pluginObject.id);
  if (!plugin.enabled) {
    return;
  }
  actionStyle.enable({ factor: plugin.scaling?.factor });
}

export const pluginObject: Plugin = {
  id: Ids.FontSize,
  title: 'Font Size',
  enabled: false,
  scaling: {
    baseFactor: 0,
    factor: 0,
    increment: 0.1,
    type: ValueManipulationType.PercentageScaling,
  },
  tasks: [
    {
      on: PluginActionTypes.enable,
      func: [onEnableOrChange],
    },
    {
      on: PluginActionTypes.increment,
      func: [onEnableOrChange],
    },
    {
      on: PluginActionTypes.decrement,
      func: [onEnableOrChange],
    },
    {
      on: PluginActionTypes.disable,
      func: [() => actionStyle.disable()],
    },
  ],
};
```

The `tasks` array defines a few common `PluginActionTypes` that are shared across many plugins via the [Reducer](src/state/redux/reducers/index.ts) methods. These actions are triggered within in the root saga's generator collection here:

```ts
export function* rootSagas() {
  yield all([
    takeEvery(
      // Pattern to track all actions from reducer
      (action: Action) => isActionFrom(action, BaseReducer),
      watchPluginTasks
    ),
  ]);
}
```

`isActionFrom` determines if the action (e.g. `PluginActionTypes.decrement`) is a method within the `BaseReducer` class. If so, the `watchPluginTasks` generator is called:

```ts
/**
 * Executes plugin tasks on state change.
 * @param action
 * @returns {Generator<never, void, unknown>}
 */
export function* watchPluginTasks(action: any) {
  if (!action || !action.payload || !action.payload.id) {
    return;
  }
  const plugin = PluginManager.getInstance().find(action.payload.id);
  if (plugin && plugin.tasks) {
    for (const task of plugin.tasks) {
      if (task.on === getActionTypeFromImmer(action)) {
        if (task.func) {
          for (const func of task.func) {
            yield func();
          }
        }
      }
    }
  }
}
```

This generator function determines which specific Plugin and which of that Plugin's `task` functions to execute. Thus, a front-end component can invoke a given action and that will trigger the appropriate state change. For example, consider this part of the `Scalable` component that provides buttons for increasing/decreasing a property:

```tsx
export const Scalable = ({
  actions,
  autoToggle = true,
  plugin,
  scaling,
  showFactor = false,
  state,
  theme,
}: PluginScalableComponentParams) => {
  return (
    <>
      <Button
        aria-label={`Increment ${plugin.title}`}
        aria-roledescription={'button'}
        className={
          greaterThan(scaling.factor, scaling.baseFactor) ? styles.active : ''
        }
        classes={{
          startIcon: greaterThan(scaling.factor, scaling.baseFactor)
            ? styles.startIcon
            : '',
        }}
        color={'primary'}
        onClick={() => actions.increment(plugin.id)}
        role={'button'}
        startIcon={<ChevronThinUpIcon />}
        tabIndex={0}
        variant={'outlined'}
      />
      <Button
        aria-label={`Decrement ${plugin.title}`}
        aria-roledescription={'button'}
        onClick={() => actions.decrement(plugin.id)}
        className={`${styles.rotated} ${
          lessThan(scaling.factor, scaling.baseFactor) ? styles.active : ''
        }`}
        classes={{
          startIcon: lessThan(scaling.factor, scaling.baseFactor)
            ? styles.startIcon
            : '',
        }}
        color={'primary'}
        role={'button'}
        startIcon={<ChevronThinUpIcon />}
        tabIndex={0}
        variant={'outlined'}
      />
    </>
  );
};
```

The `onClick()` method invokes the `actions.increment()` / `actions.decrement()` methods, which are picked up the root saga go from there to making actual state changes. In the case of the `font-size` Plugin example, this causes the `font-size` properties on relevant DOM nodes to update accordingly.

There are a handful of `PluginAction` class types that Plugins utilize:

- [`PluginActionClass`](src/classes/plugin/action/class/index.ts) alters a node's assigned CSS classes list, making it ideal for `PluginElements` that want to affect styling of the entire page.
- [`PluginActionStyle`](src/classes/plugin/action/style/index.ts) is used for fine-grained manipulation of node styling. It is best used when modifying styling incrementally (such as `font-size` via the `PluginElementScalable` element).
- [`PluginActionProperty`](src/classes/plugin/action/property/index.ts) alters the properties of the node (i.e. `node[propertyName] = newValue`).
- [`PluginActionFunction`](src/classes/plugin/action/function/index.ts) is used for arbitrary function execution and should be constructed by passing one or more functions it will execute to the `.func` property.

## Widget Testing

### Local Widget + Local Test Site

1. Run `yarn run dev`.
2. Open [http://localhost:5000/basic/](http://localhost:5000/basic/) to test local page.

### Local Widget + Local API Server

1. Run SaaS dashboard app locally.
2. Run `yarn run dev`.
3. Open [http://localhost:5000/api/](http://localhost:5000/api/) to the `tests/sites/api` page, which includes script at localhost API address (default: `http://localhost:84/api/widget`).

OR

1. Visit a web page to be tested.
2. Copy raw contents from [build/index.js](http://gitlab.solarixdigital.com/solarix/wcasg-ada-app/raw/master/build/index.js).
3. Paste into Chrome Dev Tools.

- Alternatively, create Bookmarklet that loads raw `build/index.js` content.
- Alternatively, use local Git clone'd copy by adding `build/index.js` to `Chrome Dev Tools > Filesystem > Workspace`, then `Ctrl + A` to select and `Ctrl + Shift + E` to execute on current page.

## Using Inline SVGs

1. Add SVG file to `src/assets/svg`.
2. Run `npm run svg:optimize`, which uses the [svgo](https://github.com/svg/svgo) tool to minify.
3. Reference the minified svg copy found in `srv/assets/svg-minified`:

```css
.largeCursor,
.largeCursor * {
  cursor: url(src/assets/svg-minified/cursor-arrow.svg), default !important;
}
```

## Known Issues

- Building the first time after importing / referencing a new `.scss` style or file will fail, reporting the property doesn't exist in `CssExports`. This is due to the build order for TypeScript + SCSS imports. Just force a second build to resolve the issue.
