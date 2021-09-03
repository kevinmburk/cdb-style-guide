# Coaching Dashboard Frontend Style Guide

### React Component File/Folder Naming

- Folders are named in PascalCase.
- Each folder contains an `index.tsx` file with an export function of the same name as the folder, which will be auto imported in other files when pointing to the parent folder.

  ```typescript
  // components/TacoMenuItem/index.tsx
  export default function TacoMenuItem({ menuItem, id }: Props) {
  ```

  ```typescript
  // Another component
  import TacoMenuItem from '../TacoMenuItem';
  ```

- If not an `index.tsx` file, files should be named PascalCase.
- If the file has no JSX element export, the file should be in camelCase and have a `.ts` file type, except...
- If the file is a Typescript models file, the file should be named in PascalCase with `Resource` in the file name. (i.e. `TacoMenuResource.ts`)

### Component Folder Organization

- Multiple component files can be stored in the same folder, provided they are only used in the master `index.tsx` file. If used in other components, the file should be moved up the file tree to the base of where it's shared amongst the children components.
- If a component is getting big and filled with a lot of business logic functions, they should be moved to a `utils` file. Name the file after the parent folder in camelCase. (i.e. `tacoMenuItemUtils.ts`)

### Component File Organization

- **Only one JSX component export per file.**
- File imports should be organized at the top of the file in the following order: (Note: Look into ESLint settings to automatically do this.)
  1. Built-in imports, i.e. `React`, `useState`, and `useEffect`.
  1. External imports, like `react-query`, and `moment`.
  1. Internal custom hooks, utils files, and `.css` files.
  1. Internal Typescript models files.
  1. Internal component files.
- Don't include file types in file imports, except for `.css` files.

  ```typescript
  import React, { useState } from 'react';
  import { useMutation } from 'react-query';
  import { getMenuItems } from './tacoMenuUtils';
  import { useTooltip } from '../../hooks/useTooltip';
  import './styles.css';
  import { TacoMenuItemContent } from '../../models/TacoMenuResource';
  import MenuHeader from '../MenuHeader';
  ```

- Use `export default function` to export the functional component inline with its declaration.
- Always use object destructuring for props in the arguments of the functional component.
- If component has props passed to it, argument types can be inline in the function if short and 1-2 props, otherwise should be stated in a separate `Props` interface directly above the function declaration.

  ```typescript
  export default function StoreDescription({ storeId }: {storeId: number }) {
  ```

  ```typescript
  interface Props {
    id: number;
    menuItem: TacoMenuItem;
    showFullDescription: boolean;
    setShowFullDescription: (set: boolean) => void;
  }

  export default function TacoMenuItem({
    id,
    menuItem,
    showFullDescription,
    setShowFullDescription,
    }: Props) {
  ```

- If passing a `setState` function as props, simplify the type in the interface, no need to use `React` types for this:

  ```typescript
  interface Props {
    // Bad
    setSelectedTaco: Dispatch<SetStateAction<TacoMenuItem>>;
    // Good
    setSelectedTaco: (taco: TacoMenuItem) => void;
  }
  ```

- Components should generally be between 100-200 lines of code. If more than 200 lines, the file should likely be split up into smaller components. If less than 100 lines, it may not need its own component, but it makes sense to keep the smaller component if used in multiple places. Again, business logic can be separated out into a `utils` file.
- The `Props` interface should be the only one stated in the component file. If `useState` variables are using primitives or a small type only scoped to that variable, those can be stated inline in the generic type argument. Otherwise, types/interfaces should be stored in a resource file in the `models` folder.

  ```typescript
  const [tacoQuantity, setTacoQuantity] = useState<number>(0);
  // Stating this type inline is OK, provided the state variable is only used in this component
  const [toppings, setToppings] = useState<'salsa' | 'sourCream'>(null);
  // TacoTypeResource is imported from a separate resource file
  const [tacoType, setTacoType] = useState<TacoTypeResource>(null);
  ```

- As much as possible, organize components in the following manner:

  1. Any hook instances (i.e. `const { push } = useHistory();`)
  1. Any props manipulation variables (i.e. object destructuring of props, sorting an array of objects, etc.)
  1. API call variables (`react-query` hooks)
  1. State variables
  1. `useEffect()` call
  1. Functions used in the component
  1. JSX return

  ```tsx
  interface Props {
    menu: TacoMenuItem[];
    store: StoreResource;
  }

  export default function TacoMenuPage({ menu, store }: Props) {
    const { push } = useHistory();
    const { locationName, city, state } = store;
    const sortedMenu = menu.sort((a, b) => a.name - b.name);
    const { id: userId } = getCurrentUserId();
    const [order, setOrder] = useState<TacoOrder>(null);

    useEffect(() => {
      // Function and logic here
    }, [menu]);

    const sendOrder = () => {
      postTacoOrder(order);
      push('/order/confirm');
    };

    return (
      <>
        <p>
          Delivery from {locationName}, {city}, {state}
        </p>
        <h1>Taco Menu</h1>
        {sortedMenu.map((menuItem: TacoMenuItem) => (
          <TacoMenuItem key={menuItem.id} id={userId} menuItem={menuItem} />
        ))}
        <button onClick={sendOrder}>Send order</button>
      </>
    );
  }
  ```

### JSX Code Conventions

- **Prettier will take care of most code formatting for us. Just make sure to run an** `npm run build` **of the app before opening a PR.**
- Always use self closing tags in JSX if there are no children inside the element.

  ```tsx
  // Bad
  <i className="icon icon-distance"></i>
  ```

  ```tsx
  // Good
  <i className="icon icon-distance" />
  ```

- As much as possible, use a unique data point as a key for mapped JSX elements, instead of the `index`, which can be an [antipattern](https://robinpokorny.medium.com/index-as-a-key-is-an-anti-pattern-e0349aece318).

  ```tsx
  // Bad
  <select>
    {tacoToppings.map((topping, i) => (
      <option key={i}>{topping.name}</option>
    ))}
  </select>
  ```

  ```tsx
  // Good, if topping.name is unique
  <select>
    {tacoToppings.map((topping) => (
      <option key={topping.name}>{topping.name}</option>
    ))}
  </select>
  ```

- If functions get too large and hard to read inside JSX returns (i.e. more than two or three lines), move them into the body of the function and use a callback in the JSX.

  ```tsx
  // Bad
  return (
    <div>
      <input
        type="text"
        value={order.name}
        onChange={(event) => {
          doSomething();
          doSomethingElse();
          doEvenMoreStuff();
          setOrder({ ...order, name: event.target.value });
        }}
      />
    </div>
  );

  // Good
  const handleOrderName = (event: ChangeEvent<HTMLInputElement>) => {
    doSomething();
    doSomethingElse();
    doEvenMoreStuff();
    setOrder({ ...order, name: event.target.value });
  };

  return (
    <div>
      <input type="text" value={order.name} onChange={handleOrderName} />
    </div>
  );
  ```

- If the prop being sent to a component is an optional boolean, just write the name of the prop in the component to give it an implicit true.

  ```tsx
  // Bad
  <Taco name={name} hasSalsa={true} hasSourCream={true} />
  ```

  ```tsx
  // Good
  <Taco name={name} hasSalsa hasSourCream />
  ```

  ```typescript
  // Taco.tsx
  interface Props {
    name: string;
    hasSalsa?: boolean;
    hasSourCream?: boolean;
  }

  export default function Taco({ name, hasSalsa, hasSourCream }) {
  ```

- Use ternary expressions for conditional rendering as much as possible.

  ```tsx
  // Bad
  if (hasTacos) {
    <TacoFeedback feedbackState={feedbackState} />;
  } else {
    <TacoWaitScreen waitTime={waitTime} />;
  }
  ```

  ```tsx
  // Good
  hasTacos ? (
    <TacoFeedback feedbackState={feedbackState} />
  ) : (
    <TacoWaitScreen waitTime={waitTime} />
  );
  ```

- Use a `Fragment` instead of a `div` if the attributes of a `div` aren't needed:

  ```tsx
  // Bad
  <div>
    <TacoHeader />
    <TacoMenuOptions />
  </div>
  ```

  ```tsx
  // Good
  <>
    <TacoHeader />
    <TacoMenuOptions />
  </>
  ```

- Avoid IIFE switch statements inside JSX. If you're in a place where you're thinking of one, it probably means another component with a switch statement is a good idea. Or, you could use an object literal instead.

  ```tsx
  // Bad
  export default function TacoDetails({ tacoType }: { tacoType: string }) {
    return (
      <>
        <TacoDetailsHeader />
        {(() => {
          switch (tacoType) {
            case 'chicken':
              return <ChickenTaco />;
            case 'carnitas':
              return <CarnitasTaco />;
            case 'blackBean':
              return <BlackBeanTaco />;
          }
        })()}
      </>
    );
  }
  ```

  ```tsx
  // Good
  import TacoOptions from './TacoOptions';

  export default function TacoDetails({ tacoType }: { tacoType: string }) {
    return (
      <>
        <TacoDetailsHeader />
        <TacoOptions tacoType={tacoType} />
      </>
    );
  }

  // TacoOptions.tsx
  export default function TacoOptions({ tacoType }: { tacoType: string }) {
    switch (tacoType) {
      case 'chicken':
        return <ChickenTaco />;
      case 'carnitas':
        return <CarnitasTaco />;
      case 'blackBean':
        return <BlackBeanTaco />;
    }
  }
  ```

  ```tsx
  // Also good
  export default function TacoDetails({ tacoType }: { tacoType: string }) {
    const tacoTypeComponents = {
      chicken: ChickenTaco,
      carnitas: CarnitasTaco,
      blackBean: BlackBeanTaco,
    };
    // tacoType needs a type cast here to appease the TypeScript compiler
    // Other types or interfaces can be made to avoid this
    const TacoToDisplay =
      tacoTypeComponents[tacoType as keyof tacoTypeComponents];

    return (
      <>
        <TacoDetailsHeader />
        <TacoToDisplay />
      </>
    );
  }
  ```

### Resource File Code Conventions

- Resource files should not have a default export. Export each model individually to easily tree shake the imports in other files.

```typescript
// Bad
export default class TacoMenuItem {
  id: number;
  name: string;
  protein: string;
  toppings: Toppings;
}

// Good
export class TacoMenuItem {
```

- Export everything, even if just used inside the Resource file at first. Chances are you'll need it somewhere else sometime.
- All models should be named in PascalCase. Try as much as possible to match the models in CDB backend in naming and properties.
- Do not use default values for `enums`. Always specify the value. Properties in enums should be in camelCase.

```typescript
// Bad
export enum BeanOptions {
  blackBeans,
  pintoBeans,
}
// Good
export enum BeanOptions {
  blackBeans = 'blackBeans',
  pintoBeans = 'pintoBeans',
}
// Good
export enum Proteins {
  chicken = 1,
  barbacoa = 2,
  carnitas = 3,
  alPastor = 4,
}
```

- Models should be made using `class`. Use `type` for limiting strings or making computed types from other models. Any types that are limiting numbers should be made using `enum`.

### TypeScript Code Conventions

- Write immutable code as much as possible. Prefer `const` over `let`. Instead of `for of` and `for in` loops, use array methods like `.map()`, `.reduce()`, and `.filter()` and object methods like `Object.entries()`. These methods return new instances and leave the originals intact.

- Using mutable values is OK within the scope of a function, but only if mutating values within that code block and not affecting variables declared in the body of the functional component.

  ```typescript
  // This is fine
  const getTacoIngredients = () => {
    let string = '';
    for (const ingredient of tacoIngredients) {
      string += `, ${ingredient}`;
    }

    return string;
  };
  ```

  ```typescript
  // This is better
  const getTacoIngredients = () => {
    return tacoIngredients.reduce((string, ingredient) => {
      string += `, ${ingredient}`;
      return string;
    }, '');
  };
  ```

- Use spread syntax to create new objects and arrays.

  ```typescript
  // Array example
  const ingredients = ['Tortilla', 'Al Pastor', 'Cilantro', 'Cotija'];
  // Bad
  const addToIngredients = Array.from(ingredients).push('Sour Cream');
  // Good
  const spreadAddToIngredients = [...exampleArray, 'Sour Cream'];

  // Object example
  const tacoOrder = { taco: 'Carnitas', quantity: 3, salsaType: 'spicy' };
  // Bad
  const addToTacoOrder = Object.assign({}, tacoOrder, { guacamole: true });
  // Good
  const spreadAddToTacoOrder = { ...tacoOrder, guacamole: true };
  ```

- Use object shorthand whenever possible. Put the object shorthand properties at the beginning of the object declaration.

  ```typescript
  // Bad
  const TacoOrder = {
    tortilla: 'corn',
    guacamole: false,
    protein: protein,
    salsa: salsa,
  };

  // Good
  const tacoOrder = {
    protein,
    salsa,
    tortilla: 'corn',
    guacamole: false,
  };
  ```

- Use rest syntax to get an object minus properties you don't want. (Don't use `delete`)

  ```typescript
  const toGoTacoOrder = {
    hotSauce: true,
    orderNumber: 1194,
    itemCount: 7,
    waitTime: 12,
    orderName: 'Kevin',
  };

  // Bad
  delete toGoTacoOrder.orderName;
  completeTacoOrder(toGoTacoOrder);

  // Not ideal
  completeTacoOrder({
    hotSauce: toGoTacoOrder.hotSauce,
    orderNumber: toGoTacoOrder.orderNumber,
    itemCount: toGoTacoOrder.itemCount,
    waitTime: toGoTacoOrder.waitTime,
  });

  // Good
  const { orderName, ...tacoOrderPayload } = toGoTacoOrder;
  completeTacoOrder(tacoOrderPayload);
  ```

- Use strict equality checks of `===` and `!==` for better type checking, there really shouldn't be a need to use `==` or `!=` in TypeScript.

- If a function needs more than 2-3 arguments, refactor it to take one object as an argument instead. This way, ordering doesn't matter, type checking the argument is easier, and it's overall more readable.

  ```tsx
  // Bad
  const setTacoOptions = (
    salsa: string,
    guacamole: boolean,
    tortilla: string,
    beans: string,
    protein: string
  ) => {
    // Function logic here
  };

  return (
    <button
      onClick={() => setTacoOptions('spicy', true, 'flour', 'pinto', 'chicken')}
    >
      Submit
    </button>
  );
  ```

  ```tsx
  // Good
  // Resource file
  export interface TacoOptions {
    salsa: string;
    guacamole: boolean;
    tortilla: string;
    beans: string;
    protein: string;
  }

  // Component (TacoOptions is imported)
  const setTacoOptions = (options: TacoOptions) => {
    // Function logic here
  };

  return (
    <button
      onClick={() =>
        setTacoOptions({
          salsa: 'spicy',
          guacamole: true,
          tortilla: 'flour',
          beans: 'pinto',
          protein: 'chicken',
        })
      }
    >
      Submit
    </button>
  );
  ```
