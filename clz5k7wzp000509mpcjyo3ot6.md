---
title: "The React Hell"
datePublished: Sun Jul 28 2024 12:52:00 GMT+0000 (Coordinated Universal Time)
cuid: clz5k7wzp000509mpcjyo3ot6
slug: the-react-hell
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722170824334/c587c98a-ee8f-4a58-8069-7bbd0c278a5d.png
tags: reactjs, typescript, callback-hell, react-context-hell

---

After switching to the backend and system architecture, I’ve always appreciated the elegance and efficiency of clean code. However, upon revisiting the front-end side of my recent projects, I was confronted with a common issue known as "React hell." This scenario arises when multiple react components or context providers nest within each other, leading to deeply nested structures that are hard to read and maintain. This post details my journey of tackling this problem with a custom solution I named `FlatedReact`.

### The Problem: React Hell

React’s context API is incredibly powerful for managing global state and passing props through the component tree without prop drilling. However, as applications grow, it’s easy to end up with a situation where multiple providers are nested inside each other, creating a callback hell of sorts in React. Here's a typical example:

```tsx
<AuthProvider>
  <ThemeProvider>
    <IntercomProvider>
      <EmailVerificationProvider>
        <TooltipProvider>
          {children}
        </TooltipProvider>
      </EmailVerificationProvider>
    </IntercomProvider>
  </ThemeProvider>
</AuthProvider>
```

While this setup works, it quickly becomes cumbersome and difficult to manage, especially as more providers are added.

### The Solution: FlatedReact

To address this issue, I created a utility named FlatedReact. This utility flattens the provider structure, making the code cleaner and easier to read. The core idea is to use a tuple where the first element is the React function component and the second is its props. Here’s the TypeScript type for it:

```tsx
type FlatedItem<T extends FC<any> = FC<any>> = [T] | [T, (Parameters<T>[0] & { children: undefined }) | undefined];
```

I then created a helper function to facilitate creating these tuples:

```tsx
function makeFlatedItem<T extends (...args: any[]) => any>(
  component: T, 
  props: Omit<Parameters<T>[0], 'children'> | undefined = undefined
) {
  return [component, props] as FlatedItem<T>;
}
```

### The Renderer Component

The `Renderer` component is designed to take an array of these tuples and recursively render them. This effectively flattens the nested providers into a more readable structure:

```tsx
const Renderer = ({ components, children }: { components: FlatedItem[]; children?: ReactNode }) => {
  const renderProvider = (components: FlatedItem[], children: ReactNode): ReactElement => {
    const [tuple, ...restComponent] = components;
    const [Component, componentProps = {}] = tuple as FlatedItem;

    if (Component) {
      return <Component {...componentProps}>{renderProvider(restComponent, children)}</Component>;
    }

    return <>{children}</>;
  };

  return renderProvider(components, children);
};

const FlatedReact = {
  Wrap: Renderer,
  Load: makeFlatedItem,
};
```

### Simplified Usage

The `FlatedReact.Load` function is not strictly necessary for the functionality but serves an important role for TypeScript type-checking of component props. Here’s how you can use FlatedReact to simplify your component structure:

```tsx
export default async function SomeLayoutComponent() {
  const session = await getServerSession(authOptions);
  const accessToken = await getServerAccessToken();
  const AuthSession = session ? { ...session, accessToken } : session;

  return (
    <FlatedReact.Wrap
      components={[
        FlatedReact.Load(TooltipProvider),
        FlatedReact.Load(AuthProvider, { session: AuthSession }),
        FlatedReact.Load(IntercomProvider),
        FlatedReact.Load(EmailVerificationProvider),
        FlatedReact.Load(ThemeProvider, {
          attribute: 'class',
          defaultTheme: 'dark',
          enableSystem: true,
        }),
      ]}
    >
      {/* ...other children */}
    </FlatedReact.Wrap>
  );
}
```

### Typescript support

By using `FlatedReact.Load` function, your IDE and `tsc` compiler can enforce the right props for the React component. For instance, you can see the following VSCode screenshots for type checking:

Typescript checking:
![Typescript checking](https://cdn.hashnode.com/res/hashnode/image/upload/v1722170938317/47e26509-beac-491d-b926-6cff1ecc9a22.png align="center")
![Typescript checking 2](https://cdn.hashnode.com/res/hashnode/image/upload/v1722170927608/677e63f8-5064-4a82-b70b-8fe6cc10765f.png align="center")

Code completion:
![Code completion](https://cdn.hashnode.com/res/hashnode/image/upload/v1722170945791/1a0e5664-fd48-4ae6-8739-aa7b5ed72d2c.png align="center")


### Comparison with Other Solutions

I had read some articles that offered different approaches to this problem. One such article was by Alfredo Salzillo, titled [The React Context Hell](https://dev.to/alfredosalzillo/the-react-context-hell-7p4). This approach involved cloning elements but isn't well-supported in TypeScript, and using `cloneElement` with direct will lead to double renders.

```jsx
// code from: https://dev.to/alfredosalzillo/the-react-context-hell-7p4
// calling `cloneElement` inside MultiProvider will render an other `ReduxProvider` aka call ReduxProvider twice 
<MultiProvider
      providers={[
        <ReduxProvider value={store} />, 
        // ...others, 
      ]}
    >
      <HelloWorld />
    </MultiProvider>
```

Another insightful read was [Navigating React’s Context Hell](https://medium.com/@ambrosekibet576/navigating-reacts-context-hell-f6f4fe5dc23a) by Ambrose Kibet. This article suggested moving to a global state, which, while effective in some scenarios, doesn't always work well with large applications or special providers like session providers in Next.js.

### Versatility Beyond Providers

An important note is that FlatedReact is not limited to simplifying context providers. It supports any React components nested within each other. This flexibility means that you can use FlatedReact to flatten and manage various component hierarchies, ensuring that your component structure remains clean and maintainable, regardless of the specific components involved.

### Testing FlatedReact

To ensure FlatedReact works as expected, I’ve written tests using Jest and React Testing Library. These tests verify that FlatedReact correctly renders nested components and handles different scenarios, such as components with and without props.

```tsx
import { render } from "@testing-library/react";
import FlatedReact from "../dist/cjs"; // Adjust the import based on your project structure

// Mock components for testing
const MockComponentA: React.FC<
  React.PropsWithChildren<{ message: string }>
> = ({ message, children }) => (
  <div>
    A: {message}
    {children}
  </div>
);

const MockComponentB: React.FC<React.PropsWithChildren<{ count: number }>> = ({
  count,
  children,
}) => (
  <div>
    B: {count}
    {children}
  </div>
);

let componentCIndex = 0;
const MockComponentC: React.FC<React.PropsWithChildren> = ({ children }) => (
  <div>
    C: {++componentCIndex}
    {children}
  </div>
);

describe("FlatedReact", () => {
  it("renders nested components correctly", () => {
    const { queryAllByLabelText, container } = render(
      <FlatedReact.Wrap
        components={[
          FlatedReact.Load(MockComponentA, { message: "Hello" }),
          FlatedReact.Load(MockComponentB, { count: 42 }),
          MockComponentC,
        ]}
      >
        <span>Children Content</span>
      </FlatedReact.Wrap>
    );

    expect(queryAllByLabelText("A: Hello")).toBeTruthy();
    expect(queryAllByLabelText("B: 42")).toBeTruthy();
    expect(queryAllByLabelText("C")).toBeTruthy();
    expect(container).toMatchSnapshot();
  });

  it("renders components with default props", () => {
    const { queryAllByLabelText, container } = render(
      <FlatedReact.Wrap
        components={[
          FlatedReact.Load(MockComponentA, { message: "Default Message" }),
          FlatedReact.Load(MockComponentC),
        ]}
      >
        <span>Default Children</span>
      </FlatedReact.Wrap>
    );

    expect(queryAllByLabelText("A: Default Message")).toBeTruthy();
    expect(queryAllByLabelText("C")).toBeTruthy();
    expect(queryAllByLabelText("Default Children")).toBeTruthy();
    expect(container).toMatchSnapshot();
  });

  it("renders without props and `FlatedReact.Load` correctly", () => {
    const { queryAllByLabelText, container } = render(
      <FlatedReact.Wrap
        components={[
          FlatedReact.Load(MockComponentC),
          [MockComponentC],
          [MockComponentC, undefined],
          [MockComponentC, null],
          [MockComponentC, {}],
          MockComponentC,
        ]}
      >
        <span>No Props Children</span>
      </FlatedReact.Wrap>
    );

    expect(queryAllByLabelText("C")).toBeTruthy();
    expect(queryAllByLabelText("No Props Children")).toBeTruthy();
    expect(container).toMatchSnapshot();
  });
});
```

### Testing FlatedReact

* GitHub Repository:
    
    > Explore the code and contribute to the FlatedReact project on GitHub: FlatedReact on GitHub.
    
* NPM Package:
    
    > Install FlatedReact and view detailed documentation: [FlatedReact on NPM](https://www.npmjs.com/package/@cute-me-on-repos/flated-react).
    
* Articles that Inspired the Solution:
    
    * [The React Context Hell](https://dev.to/alfredosalzillo/the-react-context-hell-7p4) by Alfredo Salzillo.
        
    * [Navigating React’s Context Hell](https://medium.com/@ambrosekibet576/navigating-reacts-context-hell-f6f4fe5dc23a) by Ambrose Kibet
        

These resources provide further insight into the challenges of nested React contexts and the different approaches to addressing them, highlighting why FlatedReact offers a more streamlined and TypeScript-friendly solution.

---

I created FlatedReact to share a utility that flattens the provider structure, making it more readable and maintainable. By leveraging TypeScript and React's powerful composition capabilities, FlatedReact helps streamline multiple context providers' setup, transforming how we manage global states in React applications.