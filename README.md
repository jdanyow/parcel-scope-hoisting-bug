# scope hoisting bug?

Three files, two entry points:


foo.ts
```ts
export const foo1 = 'foo1';
export const foo2 = 'foo2';
export const foo3 = 'foo3'; // not referenced. should not appear in bundle.
```

index-a.ts (first entry point)
```ts
import { foo1 } from "./foo";

console.log(foo1);
```

index-b.ts (second entry point)
```ts
import { foo2 } from "./foo";

console.log(foo2);
```

To reproduce, execute `yarn run build` which is `parcel build index-*.ts --experimental-scope-hoisting --no-cache`.
The resulting bundles contain `foo3` even though it is not referenced.

dist/index-a.js
```js
parcelRequire=function(e){var r="function"==typeof parcelRequire&&parcelRequire,n="function"==typeof require&&require,i={};function u(e,u){if(e in i)return i[e];var t="function"==typeof parcelRequire&&parcelRequire;if(!u&&t)return t(e,!0);if(r)return r(e,!0);if(n&&"string"==typeof e)return n(e);var o=new Error("Cannot find module '"+e+"'");throw o.code="MODULE_NOT_FOUND",o}return u.register=function(e,r){i[e]=r},i=e(u),u.modules=i,u}(function (require) {var a={},b="foo1";a.foo1=b;var c="foo2";a.foo2=c;var d="foo3";a.foo3=d;console.log(b);a.__esModule=true;return{"m60s":{},"P2gg":a};});
```

dist/index-b.js
```js
parcelRequire=function(e){var r="function"==typeof parcelRequire&&parcelRequire,n="function"==typeof require&&require,i={};function u(e,u){if(e in i)return i[e];var t="function"==typeof parcelRequire&&parcelRequire;if(!u&&t)return t(e,!0);if(r)return r(e,!0);if(n&&"string"==typeof e)return n(e);var o=new Error("Cannot find module '"+e+"'");throw o.code="MODULE_NOT_FOUND",o}return u.register=function(e,r){i[e]=r},i=e(u),u.modules=i,u}(function (require) {var a={},c="foo1";a.foo1=c;var b="foo2";a.foo2=b;var d="foo3";a.foo3=d;console.log(b);a.__esModule=true;return{"e8wD":{},"P2gg":a};});
```

The strange thing is, executing `yarn run build-a` (`parcel build index-a.ts --experimental-scope-hoisting --no-cache`) will bundle and scope hoist as expected:

dist/index-a.js
```js
(function () {var a="foo1";console.log(a);})();
```