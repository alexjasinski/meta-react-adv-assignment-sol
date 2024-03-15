# SOLUTION STEPS

In a previous video, you were introduced to a possible solution for the portfolio page, where most of the concepts you learned over the duration of this course were applied in one way or another. However, there are still some interesting extras about the solution that will be illustrated in this reading.

## Header animation

In the Header.js component, there are two React core hooks being used: useRef and useEffect.
Those two are used in conjunction to achieve the smooth animation of the header. If you run the application, you can see that the header hides when I am scrolling down, and shows up when I am scrolling back up.
To implement this behavior, I have to use a side effect and subscribe to the scroll event on the window object using window.addEventListener.
It’s important to remove all subscriptions before the unmounting phase. For that, I have to return a function inside useEffect that performs that task. That’s the window.removeEventListener call you see executed inside that function.

```javascript
useEffect(() => {
  const handleScroll = () => {
    // Business logic
  };

  window.addEventListener('scroll', handleScroll);

  return () => {
    window.removeEventListener('scroll', handleScroll);
  };
}, []);
```

To animate the header, you need to deal with its underlying DOM node and apply some style transition. Do you recall the React way to do that? If you said useRef, you guessed right! That’s what I am doing on the container Box and headerRef holds a reference to the underlying <div> node.

```javascript

const Header = () => {
  const headerRef = useRef(null);

  …
  return (
    <Box
      ref={headerRef}
      {...}
    >
      …
    </Box>
  );
};  …
```

Finally, handleScroll is the handler function that will be called every time there is a change in the vertical scroll position.
The meat of this function resides in the comparison between the previous value and the new value. That determines the direction of the scroll and which style I should apply in order to either show or hide the header. Since I am using transition properties in the container Box component, the change is animated.

```javascript

useEffect(() => {
  let prevScrollPos = window.scrollY;

  const handleScroll = () => {
    const currentScrollPos = window.scrollY;
    const headerElement = headerRef.current;
    if (!headerElement) {
      return;
    }
    if (prevScrollPos > currentScrollPos) {
      headerElement.style.transform = "translateY(0)";
    } else {
      headerElement.style.transform = "translateY(-200px)";
    }
    prevScrollPos = currentScrollPos;
  }

  window.addEventListener('scroll', handleScroll)

  return () => {
    window.removeEventListener('scroll', handleScroll)
  }
}, []);

…
  return (
    <Box
      position="fixed"
      top={0}
      left={0}
      right={0}
      translateY={0}
      transitionProperty="transform"
      transitionDuration=".3s"
      transitionTimingFunction="ease-in-out"
      backgroundColor="#18181b"
      ref={headerRef}
    >
     …
    </Box>
  );
```

## Header navigation

There is another neat trick I would like to show you, which also happens in the Header component.
Let’s see what happens when I click on one of the header sections. Do you see how it nicely animates and scrolls into its position on the page? Let me show you how simple it is to implement something like that. Coming back to the code, I have this handleClick function that is invoked when I click on one of the header navigation items, either Projects or Contact Me.

```javascript
const handleClick = (anchor) => () => {
  const id = `${anchor}-section`;
  const element = document.getElementById(id);
  if (element) {
    element.scrollIntoView({
      behavior: 'smooth',
      block: 'start',
    });
  }
};
```

I have defined some ids in other sections of the page. For instance, the header of the projects section has an id called project-section. The handleClick function is called with the anchor name depending on where the navigation should happen, as per the code below:

```javascript
<HStack spacing={8}>
  <a href="#projects" onClick={handleClick('projects')}>
    Projects
  </a>
  <a href="#contactme" onClick={handleClick('contactme')}>
    Contact Me
  </a>
</HStack>
```

To access that DOM element, you can then use document.getElementById and pass the corresponding ID. Once you have it, you can call element.scrollIntoView with an object as parameter, setting behavior as smooth and block start. Nice and simple, isn’t it?

## Formik and Yup validation

Formik works very nicely with Yup, an open source library that allows you to define validation rules in a declarative way. Let’s break down in detail the rules set for the Contact Me form, as part of the useFormik hook. useFormik hook comes with a validationSchema option as part of its configuration object.

```javascript
const formik = useFormik({
  initialValues: {
    firstName: '',
    email: '',
    type: 'hireMe',
    comment: '',
  },
  onSubmit: (values) => {
    submit('https://john.com/contactme', values);
  },
  validationSchema: Yup.object({
    firstName: Yup.string().required('Required'),
    email: Yup.string().email('Invalid email address').required('Required'),
    comment: Yup.string()
      .min(25, 'Must be at least 25 characters')
      .required('Required'),
  }),
});
```

For the firstName field, the rule states that it has to be a string and it can’t be empty. If empty, Formik will register an error message with the label “Required”.

```javascript

firstName: Yup.string().required("Required"),
```

The email input is also required. Observe how Yup already provides us with common validators out of the box, like one to verify that what users type is a valid email. If incorrect, Formik will register an error on that input with the error message “Invalid email address”. Quite straightforward right?

```javascript
email: Yup.string().email("Invalid email address").required("Required"),
```

Finally, I am making the comment field mandatory, with a minimum length of 25 characters.

```javascript

comment: Yup.string()
 .min(25, "Must be at least 25 characters")
 .required("Required"),
```
