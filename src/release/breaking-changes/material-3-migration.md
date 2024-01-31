---
title: Migrate to Material 3
description: >-
  Learn how to migrate your Flutter app's UI from Material 2 to Material 3.
---

## Summary

The Material library has been updated to match the Material 3 Design spec.
Changes include new components and component themes, updated component visuals,
and much more. Many of these updates are seamless. You’ll see the new version
of an affected widget when recompiling your app against the 3.16 (or later)
release. But some manual work is also required to complete the migration.

## Migration guide

Prior to the 3.16 release, you could opt in to the Material 3 changes by
setting the `useMaterial3` flag to true. As of the Flutter 3.16 release
(November 2023), `useMaterial3` is true by default.

By the way, you _can_ revert to Material 2 behavior in your app by setting the
`useMaterial3` to `false`. However, this is just a temporary solution. The
`useMaterial3` flag _and_ the Material 2 implementation will eventually be
removed as part of Flutter’s deprecation policy.

### Colors

The default values for `ThemeData.colorScheme` are updated to match
the Material 3 Design spec.

The `ColorScheme.fromSeed` constructor generates a `ColorScheme`
derived from the given `seedColor`. The colors generated by this
constructor are designed to work well together and meet contrast
requirements for accessibility in the Material 3 Design system.

When updating to the 3.16 release, your UI might look a little strange
without the correct `ColorScheme`. To fix this, migrate to the
`ColorScheme` generated from the `ColorScheme.fromSeed` constructor.

Code before migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.light(primary: Colors.blue),
),
```

Code after migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
),
```

To generate a content-based dynamic color scheme, use the
`ColorScheme.fromImageProvider` static method. For an example of generating a
color scheme, check out the [`ColorScheme` from a network image][] sample.

[`ColorScheme` from a network image]: {{site.api}}/flutter/material/ColorScheme/fromImageProvider.html

Changes to Flutter Material 3 include a new background color.
`ColorScheme.surfaceTint` indicates an elevated widget.
Some widgets use different colors.

To return your app’s UI to its previous behavior (which we don't recommend):
* Set `Colors.grey[50]!` to `ColorScheme.background`
   (when the theme is `Brightness.light`).
* Set  `Colors.grey[850]!`to `ColorScheme.background`
   (when the theme is `Brightness.dark`).

Code before migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
),
```

Code after migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple).copyWith(
    background: Colors.grey[50]!,
  ),
),
```

```dart
darkTheme: ThemeData(
  colorScheme: ColorScheme.fromSeed(
    seedColor: Colors.deepPurple,
    brightness: Brightness.dark,
  ).copyWith(background: Colors.grey[850]!),
),
```

The `ColorScheme.surfaceTint` value indicates a component's elevation in
Material 3. Some widgets might use both `surfaceTint` and `shadowColor` to
indicate elevation (for example, `Card` and `ElevatedButton`) and others might
only use `surfaceTint` to indicate elevation (such as `AppBar`).

To return to the widget’s previous behavior, set, set `Colors.transparent`
to `ColorScheme.surfaceTint` in the theme. To differentiate a widget’s shadow
from the content (when it has no shadow), set the `ColorScheme.shadow` color to
the `shadowColor` property in the widget theme without a default shadow color.

Code before migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
),
```

Code after migration:

```dart
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple).copyWith(
    surfaceTint: Colors.transparent,
  ),
  appBarTheme: AppBarTheme(
   elevation: 4.0,
   shadowColor: Theme.of(context).colorScheme.shadow,
 ),
),
```

The `ElevatedButton` now styles itself with a new combination of colors.
Previously, when the `useMaterial3` flag was set to false, `ElevatedButton`
styled itself with `ColorScheme.primary` for the background and
`ColorScheme.onPrimary` for the foreground. To achieve the same visuals, switch
to the new `FilledButton` widget without the elevation changes or drop shadow.

Code before migration:

```dart
ElevatedButton(
  onPressed: () {},
  child: const Text('Button'),
),
```

Code after migration:

```dart
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: Theme.of(context).colorScheme.primary,
    foregroundColor: Theme.of(context).colorScheme.onPrimary,
  ),
  onPressed: () {},
  child: const Text('Button'),
),
```

### Typography

The default values for `ThemeData.textTheme` are updated to match the
Material 3 defaults. Changes include updated font size, font weight, letter
spacing, and line height. For more details, check out the [`TextTheme`][]
documentation.

As shown in the following example, prior to the 3.16 release, when a `Text`
widget with a long string using `TextTheme.bodyLarge` in a constrained layout
wrapped the text into two lines. However, the 3.16 release wraps the text into
three lines. If you must achieve the previous behavior, adjust the text style
and, if necessary, the letter spacing.

Code before migration:

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 200),
    child: Text(
      'This is a very long text that should wrap to multiple lines.',
      style: Theme.of(context).textTheme.bodyLarge,
  ),
),
```

Code after migration:

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 200),
    child: Text(
      'This is a very long text that should wrap to multiple lines.',
      style: Theme.of(context).textTheme.bodyMedium!.copyWith(
        letterSpacing: 0.0,
      ),
  ),
),
```

[`TextTheme`]: {{site.api}}/flutter/material/TextTheme-class.html

### Components

Some components couldn't merely be updated to match the Material 3 Design spec
but needed a whole new implementation. Such components require manual migration
since the Flutter SDK doesn’t know what, exactly, you want.

Replace the Material 2 style [`BottomNavigationBar`][] widget with the new
[`NavigationBar`][] widget. It’s slightly taller, contains pill-shaped
navigation indicators, and uses new color mappings.

Code before migration:

```dart
BottomNavigationBar(
  items: const <BottomNavigationBarItem>[
    BottomNavigationBarItem(
      icon: Icon(Icons.home),
      label: 'Home',
    ),
    BottomNavigationBarItem(
      icon: Icon(Icons.business),
      label: 'Business',
    ),
    BottomNavigationBarItem(
      icon: Icon(Icons.school),
      label: 'School',
    ),
  ],
),
```

Code after migration:

```dart
NavigationBar(
  destinations: const <Widget>[
    NavigationDestination(
      icon: Icon(Icons.home),
      label: 'Home',
    ),
    NavigationDestination(
      icon: Icon(Icons.business),
      label: 'Business',
    ),
    NavigationDestination(
      icon: Icon(Icons.school),
      label: 'School',
    ),
  ],
),
```

Check out the complete sample on
[migrating from `BottomNavigationBar` to `NavigationBar`][].

Replace the [`Drawer`][] widget with [`NavigationDrawer`][], which provides
pill-shaped navigation indicators, rounded corners, and new color mappings.

Code before migration:

```dart
Drawer(
  child: ListView(
    children: <Widget>[
      DrawerHeader(
        child: Text(
          'Drawer Header',
          style: Theme.of(context).textTheme.titleLarge,
        ),
      ),
      ListTile(
        leading: const Icon(Icons.message),
        title: const Text('Messages'),
        onTap: () { },
      ),
      ListTile(
        leading: const Icon(Icons.account_circle),
        title: const Text('Profile'),
        onTap: () {},
      ),
      ListTile(
        leading: const Icon(Icons.settings),
        title: const Text('Settings'),
        onTap: () { },
      ),
    ],
  ),
),
```

Code after migration:

```dart
NavigationDrawer(
  children: <Widget>[
    DrawerHeader(
      child: Text(
        'Drawer Header',
        style: Theme.of(context).textTheme.titleLarge,
      ),
    ),
    const NavigationDrawerDestination(
      icon: Icon(Icons.message),
      label: Text('Messages'),
    ),
    const NavigationDrawerDestination(
      icon: Icon(Icons.account_circle),
      label: Text('Profile'),
    ),
    const NavigationDrawerDestination(
      icon: Icon(Icons.settings),
      label: Text('Settings'),
    ),
  ],
),
```

Check out the complete sample on [migrating from `Drawer` to `NavigationDrawer`][].

Material 3 introduces medium and large app bars that display a larger headline
before scrolling. Instead of a drop shadow, `ColorScheme.surfaceTint` color
is used create a separation from the content when scrolling.

The following code demonstrates how to implement the medium app bar:

```dart
CustomScrollView(
  slivers: <Widget>[
    const SliverAppBar.medium(
      title: Text('Title'),
    ),
    SliverToBoxAdapter(
      child: Card(
        child: SizedBox(
          height: 1200,
          child: Padding(
            padding: const EdgeInsets.fromLTRB(8, 100, 8, 100),
            child: Text(
              'Here be scrolling content...',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
          ),
        ),
      ),
    ),
  ],
),
```

There are now two types of [`TabBar`][] widgets: primary and secondary.
Secondary tabs are used within a content area to further separate
related content and establish hierarchy. Check out the [`TabBar.secondary`][]
example.

The new [`TabBar.tabAlignment`][] property specifies the horizontal alignment
of the tabs.

The following sample shows how to modify tab alignment in a scrollable `TabBar`:

```dart
AppBar(
  title: const Text('Title'),
  bottom: const TabBar(
    tabAlignment: TabAlignment.start,
    isScrollable: true,
    tabs: <Widget>[
      Tab(
        icon: Icon(Icons.cloud_outlined),
      ),
      Tab(
        icon: Icon(Icons.beach_access_sharp),
      ),
      Tab(
        icon: Icon(Icons.brightness_5_sharp),
      ),
    ],
  ),
),
```

[`SegmentedButton`][], an updated version of [`ToggleButtons`][],
uses fully rounded corners, differs in layout height and
size, and uses a Dart `Set` to determine selected items.

Code before migration:

```dart
enum Weather { cloudy, rainy, sunny }

ToggleButtons(
  isSelected: const [false, true, false],
  onPressed: (int newSelection) { },
  children: const <Widget>[
    Icon(Icons.cloud_outlined),
    Icon(Icons.beach_access_sharp),
    Icon(Icons.brightness_5_sharp),
  ],
),
```

Code after migration:

```dart
enum Weather { cloudy, rainy, sunny }

SegmentedButton<Weather>(
  selected: const <Weather>{Weather.rainy},
  onSelectionChanged: (Set<Weather> newSelection) { },
  segments: const <ButtonSegment<Weather>>[
    ButtonSegment(
      icon: Icon(Icons.cloud_outlined),
      value: Weather.cloudy,
    ),
    ButtonSegment(
      icon: Icon(Icons.beach_access_sharp),
      value: Weather.rainy,
    ),
    ButtonSegment(
      icon: Icon(Icons.brightness_5_sharp),
      value: Weather.sunny,
    ),
  ],
),
```

Check out the complete sample on
[migrating from `ToggleButtons` to `SegmentedButton`][].

[migrating from `BottomNavigationBar` to `NavigationBar`]: {{site.api}}/flutter/material/BottomNavigationBar-class.html#material.BottomNavigationBar.2
[migrating from `Drawer` to `NavigationDrawer`]: {{site.api}}/flutter/material/Drawer-class.html#material.Drawer.2
[migrating from `ToggleButtons` to `SegmentedButton`]: {{site.api}}/flutter/material/ToggleButtons-class.html#material.ToggleButtons.1

#### New components

 * "Menu bars and cascading menus" provide a desktop-style menu system that is
    fully traversable with the mouse or keyboard. Menus are anchored by
    a [`MenuBar`][] or a [`MenuAnchor`][]. The new menu system isn't
    something that existing applications must migrate to, however
    applications that are deployed on the web or on desktop platforms
    should consider using it instead of `PopupMenuButton` (and related) classes.
 * [`DropdownMenu`][] combines a text field and a menu to
    produce what's sometimes called a _combo box_. Users can select
    a menu item from a potentially large list by entering a
    matching string or by interacting with the menu with touch, mouse,
    or keyboard. This can be a good replacement for `DropdownButton`
    widget, although it isn’t necessary.
 * [`SearchBar`][] and [`SearchAnchor`][] are for interactions where the
    user enters a search query, the app computes a list of matching
    responses, and then the user either selects one or adjusts the
    query.
 * [`Badge`][] decorates its child with a small label of just a few
    characters. Like '+1'. Badges are typically used to decorate the icon
    within a `NavigationDestination`, a `NavigationRailDestination`,
    A `NavigationDrawerDestination`, or a button's icon, as in
    `TextButton.icon`.
 * [`FilledButton`] and [`FilledButton.tonal`][] are very similar to an
    `ElevatedButton` without the elevation changes and drop shadow.
 * [`FilterChip.elevated`][], [`ChoiceChip.elevated`][], and
    [`ActionChip.elevated`] are elevated variants of the same chips
    with a drop shadow and a fill color.
 * [`Dialog.fullscreen`][]  fills the entire screen and
    typically contains a title, an action button, and a close button
    at the top.

[`BottomNavigationBar`]: {{site.api}}/flutter/material/BottomNavigationBar-class.html
[`NavigationBar`]: {{site.api}}/flutter/material/NavigationBar-class.html
[`Drawer`]: {{site.api}}/flutter/material/Drawer-class.html
[`NavigationDrawer`]: {{site.api}}/flutter/material/NavigationDrawer-class.html
[`TabBar`]: {{site.api}}/flutter/material/TabBar-class.html
[`TabBar.secondary`]: {{site.api}}/flutter/material/TabBar/TabBar.secondary.html
[`TabBar.tabAlignment`]: {{site.api}}/flutter/material/TabBar/tabAlignment.html
[`SegmentedButton`]: {{site.api}}/flutter/material/SegmentedButton-class.html
[`ToggleButtons`]: {{site.api}}/flutter/material/ToggleButtons-class.html
[`MenuBar`]: {{site.api}}/flutter/material/MenuBar-class.html
[`MenuAnchor`]: {{site.api}}/flutter/material/MenuAnchor-class.html
[`DropdownMenu`]: {{site.api}}/flutter/material/DropdownMenu-class.html
[`SearchBar`]: {{site.api}}/flutter/material/SearchBar-class.html
[`SearchAnchor`]: {{site.api}}/flutter/material/SearchAnchor-class.html
[`Badge`]: {{site.api}}/flutter/material/Badge-class.html
[`FilledButton`]: {{site.api}}/flutter/material/FilledButton-class.html
[`FilledButton.tonal`]: {{site.api}}/flutter/material/FilledButton/FilledButton.tonal.html
[`FilterChip.elevated`]: {{site.api}}/flutter/material/FilterChip/FilterChip.elevated.html
[`ChoiceChip.elevated`]: {{site.api}}/flutter/material/ChoiceChip/ChoiceChip.elevated.html
[`ActionChip.elevated`]: {{site.api}}/flutter/material/ActionChip/ActionChip.elevated.html
[`Dialog.fullscreen`]: {{site.api}}/flutter/material/Dialog/Dialog.fullscreen.html

## Timeline

In stable release: 3.16

## References

Documentation:

* [Material Design for Flutter][]

API documentation:

* [`ThemeData.useMaterial3`][]

Relevant issues:

* [Material 3 umbrella issue][]

Relevant PRs:

* [Change the default for `ThemeData.useMaterial3` to true][]
* [Updated `ThemeData.useMaterial3` API doc, default is true][]


[Material 3 umbrella issue]: {{site.repo.flutter}}/issues/91605
[Material Design for Flutter]: {{site.url}}/ui/design/material
[`ThemeData.useMaterial3`]: {{site.api}}/flutter/material/ThemeData/useMaterial3.html
[Change the default for `ThemeData.useMaterial3` to true]: {{site.repo.flutter}}/pull/129724
[Updated `ThemeData.useMaterial3` API doc, default is true]: {{site.repo.flutter}}/pull/130764