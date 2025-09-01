# jest-resolve-no-esm

A [jest-resolve](https://npm.im/jest-resolve) fork that optionally disables the
[self-described "bad version" of Jest's attempt](https://github.com/jestjs/jest/blob/2737e08b33d1063ac99c4e3f0973551fe34427ee/packages/jest-resolve/src/shouldLoadAsEsm.ts#L43-L72)
at replicating
[Node's type resolution algorithm](https://github.com/nodejs/modules/issues/393).

## Motivation

[Jest's ESM support is too incomplete](https://github.com/jestjs/jest/issues/9430).
Specifically:

- There is no `jest.requireActual` function (e.g. a `jest.importActual`) for ES
  modules. This makes using `jest.unstable_mockModule`
  [too annoying](https://github.com/jestjs/jest/issues/15303) since, among other
  difficulties,
  [I can't seem to import a module with the goal of modifying only a single of its exports while leaving the rest as they are](https://github.com/jestjs/jest/issues/9430#issuecomment-3174004784).
  With something like `jest.spyOn` and
  [babel-plugin-explicit-exports-references](https://npm.im/babel-plugin-explicit-exports-references),
  this becomes is trivial. Unfortunately...

- There is no `jest.spyOn` support in ESM mode because, as far as current Jest
  spying functionality is concerned,
  [ES modules are immutable](https://stackoverflow.com/questions/73379206/jest-standard-way-to-stub-named-exports-of-esm-modules).

So, the solution is to just transpile everything back to CJS so we can regain
our mocking and spying abilities. This has been my solution for a long time and
it's been working perfectly fine up until (relatively) recently, where Jest
updated its resolution algorithm in an attempt to progress its ESM support.

An unfortunate side-effect of this is that, even when
`transformIgnorePatterns: []` and `transform: { ... }` are configured in a Jest
config file for full CJS transpilation, jest-resolve will still attempt to load
the now-transpiled-CJS-module as if it were ESM just because its file names end
in `.js` and the nearest `package.json` file contains `"type": "module"`.

This fork allows me to disable ESM loading when
`process.env.JEST_RESOLVE_NO_ESM` is truthy and avoid false positive "errors"
like `Must use import to load ES Module: ...`.

## Installation and Usage

1. Install:

```shell
npm install --save-dev jest-resolve@npm:jest-resolve-no-esm jest
```

2. Activate:

```
JEST_RESOLVE_NO_ESM=true npx jest
```

Alternatively, you could set `process.env.JEST_RESOLVE_NO_ESM = 'true'` in a
Jest [config file](https://jestjs.io/docs/configuration) or
[setup file](https://jestjs.io/docs/configuration#setupfiles-array).

For example, [unified-utils](https://github.com/Xunnamius/unified-utils) uses
this package to test its ESM-only packages.
