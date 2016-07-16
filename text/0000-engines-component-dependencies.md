- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Ember Issue: (leave this empty)

# Summary

This RFC proposes to allow the declaration of components as dependencies of engines.

# Motivation

Currently, the only way to allow engines to share components with their consuming application/engine is to extract the component and package it as a standalone add-on. The add-on can then be installed by any application/engine that requires it.

While this approach is a perfectly viable option in some circumstances, the viability begins to breakdown in distributed applications where the developer may not have control of some or all of the engines being consumed.

In such circumstances it becomes difficult to ensure consistency throughout the application as each engine could theoretically be using a different version of the same component, potentially resulting in different areas of the application having different UI/UX for the same component/functionality.

Allowing engines to declare components as dependencies would resolve this issue of consistency. If any changes were made to components in the main application, these changes would be able flow down through to each of the mounted engines without requiring the engine developers to change any of their code.

# Detailed design

As it is already possible to declare services as dependencies of engines, the underlying groundwork should already be in place to also allow components to be declared as dependencies. Declaring a component dependency would look and work exactly the same way as declaring service dependencies.

Consider the following, using the `ember-blog` engine as an example:

### Declaring Dependencies (Engine)

As with services, the engine would be required to declare any components that they expected to be supplied as dependencies.

```js
// ...

export default Engine.extend({
  modulePrefix: 'ember-blog',
  Resolver,

  dependencies: {
    components: [
      'custom-button',
      'custom-heading'
    ]
  }
});
```


### Fulfilling Dependencies (Consuming Application/Engine)

Consuming applications/engines would be required to fulfil the component dependencies declared by the engine. This would also work exactly the same as fulfilling service dependencies in engines today.

In order to mitigate the risk of naming collisions, it would be possible to re-map component dependencies to a custom key. See the 'heading' component dependency in the example below.

```js

// ...

App = Ember.Application.extend({
  modulePrefix: config.modulePrefix,
  podModulePrefix: config.podModulePrefix,
  Resolver,

  engines: {
    emberBlog: {
      dependencies: {
        components: [
          'custom-button',
          {'heading': 'custom-heading'}
        ]
      }
    }
  }
});

// ...
```

# How We Teach This

This RFC doesn't introduce any new patterns or terminology, but instead builds upon functionality that currently exists in engines today. As a result, the learning curve of either engines or Ember itself shouldn't be affected and any documentation could be added along with the engines guides themselves.

# Drawbacks

### Versioning / Breaking Changes
The main drawback to this approach is the potential for breaking changes. Any changes made to dependent components in the parent application/engine would automatically flow through to the mounted engines. As a result, if a breaking change was made, it would cause issues in any mounted engines that were using the changed component.

While this is a valid issue, I believe that the risk is low due to the fact that the main use case for this feature will be in distributed applications, where engines are being used as a means of allowing developers to extend the main application. These applications in a majority of cases would already be using versioning and changelogs at a higher level as a way of communicating breaking changes to their users/developers.

# Alternatives

### Standalone Add-ons
Extracting components into standalone add-ons (as already mentioned previously) is the main alternative to the approach in this RFC. The two main downsides to this approach are as follows:

1. Introduces the consistency issue described previously in the "Motivation" section.
2. Makes it difficult to create extendable distributed applications that would like to supply sets of components in order to ease the process of extending the application.

# Unresolved questions

None