## Notes to myself

Configuring a TypeScript project with Node (and express) made me go through the headache of using tools like tsc, ts-node, tsx, nodemon, etc.

After some research I came out with this simple configuration to reuse within future porjects.

### `package.json`

- `"type": "module"` to use ES Modules throughout the package.
- `"main": "dist/index.js"` is the main file because tsup will compile there.
- If we are working on an [internal package in a monorepo setup](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages) we don't have the dist folder because our internal packges is not compiled. Instead it is imported by another package as a dependency. The importing package is responsible to bundle the imported package within its code along with the other dependencies. More on this when I talk about tsup. In this case we set the `"main": "src/index.ts"`.

### `tsconfig.json`

- All the base and the strictness options are absolutely required in any project.
- We will use `tsup` (esbuild) as an external bundler to compile our TypeScript to JavaScript instead of tsc. We only use TypeScript as a linter and not for code generation.
- `"module": "ESNext"` to use TypeScript with ES Modules. This works better when `"type": "module"` in `package.json`.
- `"moduleResolution": "Bundler"` is to use `tsup` (external compiler built on top of esbuild) to compile the TypeScript code instead of tsc.
- `"noEmit": "true"` because the code is going to be emitted by tsup and not tsc.
- No need to specify `"target"`, `"sourceMap"`, `"outDir"` ...etc as they will be configured in tsup.

### `tsup.config.ts`

- We specify the `entry` file of the package as well as the `target :"esnext"` and the `format`.
- We can add the `noExternal` option to also bundle code that we import from external packages. For example, a shared internal packages declared in another workspace in the same monorepo.

Lastly, we add the scripts to `package.json`

```
  "scripts": {
    "dev": "tsup src/index.ts --watch --onSuccess \"node dist/index.js\"",
    "build": "tsup src/index.ts",
    "start": "node dist/index.js"
  }
```

### Why we use tsup for compiling?

- Main reasons to consider tsup are its performance and ease of use. By using esbuild under the hood, tsup is able to quickly bundle our TypeScript code, making it a great choice for large projects, or really, any typescript project.

- Additionally, tsup has a built in `onSuccess` flag that would re-run the bundled code, removing the need for installing nodemon.

### Why don't we use tsx or ts-node with nodemon?

- First, they are slow and give headache especially when trying to work with ES Modules and path aliases.
- Second, when we build and compile to prod, we have to use tsc. and tsc is slow. If we have `"moduleResolution"` option set to `"ESNext"` or `"NodeNext"` we have to specify the `.js` or `.cjs` extension when importing modules even if we are importing from `.ts` file which is pretty confusing !!! tsup just makes it very easy to use as it handles the module resolution for us (we can import modules without specifying the extension).

##### References

- <https://plusreturn.com/blog/build-better-and-faster-bundles-with-typescript-and-express-using-tsup>
- <https://www.totaltypescript.com/relative-import-paths-need-explicit-file-extensions-in-ecmascript-imports>
- <https://blog.atomrc.dev/p/typescript-library-compilation-2023>
- <https://www.totaltypescript.com/tsconfig-cheat-sheet>
