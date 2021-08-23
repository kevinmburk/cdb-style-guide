# Coaching Dashboard Frontend Style Guide

### React Component File/Folder Naming

- Folders are named in PascalCase.
- Each folder contains an `index.tsx` file with an export function of the same name as the folder, which will be auto imported in other files when pointing to the parent folder.

  ```tsx
  // components/TacoMenu/index.tsx
  export default function TacoMenuItem({ menuItem, id }: Props) {
  ```

  ```tsx
  // Another component
  import TacoMenuItem from '../TacoMenuItem';
  ```

- If not an `index.tsx` file, files should be named PascalCase.
- If the file has no JSX element export, the file should be in camelCase and have a `.ts` file type, except...
- If the file is a Typescript models file, the file should be named in PascalCase with `Resource` in the file name i.e. `TacoMenuResource.ts`

### Component Folder Organization

- **Only one JSX component export per file.**
- Multiple component files can be stored in the same folder, provided they are only used in the master `index.tsx` file. If used in other components, the file should be moved up the file tree to the base of where they're shared amongst the children components.
- If a component is getting big and filled with a lot of business logic functions, they should be moved to a `utils` file. Name the file after the parent folder in camelCase, i.e. `tacoMenuItemUtils.ts`

### Component File Organization

- File imports should be organized at the top of the file in the following order:
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

- If passing a `setState` function as props, simplify the type in the interface:

  ```typescript
  interface Props {
    // Bad
    setSelectedTaco: Dispatch<SetStateAction<TacoMenuItem>>;
    // Good
    setSelectedTaco: (taco: TacoMenuItem) => void;
  }
  ```

- Components should generally be between 100-200 lines of code. If more than 200 lines, the file should likely be split up into smaller components. If less than 100 lines, it may not need its own component, but it makes sense to keep the smaller component if used in multiple places. Again, business logic can be separated out into a `utils` file.
- The `Props` interface should be the only one stated in the component file. If `useState` variables are using primitives, those can be stated inline in the genetic type argument. Otherwise, types/interfaces should be stored in a resource file in the `models` folder.

  ```typescript
  const [tacoQuantity, setTacoQuantity] = useState<number>(0);
  // TacoTypeResource is imported from a separate resource file
  const [tacoType, setTacoType] = useState<TacoTypeResource>(null);
  ```

### Code Conventions

- Prettier will take care of most code formatting for us. Just make sure to run a build of the app before opening a PR.
- Always use self closing tags in JSX if there are no children inside the element.

  ```tsx
  // Bad
  <i className="icon icon-distance"></i>
  ```

  ```tsx
  // Good
  <i className="icon icon-distance" />
  ```

- As much as possible, organize components in the following manner:

  1. Any props manipulation variables (object destructuring of props, sorting an array of objects, etc.)
  1. API call variables (`react-query` hooks)
  1. State variables
  1. `useEffect()` call
  1. Functions used in the component
  1. JSX return

  ```tsx
  export default function TacoMenuPage({ menu }: { menu: TacoMenuItem[] }) {
    const sortedMenu = menu.sort((a, b) => a.name - b.name);
    const { id: userId } = getCurrentUserId();
    const [order, setOrder] = useState<TacoOrder>(null);

    useEffect(() => {
      // Function and logic here
    }, [menu]);

    const sendOrder = () => {
      postTacoOrder(order);
    };

    return (
      <>
        <h1>Taco Menu</h1>
        {sortedMenu.map((menuItem: TacoMenuItem) => (
          <TacoMenuItem key={menuItem.id} id={userId} menuItem={menuItem} />
        ))}
        <button onClick={sendOrder}>Send order</button>
      </>
    );
  }
  ```

- As much as possible, use a unique data point as a key for mapped JSX elements, instead of `index`, which can be an antipattern.

  ```tsx
  // Bad
  <select>
    {tacoToppings.map((topping, i) => (
      <option key={i}>{topping.name}</option>
    ))}
  </select>
  ```

  ```tsx
  // Good, if filter.name is unique
  <select>
    {tacoToppings.map((topping) => (
      <option key={topping.name}>{topping.name}</option>
    ))}
  </select>
  ```

- If functions get too large and hard to read inside JSX returns (i.e. more than two or three lines), move them into the body of the function and use a callback in the JSX.

  ```tsx
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
  // Parent component
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

    export default function Taco({name, hasSalsa, hasSourCream}) {
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

- Use a `Fragment` instead of a `div` if the html attributes of a `div` aren't needed:

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

- Try to avoid IIFE switch statements inside JSX. If you're in a place where you're thinking of one, it probably means another component with a switch statement is a good idea. Or, you could use an object literal.

  ```tsx
  // Bad

  (() => {
    switch (tacoType) {
      case 'chicken':
        return <ChickenTaco />;
      case 'carnitas':
        return <CarnitasTaco />;
      case 'blackBean':
        return <BlackBeanTaco />;
    }
  })();
  ```

  ```tsx
  // Good
  // Parent component
  return (
    <>
      <TacoOptions tacoType={tacoType} />
    </>
  );

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
  const tacoTypeComponents = {
    chicken: <ChickenTaco />,
    carnitas: <CarnitasTaco />,
    blackBean: <BlackBeanTaco />,
  };
  const TacoToDisplay = tacoTypeComponents[tacoType];

  return (
    <>
      <TacoToDisplay />
    </>
  );
  ```
