# Format with Prettier & ESLint

```
npx eslint --init
```

```
npm i -D prettier
```

```
npm i -D eslint-config-prettier
```

**.eslintrc.js**

```js
module.exports = {
  "env": {
    "browser": true,
    "es2021": true
  },
  "settings": {
    "react":
    {
      "version": 'detect'
    }
  },
  "extends": [
    "standard-with-typescript",
    "plugin:react/recommended",
    "plugin:react/jsx-runtime",
    'eslint-config-prettier'
  ],
  "overrides": [
    {
      "env": {
        "node": true
      },
      "files": [
        ".eslintrc.{js,cjs}"
      ],
      "parserOptions": {
        "sourceType": "script"
      }
    }
  ],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": [
    "react"
  ],
  "rules": {
    'no-unused-vars': 'warn',
  }
}

```

**.prettierrc**

```json
{
    "userTabs": true,
    "semi": false,
    "singleQuote": true,
    "jsxSingleQuote": true,
    "trailingComma": "es5",
    "bracketSpacing": true,
    "bracketSameLine": true,
    "arrowParens": "avoid"
}
```

## Another Approach

```pnpm install ts-standard -D
pnpm install ts-standard -D
```

.eslint.cjs
```
module.exports = {
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    './node_modules/ts-standard/eslintrc.json'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: { ecmaVersion: 'latest', sourceType: 'module', project: './tsconfig.json' },
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-floating-promises': 'off'
  }
}
```

.tsconfig

```
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "module": "ESNext",
    "skipLibCheck": true,
    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src", "./.eslintrc.cjs"],
  "references": [
    {
      "path": "./tsconfig.node.json"
    }
  ]
}

```

