# demo-react-npm-module

## How to create a custom NPM Module with React Components

This repo is aimed to describe the process of creating a custom NPM package/module with React.JS components.

It will guide you through the steps of setting up your development environment, writing and exporting your components, and publishing your package to the NPM registry.

## Installation

### Step 1: Make a directory and initialize npm

```bash
  mkdir your-npm-package
  npm init
```

Alternatively, you can use the following command to generate boilerplate:

`npm init -y`
Add the necessary fields. After the successfull walkthrough a package.json will created and looks like below.

```bash
{
  "name": "package-name",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

### Step 2: Folder structure

Create a src folder in the root add a index.js/index.ts in it also create a component folder and creates some components in it.

```bash
src
├── components
│   ├── Component1
│   │   └── index.tsx
│   ├── Component2
│   │   └── index.tsx
│   └── Component3
│       └── index.tsx
└── index.ts
```

Below is a sample component. **Note** that the component is exported as a **named** component and not as a default

```bash
import React from 'react'

export function Component1() {
  return <div>Component1</div>
}
```

The index.ts will look like this;

```bash
export * from './components/component1'
export * from './components/component2'
```


### Step 3: Dependencies

Now, Add react and react-dom as peer-dependencies and dev-dependencies.

`npm install react react-dom --dev`

`npm install --save-dev typescript`

The final package.json may looks like below.

```bash
  {
  "name": "your-npm-package",
  "version": "1.0.0",
  "description": "react npm module",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "npx babel src --out-file index.js --extensions .ts,.tsx"
  },
  "author": "your name",
  "license": "ISC",
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.34",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.4"
  }
}
```

Since it is Typescript. I've also installed typescript, typescript react with it. 

Installing as a dev dependency means it won’t be included in our final package, as it is not necessary once our code has built.

**Note**: _There is an option to use rollup or typescript. We'll explore both._

## Typescript

### Configure the tsconfig.json file
Setup the typescript compiler using `npx tsc --init`, which will create a tsconfig.json file which contains any typescript compilation options.

Now replace the contents with below to ensure our typescript files are generated correctly for our package and in the right location.

```bash
{
  "compilerOptions": {
    "declaration": true /* Generate .d.ts files from TypeScript and JavaScript files in your project. */,
    "module": "esnext" /* Specify what module code is generated. */,
    "outDir": "dist",
    "target": "es5" /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */,
    "lib": ["es6", "dom", "es2016", "es2017"],
    "sourceMap": true,
    "jsx": "react",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */,
    "forceConsistentCasingInFileNames": true /* Ensure that casing is correct in imports. */,
    "strict": true /* Enable all strict type-checking options. */,
    "skipLibCheck": true /* Skip type checking all .d.ts files. */
  },
  "include": ["src/**/*"],
  "exclude": [
    "node_modules",
    "src/**/*.stories.tsx",
    "src/**/*.test.tsx"
  ]
}
```

### Package.json Setup

Make sure to include our `dist` files in the package, and React is included as a peerDependency.

We also need a build script to compile the code.

The `package.json` file should look like the following:

```bash
{
  "name": "your-package-name",
  "version": "1.0.0",
  "description": "This is a custom demo react application to be published to npm.",
  "main": "dist/index.js",
  "files": [
    "dist"
  ],
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  },
  "scripts": {
    "test": "echo \\\"Error: no test specified\\\" && exit 1",
    "build": "tsc"
  },
  "repository": {
    "type": "git",
    "url": "your-repo"
  },
  "keywords": [
    "react",
    "components",
    "ui",
    "npm",
    "package"
  ],
  "author": "your-name",
  "license": "ISC",
  "bugs": {
    "url": "your-repo/issues"
  },
  "homepage": "your-repo#readme",
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@types/react": "^18.3.3",
    "typescript": "^5.5.3"
  }
}
```

## Rollup

### Bundle the code. Install rollup.js

  `npm install --save react rollup`
You will also need to install the Rollup plugins for Babel, CommonJS, and Node Resolve by running

`npm install --save-dev @rollup/plugin-babel @rollup/plugin-commonjs @rollup/plugin-node-resolve @rollup/plugin-terser`

### Configure Rollup

Create a `rollup.config.mjs` or `rollup.config.js` file in the root of your project. 

In this file, configure Rollup to bundle your React component by specifying the input file, output format, and plugins to use. Here is an example configuration:

```bash
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import external from 'rollup-plugin-peer-deps-external';
import terser from '@rollup/plugin-terser';
import typescript from "rollup-plugin-typescript2"; // For Typescript

export default [
  {
    input: './src/index.ts',
    output: [
      {
        file: 'dist/index.js',
        format: 'cjs',
      },
      {
        file: 'dist/index.es.js',
        format: 'es',
        exports: 'named',
      }
    ],
    plugins: [
      babel({
        exclude: 'node_modules/**',
        presets: ['@babel/preset-react']
      }),
      external({
        includeDependencies: true
      }),
      resolve(),
      terser(),
      typescript({ useTsconfigDeclarationDir: true }),
    ]
  }
];
```

To our package.json, let's add this as build command so that we won't have to type it repeatedly. Under "scripts" in package.json add the following.

```bash
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "npx rollup -c"
  }
```

now we can simply use npm run build or yarn build to bundle the code.

If it compiled without any error you may see a new folder dist generated in the root folder with transpiled Javascript Code.

**Note**: that if you're using a build tool like Webpack, you may not need to use npx babel at all. Instead, you can use a TypeScript loader like ts-loader or awesome-typescript-loader that will transpile and transform your TypeScript files as part of the build process.

## Test the package in other projects. 

Make sure to build the package first using the `npm run build` command.

**There's to ways to test the package;**

Run the following command from the root folder.

  `npm pack`
This will generate a .tgz file that we can use to install the package locally in other projects.

**Alternatively**, you can link the package for testing purpose [**Recommended**].

Using `npm link` in your package's root directory, create a global symlink of your package. A shortcut that directs your system to another directory or file is known as a "symlink," short for symbolic link.

**Note:** If npm link gives a `code: 'EACCES'` error, use `sudo npm link`. You might need to enter your password. The error should be solved

Now, create an another application and tell the application to use the global symlink with `npm link your-package-name`. Import and use the package like below.

```bash
import { Component1, Component2, Component2 } from '_your-custom-package_'

export default function Home() {
  return (
    <main>
        <Component1 />
        <Component2 />
        <Component3 />
    </main>
  )
}

```

This way, we could save a lot of time 🙂.

## Publish your package to NPM Registry

When satisfied after testing,you could now publish a package to the npm registry, using these steps:

1. Create an npm account on npmjs.com.
2. Sign in to npm from your terminal using the `npm login` command.
3. Prepare your package for publishing.
4. Publish the package from the terminal using the `npm publish` command.

### Navigate to your other project's directory and install your package using npm install . For example:

  `npm i 'path_to_the_packed_file'`

### In your host project's code you can import and use your component as follows:

```bash
  import "./App.css";
  import TextInput from "my_npm_package"; **// Importing from the package**
  import { useState } from "react";

  export default function App() {
    const [value, setValue] = useState('');

    const handleChange(e)= () => setValue(e.target.value);

    return (
      <div className="App">
          <p>My npm module.</p>
          <TextInput label="Name" value={value} onChange={handleChange} />
      </div>
    );
  }
```

Additionally we can use packages like storybook.js to build UI components and pages in isolation and by doing so we could see the changes that happens to our components without the need of installing it in a host application. You could find the documention for implementing story book here.
