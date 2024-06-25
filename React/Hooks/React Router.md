# React Router

**Approach #1** Best One :star:

```react
const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/products', element: <Products /> },
])

function App() {
  return (
    <RouterProvider router={router}/>
  )
}
```

**Approach #2**

```react
const routeDefinitations = createRoutesFromElements([
  <Route>
    <Route path="/" element={<Home />} />,
    <Route path="/products" element={<Products />} />
  </Route>
])

const router = createBrowserRouter(routeDefinitations)


function App() {
  return (
    <RouterProvider router={router} />
  )
}
```

2. Activate our Router and load the Route definitions that we defined in the First Step.
3. Make sure that we have all these components that we do wanna load and that we maybe also provide some means of navigating between those pages so that our users can move smoothly between the different pages.

## useLoaderData

```react
import { useLoaderData } from 'react-router-dom';
import EventsList from '../components/EventsList';

function EventsPage() {
	const fetchedEvents  = useLoaderData();
  return (
    <>
       <EventsList events={fetchedEvents} />
    </>
  );
}

export default EventsPage;
```

**fetchedEvents** will be that data return by that loader

```react
const router = createBrowserRouter([
  {
    path: '/', element: <Root />,
    errorElement: <Error />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'events', element: <EventRootLayout />,
        children: [
          {
            index: true,
            element: <EventsPage />,
            loader: async () => {
              const response = await fetch('http://localhost:8080/events')
              if (!response.ok) {
                throw new Error('Fetching events failed.')
              } else {
                const json = await response.json()
                return json.events
              }
            }
          },
          { path: ':id', element: <EventDeatil /> },
          { path: 'new', element: <AddEvent /> },
          { path: ':id/edit', element: <AddEvent /> }
        ]
      },
    ],

  },

```

**Where should loader() code be stored?**

You actually put that loader code into your component file where you need it, so for this example it will be in **EventsPage**

```react
import EventsPage , {loader as eventsLoader }from './pages/EventsPage'

const router = createBrowserRouter([
  {
    path: '/', element: <Root />,
    errorElement: <Error />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'events', element: <EventRootLayout />,
        children: [
          {
            index: true,
            element: <EventsPage />,
            loader: eventsLoader
          },
          { path: ':id', element: <EventDeatil /> },
          { path: 'new', element: <AddEvent /> },
          { path: ':id/edit', element: <AddEvent /> }
        ]
      },
    ],

  },

])
```

```react
import { useLoaderData } from 'react-router-dom';
import EventsList from '../components/EventsList';

function EventsPage() {
	const fetchedEvents = useLoaderData();

	return (
		<>
			<EventsList events={fetchedEvents} />
		</>
	);
}

export default EventsPage;

export async function loader() {
	const response = await fetch('http://localhost:8080/events')
	if (!response.ok) {
		throw new Error('Fetching events failed.')
	} else {
		const json = await response.json()
		console.log(json.events)
		return json.events
	}
}
```

**Where are loader() functions executed?**

The loader for a page will be called right when we start navigating to that page. So not after the page component has been rendered.

## Reflecting the Current Navigation State in the UI

**Approach #1:**

```react
import { Outlet, useNavigation } from "react-router-dom";
import MainNavigation from "../components/MainNavigation";
import classes from './Root.module.css'

function Root() {
  const navigation = useNavigation()

  return <>
    <MainNavigation />
    <main className={classes.content}>
    {
      navigation.state === 'loading' && <p>Loading...</p>
    }
      <Outlet />
    </main>
  </>
}

export default Root
```

## Error Handling with Custom Events

React Router will simply render the closest error element

```react
export async function loader() {
	const response = await fetch('http://localhost:8080/eventddds')
	if (!response.ok) {
		throw {message: 'Something went wrong.'}
	} else {
		return response
  }
}
```

In this section we added an Error Element to the Root Route to have a fallback page thhat would be displayed in case of 404 errors.

```react
const router = createBrowserRouter([
  {
    path: '/', element: <Root />,
    errorElement: <Error />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'events', element: <EventRootLayout />,
        children: [
          {
            index: true,
            element: <EventsPage />,
            loader: eventsLoader
          },
          { path: ':id', element: <EventDeatil /> },
          { path: 'new', element: <AddEvent /> },
          { path: ':id/edit', element: <AddEvent /> }
        ]
      },
    ],

  },

])
```

## Extracting Error Dta & Throwing Responses

**useRouteError** 

This hives us an error object and the shape of that object now depends on wether you threw a response or any other kinf of object or data

If you threw a response as Im doint it right here now, this error will include a status field.

Which actually reflects the status of the response you threw. In my case would be a 500 error.

```react
function Error() {
  const error = useRouteError()
  let title = ""
  let message = ""
  switch (error?.status) {
    case 404:
      {
        title = "Page Not Found"
        message = "Sorry, but the page you were trying to view does not exist."
        break
      }

    case 500:
      {
        title = "Server Error"
        message = JSON.parse(error.data).message
        break
      }

    default:
      {
        title = "Error"
        message = "Sorry, an error has occurred."
        break
      }
  }

  return (
    <>
      <MainNavigation />
      <PageContent title={title}>
        <p>{message}</p>
      </PageContent>
    </>

  )

}
export default Error
```

## The json() Utility Function

React Router gives you a little helper utility instead of creating your response liket his and returning like this 

```react
		throw new Response(
      JSON.stringify({
        message: 'Failed to load events.',
      }),
      { status: 500 }
    )
```

To

```react
	return json({
      message: 'Failed to load events.',
    }, {
      status: response.status,
    })
```

Now with this json function you don't have to parse the json format manually, instead you can simplify your code to this, beacuse the parsing will now be donr by React Router for you.

**Another Example**

```react
case 500:
      {
        title = "Server Error"
        message = JSON.parse(error.data).message
        break
      }
```

To

```react
 case 500:
      {
        title = "Server Error"
        message = error.data.message
        break
      }
```

## useFetcher

This hook when executed gives you an object, and this object inckudes a bunch of useful properties and methods. For example, it gives you another form component which is different from that other component we used before. It also gives you a submit function which is different from the submit function we got from useSubmit, which we used before.

But what is the difference between this form we get here and this submit function which we get here?

Well, if we use this fetcher form component like this, this will actually still trigger an action but it will not initalize a route transition, so fetcher shpuld basically be used whenever you wanna trigger an action or also a loader will help of the load function without actually navigating to the page to which the **loader** belongs or the page to which the **action** belongs.

On this form here we add the action attribute and point to /newletter because I know that I wanna trigger the action off that newsletter route, but i wanna make sure that i dont load that route's component.

```react
 <Form method="post" className={classes.newsletter}>
      <input
        type="email"
        placeholder="Sign up for newsletter..."
        aria-label="Sign up for newsletter"
      />
      <button>Sign up</button>
    </Form>
```

If i use **Form** without fetcher, if I enter some email and click Submit the user will navigate to Newsletter page, and we do **NOT** want this. 

Now I use fetcher, we dont transition, we dont move to a different route

**useFetcher** hook is basically the tool you should use if you want to interact withc some action or a loader without transitioning, so if you waana send your requests behind the scenes, without triggering any route changes.

```react
<fetcher.Form method="post"
            action='/newsletter'
            className={classes.newsletter}>
            <input
                type="email"
                placeholder="Sign up for newsletter..."
                aria-label="Sign up for newsletter"
            />
            <button>Sign up</button>
        </fetcher.Form>
```

##  Defering Data Fetching with defer()