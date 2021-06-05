# ESLint

Linting is analysing code for errors.  ESLint is a linter.

### Installing eslint

Install eslint and eslint-plugin-react.

We use --save-dev as these packages are only required during development, not build.

```text
npm i eslint eslint-plugin-react --save-dev
```

ESLint is driven by a configuration file, eslintrc.json.  This file tells ESLint what frameworks and style to expect when it analyses the code.  

There's many, many rules and options we can set here depending on what we want ESLint to check.

This is our current config for react.

```json
{
    "env": {
        "browser": true,
        "es6": true,
        "mocha": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "sourceType": "module"
    },
    "plugins": [
      "react"
    ],
    "rules": {
        "no-const-assign": "warn",
        "no-this-before-super": "warn",
        "no-undef": "warn",
        "no-unreachable": "warn",
        "no-unused-vars": "warn",
        "constructor-super": "warn",
        "valid-typeof": "warn"
    }
}
```

### Running eslint 

To run eslint on the command line:

```text
./node_modules/.bin/eslint src
```

To run eslint in WebStorm:

* Go to File > Settings > Languages & Frameworks > JavaScript > Code Quality Tools > ESLint  
* Click "Enable"

The inspection should run automatically and highlight errors in the code.  

To manually trigger ESLint and see a list of errors:

* Code > Run Inspection By Name - "ESLint"
