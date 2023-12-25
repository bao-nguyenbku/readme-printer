## Code Convention

>**Notice**: <br /> - The goal of convention is to ensure that all members in the team follow a consistent process. Typically, each individual may have their own preferred naming and coding style. What's crucial is that not all opinions are clearly right or wrong, which is why we need a convention to keep everything aligned and consistent. This will help everyone understand, search, and collaborate on each other's work more quickly and easily. <br /> - However, the convention is meant to improve performance rather than restrict it, so everyone's contributions will be welcomed. Flexibility in its usage is also an important factor in achieving good results.
## Table of contents
- [Code Convention](#code-convention)
- [Folder structure](#folder-structure)
- [Name conventions](#name-conventions)
- [Common rules](#common-rules)
- [Naming React components & props](#naming-react-components-props)
- [Styled components](#styled-components)
- [API Integration](#api-integration)
- [Internalization (i18n)](#internalization-i18n)
- [Functions](#functions)
- [Variables](#variables)
- [Control statements](#control-statements)
- [Switch statements](#switch-statements)
- [Methods](#methods)
- [Operators](#operators)
- [Strings](#strings)
- [Objects](#objects)
- [Conclusion](#conclusion)

## Folder structure
```
public/
src/
├── apis/                 // Contain async function to call api
├── assets/               // Contains static files like images, audio, etc
├── components/           // Share components and common component
├── constants/            // Share components and common component
│   ├── time.constant.ts  // Defined constant variables
├── hooks/                // Custom hooks
│   ├── useMediaQuery.ts
├── i18n/
│   ├── locales/          // Hold languages translation files
├── layouts/
│   ├── components/
├── pages/
│   ├── authentication/   // Folder for the Auth feature
│   │   ├── components/   // Components related to the Auth
│   │   └── ...
│   └── ...               // Additional feature folders
├── providers/
│   └── toast.provider.ts   
├── services/             // Async function that call api 
│   │                        and transform data using dto
│   └── auth/
│       ├── auth.service.ts  
│       ├── auth.dto.ts  
│       ├── auth.request.ts  
│       └── auth.response.ts  
├── helpers/              // Contain some helpers like: format date, format 
│   │                        currency, number,...
│   ├── time.helper.ts      
│   └── ...
├── styles/               // Global setting and custom styles
│   ├── _scrollbar.scsss
│   ├── ...
├── index.scss
├── app.jsx              
└── main.jsx
.eslintrs.cjs
.prettierrc
index.html
package.json
...
```
### Name conventions

- **Camel case: `textColor`, `backgroundImage`,…**
- **Kebab case: `kebab-case`, `customer-service` ,…**
- **Upper case: `UPPER_CASE`, `DEFAULT_PAGE_SIZE` ,…**
- **Pascal case: `PascalCase` ,…**

### Common code formation
Follow rules in `.prettierrc` file. If you use [vscode](https://code.visualstudio.com/), please install [Prettier Extension](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) to format code consistently. If you use other editors, you should follow their documentation.

### Naming React components & props
Here is example of how a typicular react component looks:
```jsx
export type ClaimPageProps = {
  work: WorkDTO;
  data: WR_TableData[];
  paging: PagingState;
  loading?: boolean;
  onPageChange: (paging: PagingREQ) => void;
}; 

export default function ClaimPage({
  work,
  data,
  paging,
  loading = false,
  onPageChange,
}: WR_TableProps) {
  const navigate = useNavigate();
  const { t: tCommon } = useTranslation('common');
  const { t: tClaim } = useTranslation('claim');
  const [loadReceipt, loadReceiptState] = useLoadReceiptMutation();

  /* State */
  const { editForm } = useSelector((state: RootState) => state.receipt) as ReceiptState;

  /* Ref */
  const Table = MyTable<HtmlElementInput>;

  /* Handler */
  const loadHandler = async (receiptId: number) => {
    ...
  };
  
  /* Effect */
  useEffect(() => {}, []);
  
  return (...);
}
```
1. Component naming  
It is required `PascalCase` to name a React component. Eg: `ClaimPage.tsx`

2. Props naming  
Use component name and postfix with _Props, → ClaimPageProps
If props properties are small, you can inline it. Otherwise, split it to single type and naming as above
Export or not export is your choice  
3. I18n usages  
I18n have many modules (each feature has it own module, create by us). Eg: common, claim, authentication, landing, etc. Use postfix with its module
```jsx
const { t: tCommon } = useTranslation('common')
```
instead of 
```jsx
const { t } = useTranslation('common') 
```
even if only one translation is used.

4. Handler  
For logic handler, postfix with _handler. Mostly, it use verb for naming, eg: fetchClaimsHandler, updateClaimHandler

5. Component callback  
Prefix with on_ , eg: onLoad, onClaimUpdate, onClaimDelete. This naming will separate component callback function with other functions

6. Order in component  
  a. Most of time, our component will includes dependencies, state, handler, effect  
  b. Personally, I always separate the zone by comment and place with following order on every of my component dependencies -> ref -> state -> handler / effect / etc
### Styled components
We use tailwindCSS and scss to style components.
If tailwind classes are too long as following or we need to use dynamic classnames:
```jsx
<div className='lg:w-[468px] w-auto 450:w-[45vw] object-contain h-[146px] 450:h-auto lg:h-[402px] z-10 absolute bottom-full left-0 sm:left-10'></div> 
```
Use `cn` function from helpers. It is a function which using [clsx](https://www.npmjs.com/package/clsx) and [tailwind-merge](https://www.npmjs.com/package/tailwind-merge) to manage classes with various feature. Checkout their docs for more detail.
Here is result of using `cn`:
```jsx
<div className={cn(
  'lg:w-[468px] w-auto 450:w-[45vw]', 
  'object-contain z-10 absolute',
  'h-[146px] lg:h-[402px] 450:h-auto',
  'bottom-full left-0 sm:left-10',
  isFlipX && 'scale-x-[-1] -ml-0 w-[208px]',
  frameBg.includes('bg-gray-1') && 'text-primary-text',
)}></div> 
```
It is highly recommended to use `cn` function for dynamic classes instead of template literal string.
```jsx
// Do✅
<div 
  className={cn(
    'w-full',
    isShow && 'h-full',
  )}
></div>
// Don't❌
<div className={`w-full ${isShow ? 'h-full': 'h-0'}`}></div>
```
### API Integration
**Transformation**  
We have 3 types includes dto, request, response. Three of them are just type, but each have its own purpose. We can find them inside `services` folder:
```jsx
  auth/
   ├── auth.service.ts  // Contain functions that transform data from 
   │                       API response (DTO transformation)
   ├── auth.dto.ts      // Contain dto types to transform from API response
   ├── auth.request.ts  // Define request types for calling API
   └── auth.response.ts // Define response types for receiving data from API
```
- Request: What you send in request body will be defined here
- Response: What type and structure we receive from API
- DTO: Is what we really need and store to use in your application
  - Never use ResponseType as DTOType, sometime it might look the same but always have to separate
  - DTO is result after mapping step by `auth.service.ts` 
    - Enum is validate and parse
    - Default value is set if null happen here
    - And more are done in side parsing step

Example of get transformed data from response:
```jsx
// article.service.ts
export const getArticlesDTO = (article: ArticleRES) => {
  const convertedArticle: ArticleDTO = {
    id: article.id,
    title: article.title,
    createdAt: article.createdAt,
    content: article.content,
    attachments: article.attachments,
    modifiedAt: article.modifiedAt,
    thumbnailImage: article.thumbnailImage,
  };
  return convertedArticle;
};

// article.response.ts
export type ArticleRES = {
  id: number;
  title: string;
  content: string;
  thumbnailImage: string[] | string;
  attachments: {
    fileName: string;
    fileKey: string;
  }[];
  createdAt: string;
  modifiedAt: string;
};

export type ArticleDetailRES<T> = {
  currentArticle: T;
  beforeArticle: T | null;
  afterArticle: T | null;
};

// article.request.ts
export type ListArticleParams = {
  page: number;
  keyword: string;
};

// article.dto.ts
import { ArticleRES } from './article.response';

export type ArticleDTO = ArticleRES;
```
**React Query - data fetching**  
We use `React Query` to get & send data through api endpoints. Read more about this library at [React Query docs](https://tanstack.com/query/v3/docs/react/overview).
### Internalization (i18n)
> This should be define before we handle UI

Workflow:  
- Check figma if any place need i18n  
- For common text share between buttons, labels,... define in common.json file
- For enum → define in enum json also
- Other will be separated by feature/page 
- Each will have its own json file
### Functions

For function names, use **camelCase** starting with a lowercase character. Use concise, human-readable, and semantic names where appropriate. Functions should have one responsibility.

When using anonymous functions as a callback (a function passed to another method invocation), if we do not need to access `this`, use an arrow function to make the code shorter and cleaner.

```js
const numbers = [1, 2, 3, 4];
// Do✅
const sum = numbers.reduce((a, b) => a + b);

// Don't❌
const sum = numbers.reduce(function(a, b) => {
	return a + b;
});
```

When using arrow functions, use [implicit return](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#function_body) (also known as *expression body*) when possible:

```js
// Do✅
arr.map((e) => e.id);

// Don't❌
arr.map((e) => {
  return e.id;
});
```

### Variables

**Variable names**

- Use short identifiers, and avoid non-common abbreviations. Good variable names are usually between 3 to 10-character long, but as a hint only. For example, `accelerometer` is more descriptive than abbreviating to `acclmtr`.
- Use real-world relevant examples where each variable has clear semantics.
- Do not use the [Hungarian notation](https://en.wikipedia.org/wiki/Hungarian_notation) naming convention. Do not prefix the variable name with its type.
```js
// Do✅
bought = car.buyer !== null
name = "John"

// Don't❌
bBought = oCar.sBuyer != null
sName = "John"
```
- For primitive values, use **_camelCase_**, starting with a lowercase character. Do not use `_`. Use concise, human-readable, and semantic names where appropriate. For example, use `currencyName` rather than `currency_name`.
- Avoid using articles and possessives. For example, use `car` instead of `myCar` or `aCar`. There may be exceptions, like when describing a feature in general without a practical context.

Use variable names as shown below:

```jsx
// Do✅
const playerScore = 0;
const speed = distance / time;

// Don't❌
const thisIsaveryLONGVariableThatRecordsPlayerscore345654 = 0;
const s = d / t;
```

**Variable declarations**

When declaring variables and constants, use the `let` and `const` keywords, not `var`.

If a variable will not be reassigned, prefer `const`, like so:

```jsx
const name = "Shilpa";
console.log(name);
```

If we will change the value in the future, use `let` as below:

```jsx
let age = 40;
age++;
console.log("Happy birthday!");
```

Declare one variable per line:

```jsx
// Do✅
let var1;
let var2;
let var3 = "Apapou";
let var4 = var3;

// Don't❌
let var1, var2;
let var3 = (var4 = "Apapou");
```

### Control statements

There is one notable case to keep in mind for the `if...else` control statements. If the `if` statement ends with a `return`, do not add an `else` statement.

```jsx
// Do✅
if (test) {
  // Perform something if test is true
  return;
}

// Perform something if test is false

// Don't❌
if (test) {
  // Perform something if test is true
  return;
} else {
  // Perform something if test is false
}
```

_Use braces {} with control flow statements and loops_ even if there is only one statement inside the block
```js
// Do✅
if (test) {
  return 'Something';
}

// Don't❌
if (test) return 'Something'
```
### Switch statements

Don't add a `break` statement after a `return` statement in a specific case. Instead, write `return` statements like this:

```jsx
switch (species) {
  case "chicken":
    return farm.shed;
  case "horse":
    return corral.entry;
  default:
    return "";
}
```

### Methods

To define methods, use the method definition syntax:

```jsx
// Do✅
const obj = {
  getName() {
    // …
  },
  getAge() {
    // …
  },
};
```

Instead of:

```jsx
// Don't❌
const obj = {
  getName: function () {
    // …
  },
  getAge: function () {
    // …
  },
};
```

When possible, use the shorthand avoiding the duplication of the property identifier. Write:

```jsx
function createObject(name, age) {
  return { name, age };
}
```

Don’t write:

```jsx
function createObject(name, age) {
  return { name: name, age: age };
}
```

### Operators

**Conditional operators**

When you want to store to a variable a literal value depending on a condition, use a [conditional (ternary) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator) instead of an `if...else` statement. This rule also applies when returning a value. Write:

```jsx
// Do✅
const x = condition ? 1 : 2;

// Don't❌
let x;
if (condition) {
  x = 1;
} else {
  x = 2;
}
```

**Strict equality operator**

Prefer the [strict equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Strict_equality) (triple equals) and inequality operators over the loose equality (double equals) and inequality operators.

Use the strict equality and inequality operators like this:

```jsx
// Do✅
name === "Shilpa";
age !== 25;

// Don't❌
name == "alpha";
age != 25;
```

**Shortcuts for boolean tests**

Prefer shortcuts for boolean tests. For example, use `if (x)` and `if (!x)`, not `if (x === true)` and `if (x === false)`, unless different kinds of truthy or falsy values are handled differently.

### Strings

**Template literials**

For inserting values into strings. Here is an example of the recommended way to use template literals. Their use prevents a lot of spacing errors.
```js
// Do✅
const name = "Shilpa";
console.log(`Hi! I'm ${name}!`);

// Don't❌
const name = "Shilpa";
console.log("Hi! I'm" + name + "!"); // Hi! I'mShilpa!
```
### Objects
Every property should be written each line and finished with a comma.
```js
// Do✅
const person = {
  firstName: "John",
  lastName: "Doe",
  age: 50,
  eyeColor: "blue"
};

// Don't❌
const person = { firstName: "John", lastName: "Doe", age: 50, eyeColor: "blue" };
```

## Conclusion
If you have valuable suggestions, don't hesitate to raise them for everyone to apply. We should avoid each person choosing a different approach. All opinions will be heard, but we need to reach a consensus before applying them.
