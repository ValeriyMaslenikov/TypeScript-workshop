## First of all – installation of the TypeScript

Let's init the repository `npm init`

Run `npm install --save-dev typescript @types/node`

`@types/node` are needed to support default NodeJS variables, modules, etc. Like `console.log`, `process`, like module `fs`.

## Configuration

For now we're going to use my version of `tsconfig.json`, it's a configuration for `TypeScript` compiler.

```JSON
// https://www.typescriptlang.org/tsconfig
{
  "compilerOptions": {
    "strict": true,
    "outDir": "dist",
    "lib": [
      "es2019",
      "es2020.bigint",
      "es2020.string",
      "es2020.symbol.wellknown"
    ],
    "target": "ES2019",
    "module": "commonjs",
    // Stop compiling on erros (dist will not be upgraded)
    "noEmitOnError": true
  }
}
```

## Let's install npx
```
npm install -g npx
```


Check that it's working
```
npx tsc --version 
```

It should show you your version of installed typescript.

## Proceed with compilation

Let's introduce a new variables with different types

```typescript
let isBoolean: boolean = false;
```

Compile this file via


```bash
npx tsc
```
If you have an error because of `npx`, try to use local version of `tsc` (tsc === typescript compilator):
```bash
./node_modules/.bin/tsc
```
And take a look on the results in the `./dist/types.js`

## Overwrite the value of the variable with value of incorrect type

```typescript
// Trying to set non-boolean type
isBoolean = "hello world!";
```

Compile the file. You should see an error:
```
types.ts:3:1 - error TS2322: Type 'string' is not assignable to type 'boolean'.

3 isBoolean = 'hello world!';
  ~~~~~~~~~


Found 1 error.
```

What's the problem?

Types are linked to the variable in the place of declaration, after you assigned the type for the variable you can't change it.

## What if I don't use types, it mean I will be able to change them?

Yes and no. 

If you create a variable without the type and without default value, it will have a type `any`
Remember, FORGET ABOUT USAGE OF ANY. It's not an option. Usage of `any`  is the same as if you don't use TypeScript at all, cause TypeScript doesn't know what kind of type you're going to store in that variable.

And no, because there's a [Type inference](https://www.typescriptlang.org/docs/handbook/type-inference.html):
![](images/var-type-in-ide.png)

The type of the `isBoolean` variable is inferred to be a number. This kind of inference takes place when initializing variables and members, setting parameter default values, and determining function return types.


Let's open [a URL and check whether how it works](https://www.typescriptlang.org/docs/handbook/type-inference.html).

## Enums and compilation result

Enums are compiled to the Object, it's as common type as others from the POV.

More about [ENUMS](https://www.typescriptlang.org/docs/handbook/enums.html).

## Let's create an application to display the weather for the next week

We will be using the API openweather, the link to make the request:

https://api.openweathermap.org/data/2.5/onecall?lat=48.462750&lon=35.054659&appid=da19731b23c38d11bafd8de6e177f5a5&units=metric&lang=ru

https://openweathermap.org/api/one-call-api

## Write the code for that application

Let's install an axios via:
```
npm i --save axios
```

https://github.com/axios/axios

And make a get request to the URL above.

We may see that axios returns the object with type `any`. It's not good.

Let's write our own type for the response of the Weather map. As far as we're going to show just the weather for the next 5 days,
we're mostly interested in the fields from the list:

response.daily[i].dt
response.daily[i].temp
response.daily[i].temp.day
response.daily[i].temp.min
response.daily[i].temp.max
response.daily[i].temp.night
response.daily[i].temp.eve

To allow axios know about the type of response, you may hint him, like:
```
const result = await axios.get<OpenWeatherResponse>('https://api.openweathermap.org/data/2.5/onecall?lat=48.462750&lon=35.054659&appid=da19731b23c38d11bafd8de6e177f5a5&units=metric');
```
Where `OpenWeatherResponse` is the name of the type used for the openweathermap response.


## It's not convenient to use `tsc` and `node ./dist/weather-app.ts` every time

Let's used `ts-node`. It's alternative to `node`, but it works with `typescript`. It compiles TypeScript to JS and then executes it via `node` command under the hood.

## Let's proceed with application

Let's iterate over daily object and use code like this to format date:
```
    const dateObject = new Date(daily.dt * MS_IN_SEC);

    const date = dateObject.toISOString().split('T')[0];
```

JavaScript abilities to format date is not very powerful, but JS is popular for its packages created.
Let's install library https://www.npmjs.com/package/dateformat

After install you may see an error,

https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html
https://github.com/DefinitelyTyped/DefinitelyTyped  

## Type Unions and Type Intersections

Let's create several types like: User, UserAdmin, UserModerator

```
type BaseUser = {
  name: string;
  password: string;
};

type UserAdmin = BaseUser & {
  type: "admin";
  adminSince: number;
};

type UserModerator = BaseUser & {
  type: "moderator";
  moderatorSince: number;
};

type User = UserAdmin | UserModerator;
```

Then let's add some functions which generate every type of the users, like:
```

function getAdminUser(): UserAdmin {
  return {
    name: "valerii",
    password: "very-secure-password",
    type: "admin",
    adminSince: 123,
  };
}

function getModeratorUser(): UserModerator {
  return {
    name: "someone-but-valerii",
    password: "very-secure-password+1",
    type: "moderator",
    moderatorSince: 123,
  };
}
```

Let's try to rename `adminSince` to `moderatorSince`. TypeScript will throw an error.
Let's write type guard to check whether the user is admin.

```
const users = [getModeratorUser(), getAdminUser()];

function isAdmin(user: User): user is UserAdmin {
  return user.type === "admin";
}
```
We may create a lot of general methods for every users like:
```
function showMeUserName(user: User) {
  console.log(user.name);
}
```

It's working cause every user  has a name.

TypeScript disallow you to check the type if it is not in the list of available values,like:
```
//  This condition will always return 'false' since the types '"admin" | "moderator"' and '"admi"' have no overlap.
if (user.type === 'super-user') {

  
}
```

But if you create a valid check, typescript cast a type User to specific type, 
even without user-defined type guards, like:
```

if (user.type === "admin") {
  console.log(user.adminSince); // User here is admin
}
```

or you can even check the presence of the property:
```
if ('adminSince' in user) {
  console.log(user.adminSince); // User here is admin
}
```