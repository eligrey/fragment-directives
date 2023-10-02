# Fragment Directives API explainer

A [Proposal](https://privacycg.github.io/charter.html#proposals)
of the [Privacy Community Group](https://privacycg.github.io/).

## Authors:

- [Eli Grey](https://dangerous.link/virus.exe)

## Participate
- https://github.com/eligrey/fragment-directives/issues

## Introduction

Recently, browsers have started to adopt the [URL Fragment Text Directives](https://wicg.github.io/scroll-to-text-fragment/) aka scroll-to-text specification. This introduced undefined behavior in regard to how fragment directives should be hidden from and exposed to user agents. This standard aims to codify a way to query fragment directives from all supported location interfaces while accounting for the privacy concerns introduced by accessing potentially privacy-sensitive directives.

### Motivating Use Cases

[Fragment directives](https://wicg.github.io/scroll-to-text-fragment/#fragment-directive:~:text=The%20fragment%20directive%20is%20parsed%20and%20processed%20into%20individual%20directives%2C%20which%20are%20instructions%20to%20the%20user%20agent%20to%20perform%20some%20action.%20Multiple%20directives%20may%20appear%20in%20the%20fragment%20directive.) are instructions for user agents to perform some action. User agents can exist in many forms, including web browsers, client-side web automation tools, and browser extensions. The effect of the scroll-to-text standard is that these fragment directives were hidden from location interfaces for compatibility reasons.

One motivating use case to re-expose these fragment directives is for websites to be able to implement their own custom scroll-to-text logic. Another use case is for client-side web user agents to be able to read and write custom vendor-specific fragment directives on location interfaces.

### Privacy

Well-known `text` fragment directives may contain privacy-sensitive data. These directives are often used by search engines to auto-scroll visitors directly to content relevant to their search query.

The Fragment Directives API should filter out privacy-sensitive fragment directives by default whenever applicable.

## API

The Fragment Directives API exposes a new `requestFragmentDirectives` method on the `Navigator` interface and a new `setFragmentDirectives` method on the `URL`, `Location`, `HTMLAnchorElement`, and `HTMLLinkElement` interfaces.

The `navigator.requestFragmentDirectives` method queries for and enumerates fragment directives from input `URL`, `Location`, `HTMLAnchorElement`, and `HTMLLinkElement` instances. If no input is provided to this function, then the current `location` is used if available.

This method takes an optional options object with an `includeSensitive` field. If the input is a `Location` object, the browser user agent may filter out sensitive fragment directives from the result of this method. Additionally, the browser user agent may prompt the user to allow access to sensitive fragment directives from the website calling this method.

If the browser user agent chooses to prompt the user for sensitive fragment directive access permission for the current origin, then the permission choice should be persisted for this origin during future visits until cleared.

## Example use cases

### Custom scroll-to-text

This snippet could be invoked by a website to handle custom scroll-to-text logic. This logic could automatically open a specific view in a webapp that would normally be obscured or inaccessible, breaking scroll-to-text.

```js
const directives = await navigator?.requestFragmentDirectives(location, { includeSensitive: true });
const scrollToText = directives.getAll('text');
if (scrollToText) {
  // implement custom scroll-to-text
}
```

### Vendor-specific fragment directives

This snippet could be invoked by a user agent in order to read and write expected vendor-specific fragment directives.

```js
// on the previous page:
const directives = new URLSearchParams();
directives.set('my-custom-directive', '...');
const url = new URL('/some-page', location);
url.setFragmentDirectives?.(directives);
navigator.navigate(url);

// later on in /some-page:
const directives = await navigator.requestFragmentDirectives?.();
const customDirective = directives?.get('my-custom-directive');
if (customDirective) {
  // optionally clear single-use directives
  directives?.delete('my-custom-directive');
  location.setFragmentDirectives?.(directives.size === 0 ? null : directives);

  // handle directive
  handleMyCustomDirective(customDirective);
}
```

## Interface definitions

```ts
interface RequestFragmentDirectives {
  (source?: Location | URL, options?: NavigatorRequestFragmentDirectivesOptions): Promise<URLSearchParams>;
}

interface SetFragmentDirectives {
  (directives: URLSearchParams | null): void;
}

interface RequestFragmentDirectivesOptions {
  includeSensitive?: boolean;
}

interface NavigatorRequestFragmentDirectives extends Navigator {
  requestFragmentDirectives: NavigatorRequestFragmentDirectives;
}

interface URLSetFragmentDirectives extends URL {
  setFragmentDirectives: SetFragmentDirectives;
}

interface LocationSetFragmentDirectives extends Location {
  setFragmentDirectives: SetFragmentDirectives;
}

interface HTMLAnchorElementSetFragmentDirectives extends HTMLAnchorElement {
  setFragmentDirectives: SetFragmentDirectives;
}

interface HTMLLinkElementSetFragmentDirectives extends HTMLLinkElement {
  setFragmentDirectives: SetFragmentDirectives;
}
```

## Stakeholder Feedback / Opposition

- Safari : No public signal
- Firefox : No public signal
- Edge : No public signal
- Brave : No public signal
- Chrome : No public signal