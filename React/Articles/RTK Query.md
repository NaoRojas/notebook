# RTK Query

Web applications normally need to fetch data from a server in order to display it. They also usually need to make updates to that data, send those updates to the server, and keep the cached data on the client in sync with the data on the server. This is made more complicated by the need to implement other behaviors used in today's applications:

- Tracking loading state in order to show UI spinners
- Avoiding duplicate requests for the same data
- Optimistic updates to make the UI feel faster
- Managing cache lifetimes as the user interacts with the UI