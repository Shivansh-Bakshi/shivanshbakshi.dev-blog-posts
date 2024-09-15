---
title: "Setting up Next.js for an efficient and scalable architecture"
date: 2024-09-15
synopsis: Build an efficient base for your Next.js application to maintain high standards of coding and improve overall development experience
cover: ./pt-1/cover.jpeg
---

## Why am I switching to Next.js

I built my website [shivanshbakshi.dev](https://shivanshbakshi.dev) back in January 2023. I have always wanted to have one of those cool-looking portfolios that could really showcase my abilities. Back then, I was new to the field of Web Development. The website was my first big project, which I developed over a period of a month with thoughtful considerations (to the best of my then abilities) about performance and optimization. I used Gatsby with JavaScript, since I did not know much about TypeScript, or what I would be getting myself into when I locked myself within the limits of static websites.

Any update such as changing the Resume or adding a new experience, a new project, a new certificate, required me to directly add them in the code and trigger a rebuild and redeployment of the whole website. You can quickly see how tedious this can get over time. I want to be able to add a new update to my website simply by updating data, not having to edit code and rebuild my website. i.e., I want to separate the data from the website. Which means I need to use server rendered pages. Gatsby is used for creating static websites with infrequent changes. Gatsby builds webpages at build time. Thus, the choice was obvious, I need to jump to Next.js.

There are plenty of blogs online about the similarities and differences between the framework, but the primary reason for me was simple; Separation of concerns. Another reason why I am making the jump is because I have since gained experience in using multiple frameworks at my work (Next.js, Vite, etc) and believe a new, refreshed website would be able to showcase my abilities better.

Set up a base Next.js project with all the tools needed for a scalable development practice.


## Project Setup

Create a new next.js project using the following command:

```bash
PROJECT_NAME=<Your project name>
npx create-next-app --ts $PROJECT_NAME
cd $PROJECT_NAME/
```

Now we begin setting up all our tools before we start development

## Engine Locking

We want to ensure all tools (such as CI integrations) use the same version of Node.js. This practice is best to avoid inconsistencies between environments and can prevent bugs from being shipped to production.

We'll create two files for this:

`.nvmrc`

```text
20.7.0
```

This file tells all the tools to use Node.js Version 20.7.0. I am setting this lock since this is the version I'll be using during development locally

`.npmrc`

```text
engine-strict=true
```

This file tells all tools to use the versions specified in the `package.json` file for node and npm.



I will be using npm and not yarn, and I want to enforce that to all the tools as well. For this, we'll update our `package.json` by adding a section called "engines" as below:

```
{
  "name": "<Project Name>",
  "author": "<Project Author>",
  "description": "<Project Description>",
  ...,
  "engines": {
    "node": ">=20.7.0",
    "npm": ">=10.8.2",
    "yarn": "please-use-npm"
  },
  ...
}
```

This tells all the required tools to use the specified version of node and npm and to not use yarn for managing dependencies.



Since we have made some changes to package.json and added more node-level stuff, let’s ensure our project is working at this stage before proceeding further.

To test if the dev server is running:

```
npm install
npm run dev
```

You should see that a server has started on port 3000. Open up `http://localhost:3000/` in your browser and you should be able to see the Next.js template screen

![](image.png){width=70%}

Let's try running the build command as well to be sure that we'll be able to deploy the site later on

```
npm run build
npm run start
```

If all is good, you should again see the Next.js template screen on `http://localhost:3000/`

If you can't fix any errors before proceeding.

## Git Setup

This is a good point to commit everything to the remote repo. Later we will enforce code quality through linting and pre-commit hooks. For now we are just commiting all changes to the remote repo.

A fresh Next.js project created with `create-next-app` initializes a Git repo in the project directory. I am using GitHub as my remote.  To push changes, run the below commands:

```bash
git remote add origin <your git origin>
git push -u origin main
```



If everything is fine, we'll commit our changes of enabling engine locking

```bash
git add *
git add .*rc
git commit -m "Enabling engine locking"
git push
```

## Code Formatting and Quality Tools

To ensure good quality code is written anytime any update is done, we will enforce code style consistency and best practices using `eslint` and `prettier`. ESLint enforces best practices, and prettier will automatically format our code files. We will also setup triggers in VSCode to automatically run prettier when we save the file.

To be more strict, we are going to use Vercel's Style Guide, which is a strict set of rules for maintaining code quality.

Refer to the following:

* <https://github.com/vercel/style-guide>
* <https://mwskwong.com/blog/enforcing-coding-style-with-vercel-style-guide>

ESLint comes pre-installed with `create-next-app` projects. However, we want to use the `.eslintrc.js` format and not `.eslintrc.json`, since we want to use Vercel's style guide.

First, let's install the required packages:

```bash
npm install --save-dev @vercel/style-guide eslint@8 eslint-config-next@14 typescript@5
```

Now, let's set up the ESLint configuration. You can go ahead and delete the `.eslintrc.json` file. We will not be using it. Create a new file `.eslintrc.js`, and paste the below code in it:

```javascript
const { resolve } = require('node:path');

const { JAVASCRIPT_FILES } = require('@vercel/style-guide/eslint/constants');

const project = resolve(__dirname, 'tsconfig.json');

/** @type {import('eslint').Linter.Config} */
module.exports = {
  root: true,
  extends: [
    require.resolve('@vercel/style-guide/eslint/browser'),
    require.resolve('@vercel/style-guide/eslint/node'),
    require.resolve('@vercel/style-guide/eslint/react'),
    require.resolve('@vercel/style-guide/eslint/next'),
    require.resolve('@vercel/style-guide/eslint/typescript'),
  ],
  parserOptions: {
    project,
  },
  settings: {
    'import/resolver': {
      typescript: {
        project,
      },
    },
  },
  rules: {
    '@typescript-eslint/explicit-function-return-type': [
      'error',
      {
        allowExpressions: true,
        allowHigherOrderFunctions: true,
      },
    ],
    '@typescript-eslint/no-confusing-void-expression': 'error',
    '@typescript-eslint/no-shadow': 'error',
    '@typescript-eslint/no-misused-promises': 'error',
    '@typescript-eslint/no-floating-promises': 'error',
    '@typescript-eslint/restrict-template-expressions': [
      'error',
      {
        allowAny: false,
        allowBoolean: false,
        allowNullish: false,
        allowRegExp: false,
        allowNever: false,
      },
    ],
    'react/function-component-definition': [
      'warn',
      {
        namedComponents: 'arrow-function',
        unnamedComponents: 'arrow-function',
      },
    ],
    'react/jsx-sort-props': [
      'warn',
      {
        callbacksLast: true,
        shorthandFirst: true,
        reservedFirst: true,
        multiline: 'last',
      },
    ],
    'import/order': [
      'warn',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index',
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true,
        },
      },
    ],
    'sort-imports': [
      'warn',
      {
        ignoreDeclarationSort: true,
      },
    ],
  },
  overrides: [
    /**
     * JS files are using @babel/eslint-parser, so typed linting doesn't work there.
     * @see {@link https://github.com/vercel/style-guide/blob/canary/eslint/_base.js}
     * @see {@link https://typescript-eslint.io/linting/typed-linting#how-can-i-disable-type-aware-linting-for-a-subset-of-files}
     */
    {
      files: JAVASCRIPT_FILES,
      extends: ['plugin:@typescript-eslint/disable-type-checked'],
      rules: {
        '@typescript-eslint/explicit-function-return-type': 'off',
      },
    },
    // Next.js App Router and Prettier will have default export
    {
      files: [
        '*.config.{mjs,ts}',
        'src/app/**/{page,layout,not-found}.tsx',
        'src/app/**/{sitemap,robots}.ts',
      ],
      rules: {
        'import/no-default-export': 'off',
        'import/prefer-default-export': ['error', { target: 'any' }],
      },
    },
    // Module declaration files will have default export
    {
      files: ['**/*.d.ts'],
      rules: { 'import/no-default-export': 'off' },
    },
    // Storybook files need separate rules
    {
      files: ['**/*.stories.@(ts|tsx|js|jsx|mjs|cjs)'],
      rules: {
        'import/no-default-export': 'off',
        'unicorn/filename-case': 'off',
      },
    },
  ],
};
```

We have added a nice set of rules to enforce code quality across our files. You can configure the rules as per your preference, but this should give you a nice starting point. Let's try running it on the default starter that `create-next-app` gives us

```
npm run lint
```

At the time of writing this, I receive the following errors; you might receive something different depending on updates done by the Next.js team:

```
./src/app/layout.tsx
21:16  Error: Missing return type on function.  @typescript-eslint/explicit-function-return-type

./src/app/page.tsx
3:16  Error: Missing return type on function.  @typescript-eslint/explicit-function-return-type
10:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
12:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
13:11  Warning: Shorthand props must be listed before all other props  react/jsx-sort-props
31:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
36:15  Warning: Props should be sorted alphabetically  react/jsx-sort-props
38:15  Warning: Props should be sorted alphabetically  react/jsx-sort-props
46:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
57:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
62:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
64:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
72:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
77:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
79:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
87:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
92:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
94:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
```

Perfect. we know our linting rules are working now. If you open the files in VSCode, you will see that it also highlights the code where the ESLint checks are failing.



Next, let's set up Prettier. Install it from npm as follows:

```bash
npm install --save-dev prettier
```

Create a file in your root directory to specify the configuration at `prettier.config.mjs` and add the following code to it:

```javascript
import vercelPrettierOptions from '@vercel/style-guide/prettier';

/** @type {import('prettier').Config} */
const config = {
  ...vercelPrettierOptions
};

export default config;
```

This specifies that we are using the settings specified by Vercel's style guide. We can also override any of their settings inside config.

Let's also add a script to `package.json` to run prettier.

```
{
  ...,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "prettier": "prettier --write ."
  },
  ...
}
```

Save the file and run: `npm run prettier` . You will see the files get updated in real-time. In my case it changed about six files. This was my output at this point:

```
.eslintrc.js 56ms
next.config.mjs 3ms (unchanged)
package-lock.json 123ms (unchanged)
package.json 53ms
postcss.config.mjs 3ms (unchanged)
prettier.config.mjs 2ms
README.md 29ms (unchanged)
src/app/globals.css 28ms (unchanged)
src/app/layout.tsx 53ms
src/app/page.tsx 36ms
tailwind.config.ts 7ms
tsconfig.json 6ms (unchanged)
```

We can also tell VSCode to run prettier when we save a file. This way, we won't see sudden surprises in bulk. Install the `Prettier - Code Formatter` extension for VSCode.Press `Ctrl+Shift+P` to open the shortcut search bar and search for `Preferences: Open Workspace Settings`. Here, search for `Format On Save`  and turn on the checkbox next to `Editor: Format On Save` . Now search for `Default Formatter` and select `Prettier - Code Formatter` .

Now we can see this in action. Come to the `eslintrc.js` file and change any of the single quotes to double quotes. Finally, press `Ctrl+S` . You can see the double quotes changing back into single quotes in front of you.

Phew, that's a lot of changes. Now would be a good time to commit these changes to your git remote.

## Git Hooks

Git hooks are a wonderful tool that allows us to run scripts at any stage of the git process, such as add, commit, push, etc. We want to be able to set conditions, such as only allowing git commit and push to succeed if our code meets the lint checks. We are going to install [Husky](https://typicode.github.io/husky/), a tool that allows us to do just this.

Install Husky using the below command:

```bash
npm install --save-dev husky
npx husky init
```

Now, you should have a folder called `.husky` created at your project root. This is where we'll store all our Git Hooks. Note that this also added a line in your `package.json` scripts called `"prepare": "husky"` .  This script will automatically run when anyone does `npm install` in your project, so the git hooks will get added to the project.

Any script that we wish to run before the `git commit` statement can now be added in the `.husky/pre-commit` file. Let's add a script to run ESLint before commit and only allow the commit to pass if the code passes all lint checks.

### lint-staged

Most often, we don't want to run lint checks on the entire project. As the code grows, this will become a very slow process. Instead, it might be better to run lint checks only on the files staged for commit. We can use [lint-staged](https://github.com/lint-staged/lint-staged) for this.

Install it using the below command:

```bash
npm install --save-dev lint-staged
```

Create a file called `.lintstagedrc.js` in your root directory and add the below code to it:

```javascript
const path = require('node:path');

const buildEslintCommand = (filenames) =>
  `next lint --file ${filenames.map((f) => path.relative(process.cwd(), f)).join(' --file ')}`;

/** @type {import('lint-staged').Config} */
module.exports = {
  '**/*.(js|jsx|ts|tsx)': buildEslintCommand,
};
```

This specifies that we want to run eslint only on the TypeScript and JavaScript files staged for changes (i.e., added via git add).

Go ahead and make some changes to `src/app/layout.tsx` and `src/app/page.tsx`, stage the changes, and then run

```
npx lint-staged
```

Remember, we haven't fixed the errors from before, so we'd expect the command to fail. And that is exactly what happens. This is the output I receive:

```bash
✔ Preparing lint-staged...
⚠ Running tasks for staged files...
  ❯ .lintstagedrc.js — 4 files
    ❯ *.(js|jsx|ts|tsx) — 2 files
      ✖ next lint --file src/app/layout.tsx --file src/app/page.tsx [FAILED]
↓ Skipped because of errors from tasks.
✔ Reverting to original state because of errors...
✔ Cleaning up temporary files...

✖ next lint --file src/app/layout.tsx --file src/app/page.tsx:

./src/app/layout.tsx
21:16  Error: Missing return type on function.  @typescript-eslint/explicit-function-return-type

./src/app/page.tsx
3:16  Error: Missing return type on function.  @typescript-eslint/explicit-function-return-type
11:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
13:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
14:11  Warning: Shorthand props must be listed before all other props  react/jsx-sort-props
32:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
37:15  Warning: Props should be sorted alphabetically  react/jsx-sort-props
39:15  Warning: Props should be sorted alphabetically  react/jsx-sort-props
47:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
58:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
63:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
65:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
73:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
78:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
80:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
88:11  Warning: Props should be sorted alphabetically  react/jsx-sort-props
93:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
95:13  Warning: Props should be sorted alphabetically  react/jsx-sort-props
```

Now we can use this in our pre-commit hook. Modify the `/.husky/pre-commit` file as follows:

```bash
npx lint-staged
```

Save it and try running a git commit

```bash
git commit -m "testing commit with bad linting"
```

You'll observe that it fails, as our staged files fail ESLint checks.

### Conventional Commits

Let's also add a pre-commit hook to ensure that our commit message follows conventions as specified by [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). This provides us a standard to ensure our commit messages are both human and machine readable, and allows us to autogenerate changelogs in the future.

Install commitlint from npm via the below command:

```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

In your root directory, add a file called `.commitlintrc.js`, and paste the below code in it:

```javascript
// build: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
// ci: Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
// docs: Documentation only changes
// feat: A new feature
// fix: A bug fix
// perf: A code change that improves performance
// refactor: A code change that neither fixes a bug nor adds a feature
// style: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
// test: Adding missing tests or correcting existing tests

/** @type {import('@commitlint/types').UserConfig} */
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'body-leading-blank': [1, 'always'],
    'body-max-line-length': [2, 'always', 100],
    'footer-leading-blank': [1, 'always'],
    'footer-max-line-length': [2, 'always', 100],
    'header-max-length': [2, 'always', 100],
    'scope-case': [2, 'always', 'lower-case'],
    'subject-case': [
      2,
      'never',
      ['sentence-case', 'start-case', 'pascal-case', 'upper-case'],
    ],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'type-enum': [
      2,
      'always',
      [
        'build',
        'chore',
        'ci',
        'docs',
        'feat',
        'fix',
        'perf',
        'refactor',
        'revert',
        'style',
        'test',
        'translation',
        'security',
        'changeset',
      ],
    ],
  },
};
```

Also, create a file `./husky/commit-msg`, and paste the below code in it:

```bash
npx --no-install commitlint --edit $1
```

Now, you have enforced Conventional Commits also. Refer to the hyperlinked url to understand the rules of commit messages.

Let's test if this is working. First, fix the ESLint errors, otherwise the pre-commit hook will fail and we wont be able to test the commit message. After that, if we run:

```bash
git commit -m "testing commit with bad message"
```

You will get the following output:

```bash
✔ Preparing lint-staged...
✔ Running tasks for staged files...
✔ Applying modifications from tasks...
✔ Cleaning up temporary files...
⧗   input: testing commit with bad message
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky - commit-msg script failed (code 1)
```

Let's provide a valid commit message

```bash
git commit -m "build: implement git hooks for commit messages"
```

We see a successful commit:

```bash
✔ Preparing lint-staged...
✔ Running tasks for staged files...
✔ Applying modifications from tasks...
✔ Cleaning up temporary files...
[main 9bf8cdd] build: implement git hooks for commit messages
 4 files changed, 1724 insertions(+), 40 deletions(-)
```

## VSCode Configuration

Make sure your`.vscode/settings.json` looks like this:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.organizeImports": "always"
  }
}
```

This will ensure that Prettier runs on save and your top level imports are sorted

### Debugging Configurations

Create a file `.vscode/launch.json` and fill it as follows:

```json
{
  "version": "0.1.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev"
    },
    {
      "name": "Next.js: debug client-side",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000"
    },
    {
      "name": "Next.js: debug full stack",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "serverReadyAction": {
        "pattern": "started server on .+, url: (https?://.+)",
        "uriFormat": "%s",
        "action": "debugWithChrome"
      }
    }
  ]
}
```

This will allow you to test your application from the Debug option on the left sidebar on VSCode


## Conclusion

I am still in the process of migrating my website. Plenty of other blogs also integrate Storybook into their scalable architecture. I am still in the process of understanding how beneficial it will be for my usecase, and hence I have omitted its setup from the blog currently. I might come back in the future to add its configuration once I've understood it myself. Until then, this should provide you a nice starting point for any Next.js application you might develop. Again, feel free to play around with any of the configuration files we wrote to find out which rulesets work the best for you. 

There are plenty of things you can do with your git repository at this point. Maybe you can make a template out of this setup which you can clone for any project later. Maybe you can also add a workflow to ensure the dependencies in the template repository stay updated? I am unsure of how well the workflow might work since these frameworks can have updates that can cause breaking changes at any point. 

If you have any recommendations about the setup or any enhancements, please feel free to leave them in the comments or [reach out to me directly](https://shivanshbakshi.dev/contact/)!