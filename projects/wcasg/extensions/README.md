**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

# WCASG Extensions Package

## Predicate

- [Predicate Model](src/types/predicate.ts#L47)

A `Predicate` is a model with one or more functions that return a boolean value. An `Extension` is (optionally) assigned predicates, the results of which determine if the extension's `action(s)` should be executed.

Simple predicate examples include checking if the current date/time falls after a specific timestamp, whether a given Widget plugin is enabled, or of the base page has Wordpress running.

A predicate's `fn` property should be one or more very simple, immutable functions. See [examples](#examples) for more info.

## Extension

- [Extension Model](src/types/extension.ts#L50)

An `extension` is a basic model that defines one or more `action` functions that are executed to perform an arbitrary action, such as execute JS, change CSS, etc. An extension can be (optionally) assigned one or more predicates, the truthiness of which determine if the extension is valid and its action is to be executed.

See [examples](#examples) for more info.

## Development

1. Make changes to code.
2. Run `yarn run build` or `yarn run watch`.

### Adding to WCASG Widget App

1. Navigate to root `wcasg/extensions` directory.
2. Run `yarn link`.
3. Navigate to root `wcasg/widget` directory.
4. Run `yarn link wcasg-extensions` to create local symbolic link.
5. Import extensions package anywhere: `import Extensions from 'wcasg-extensions';`
6. Execute all extensions in code with: `Extensions.manager.runExtensions();`.
7. Rebuild `wcasg/widget` with `yarn run build`.

## Creating a New Extension

1. Add a new file under `src/extensions`. Demo files are roughly organized under framework-specific directories.
2. Create a `new Extension()` instance and pass an object with an `action` property equal to a single function, or an array of functions:

```ts
export const MyExtension = new Extension({
  action: () => console.log('stuff'),
});
// or
export const MyExtension = new Extension({
  action: [() => console.log('stuff'), () => console.log('other stuff')],
});
```

3. (Optionally) Add a `name` property to help identify the extension in log outputs.
4. (Optionally) Add a `predicate` property assigned to a `new Predicate()` instance, or an array of predicates:

```ts
export const MyExtension = new Extension({
  action: () => console.log('stuff'),
  name: 'MyExtensionName'
  predicate: MyPredicate
})
// or
export const MyExtension = new Extension({
  action: () => console.log('stuff'),
  name: 'MyExtensionName'
  predicate: [MyPredicate, MyOtherPredicate]
})
```

5. Add the extension to the `ExtensionManager` you're using. At present it's a single instance exported in the public package API, so just add your extension to the `extensions` property array:

```ts
import { ExtensionManager } from './extension-manager';
import MyExtension from 'extensions/js/MyExtension';

const extensions = (function () {
  return {
    manager: new ExtensionManager({
      extensions: [MyExtension],
    }),
  };
})();

export default extensions;
```

6. Finally, just rebuild the project and any dependency projects (i.e. `wcasg-ada-app`). When the Widget loads it will iterate through all extensions and execute all actions for which the predicates passed, if applicable.

## Examples

### Disable All jQuery Animations

- Predicate: [HasJquery](src/predicates/HasJquery.ts)
- Extension: [js/jquery/DisableAnimations](src/extensions/js/jquery/DisableAnimations.ts)

### Add Missing Aria Labels in Wordpress

- Predicate: [HasWordpress](src/predicates/HasWordpress.ts)
- Extension: [js/wordpress/AddAriaLabels](src/extensions/js/wordpress/AddAriaLabels.ts)

### Enable Screen Reader Only Visuals in Bootstrap

- Predicate: [HasBootstrap](src/predicates/HasBootstrap.ts)
- Extension: [css/bootstrap/AddScreenReaderOnlyClasses](src/extensions/css/bootstrap/AddScreenReaderOnlyClasses.ts)
