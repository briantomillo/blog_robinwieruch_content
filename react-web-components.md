+++
title = "How to use Web Components in React"
description = "A tutorial on how to use Web Components in React. It comes with a library as a wrapper to use a custom element within your React component for passing props as attributes and event listeners ..."
date = "2019-06-12T13:50:46+02:00"
tags = ["Web Components", "React", "JavaScript"]
categories = ["Web Components", "React", "JavaScript"]
keywords = ["react web components"]
news_keywords = ["react web components"]
hashtag = "#ReactJs"
card = "img/posts/react-web-components/banner_640.jpg"
banner = "img/posts/react-web-components/banner.jpg"
contribute = "react-web-components.md"
headline = "How to use Web Components in React"

summary = "A tutorial on how to use Web Components in React. It comes with a library as a wrapper to use a custom element within your React component for passing props as attributes and event listeners."
+++

{{% sponsorship %}}

{{% pin_it_image "react web components" "img/posts/react-web-components/banner.jpg" "is-src-set" %}}

In this tutorial, you will learn **how to use Web Components, alias Custom Elements, in React**. If you want to get started to build your own Web Components before, check out this tutorial: [Web Components Tutorial](https://www.robinwieruch.de/web-components-tutorial). Otherwise, we will install an external Web Component in this tutorial to use it in React.

You will learn how to pass props as attributes/properties to your Custom Element and how to add event listeners for your Custom Element's events in a React component. In the first step, you will pass props manually, however, afterward I will show you how to use a custom [React Hook](https://www.robinwieruch.de/react-hooks/) to automate this process. The custom React Hook is a library to bridge Web Components to React effortlessly.

{{% chapter_header "From React Components to Web Components: Attributes, Properties and Events" "react-web-components-attributes-properties-events" %}}

Let's say we wanted to use a premade Web Component which represents a Dropdown Component in a React component. We can import this Web Component and render it within our React component.

{{< highlight javascript >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = props => {
  return <road-dropdown />;
};
{{< /highlight >}}

You can install the Web Component via `npm install road-dropdown`. So far, the React Component is only rendering the Custom Element, but no props are passed to it. It isn't as simple as passing the props as attributes the following way, because you need to pass objects, arrays, and functions in a different way to Custom Elements.

{{< highlight javascript "hl_lines=6 7" >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = props => {
  // doesn't work for objects/arrays/functions
  return <road-dropdown {...props} />;
};
{{< /highlight >}}

Let's see how our React component would be used in our React application to get to know the props that we need to pass to our Web Component:

{{< highlight javascript >}}
const props = {
  label: 'Label',
  option: 'option1',
  options: {
    option1: { label: 'Option 1' },
    option2: { label: 'Option 2' },
  },
  onChange: value => console.log(value),
};

return <Dropdown {...props} />;
{{< /highlight >}}

Passing the `label` and `option` property unchanged as attributes to our Web Components is fine:

{{< highlight javascript "hl_lines=5 8 9" >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = ({ label, option, options, onChange }) => {
  return (
    <road-dropdown
      label={label}
      option={option}
    />
  );
};
{{< /highlight >}}

However, we need to do something about the `options` object and the `onChange` function, because they need to be adjusted and cannot be passed simply as attributes. Let's start with the object: Similar to arrays, the object needs to be passed as JSON formatted string to the Web Component instead of a JavaScript object:

{{< highlight javascript "hl_lines=10" >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = ({ label, option, options, onChange }) => {
  return (
    <road-dropdown
      label={label}
      option={option}
      options={JSON.stringify(options)}
    />
  );
};
{{< /highlight >}}

That's it for the object. Next, we need to take care about the function. Rather than passing it as attribute, we need to register an event listener for it. That's where we can use React's useLayoutEffect when the component is rendered for the first time:

{{< highlight javascript "hl_lines=6 8 9 10 11 12 13 14 18" >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = ({ label, option, options, onChange }) => {
  const ref = React.createRef();

  React.useLayoutEffect(() => {
    const { current } = ref;

    current.addEventListener('onChange', customEvent =>
      onChange(customEvent.detail)
    );
  }, [ref]);

  return (
    <road-dropdown
      ref={ref}
      label={label}
      option={option}
      options={JSON.stringify(options)}
    />
  );
};
{{< /highlight >}}

We are creating a reference for our Custom Element -- which is passed as ref attribute to the Custom Element -- to add an event listener in our React hook. Since we are dispatching a custom event from the custom dropdown element, we can register on this `onChange` event and propagate the information up with our own `onChange` handler from the props.  A custom event comes with a detail property to send an optional payload with it.

*Note: If you would have a built-in DOM event (e.g. `click` or `change` event) in your Web Component, you could also register to this event. However, this Web Component already dispatches a custom event which matches the naming convention of React components.*

An improvement would be to extract the event listener callback function in order to remove the listener when the component unmounts.

{{< highlight javascript "hl_lines=9 13 15" >}}
import React from 'react';

import 'road-dropdown';

const Dropdown = ({ label, option, options, onChange }) => {
  const ref = React.createRef();

  React.useLayoutEffect(() => {
    const handleChange = customEvent => onChange(customEvent.detail);

    const { current } = ref;

    current.addEventListener('onChange', handleChange);

    return () => current.removeEventListener('onChange', handleChange);
  }, [ref]);

  return (
    <road-dropdown
      ref={ref}
      label={label}
      option={option}
      options={JSON.stringify(options)}
    />
  );
};
{{< /highlight >}}

That's it for adding an event listener for our callback function that is passed as prop to our Dropdown Component. Therefore, we used a reference attached to the Custom Element to register this event listener. All other properties are passed as attributes to the Custom Element. The `option` and `label` properties are passed without modification. In addition, we passed the `options` object as stringified JSON format. In the end, you should be able to use this Web Component in React now.

{{% chapter_header "React to Web Components Library" "react-web-components-library" %}}

The previous section has shown you how to wire Web Components into React Components yourself. However, this process could be automated with a wrapper that takes care about formatting objects and arrays to JSON and registering functions as event listeners. Let's see how this works with the `useCustomElement` React Hook which can be installed via `npm install use-custom-element`:

{{< highlight javascript "hl_lines=5 8 10" >}}
import React from 'react';

import 'road-dropdown';

import useCustomElement from 'use-custom-element';

const Dropdown = props => {
  const [customElementProps, ref] = useCustomElement(props);

  return <road-dropdown {...customElementProps} ref={ref} />;
};
{{< /highlight >}}

The custom hook gives us all the properties in a custom format by formatting all arrays and objects to JSON, keeping the strings, integers, and booleans intact, and removing the functions from the custom props. Instead, the functions will be registered as event listeners within the hook. Don't forget to pass the ref attribute to your Web Component as well, because as you have seen before, it is needed to register all callback functions to the Web Component.

If you want to know more about this custom hook to integrate Web Components in React, check out its {{% a_blank "documentation" "https://github.com/the-road-to-learn-react/use-custom-element" %}}. There you can also see how to create a custom mapping for props to custom props, because you may want to map an `onClick` callback function from the props to a built-in `click` event in the Web Component. Also, if you have any feedback regarding this hook, let me know about it. In the end, if you are using this Web Components hook for your projects, support it by giving it a star.

<hr class="section-divider">

You have seen that it isn't difficult to use Web Components in React Components. You only need to take care about the JSON formatting and the registering of event listeners. Afterward, everything should work out of the box. If you don't want to do this tedious process yourself, you can use the custom hook for it. Let me know what you think about it in the comments :-)