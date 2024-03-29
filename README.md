# Messaging App

![](./demo.png)

_Started Thursday November 28th, Wrapped up Tuesday Dec 3rd (tagged as 4.0), next version (v5.0?) worked on since_

🎉 My excitement for this project has continued and so I am still actively adding to app! I've been reorganizing code, adding features, and cleaning up known bugs. To see the state of this project upon initial completion (and submission) see the `4.0` tags of the respective projects [frontend-4.0](https://github.com/Tzinov15/messaging-app-frontend/tree/v4.0) and [backend-4.0](https://github.com/Tzinov15/messaging-app-backend/tree/v4.0)

[Frontend Repo](https://github.com/Tzinov15/messaging-app-frontend) (React 16 (CRA), TypeScript, Hosted on Netlify )
[![Netlify Status](https://api.netlify.com/api/v1/badges/70a17889-81a2-4ffa-8fd0-8f670f8b6cf8/deploy-status)](https://app.netlify.com/sites/hopeful-beaver-35d7aa/deploys)

[Backend Repo](https://github.com/Tzinov15/messaging-app-backend) (Node/Express, WebSockets, TypeScript, MongoDB, Hosted on Heroku, Using Self-Signed Certs)

[EDIT] *The repos above have had 2.0 branches added since. I wanted to keep `master` at what the project looked like when I submitted the assignment (around noon on Dec 4th) but wanted to continue working on the project as far as adding tests, cleaning up code, etc. To see the latest and greatest of the project checkout the `2.0` branches on both the frontend and backend repos*

# Live Link

# [`www.messengy.com`](https://www.messengy.com)

# Where to Start

**To Chat with the Server:**  
Open up the [App](https://hopeful-beaver-35d7aa.netlify.com/`), select the Server user (with the fancy top hat), and chat away! Kind of boring, server only sends automated messages

**To Chat with another Client:**  
Either send the link to a friend and tell them what your username is so they can identify you or just open up the app in a second, **Private/Incognito Window** to generate a second user, chat away!  
⚠️ Opening up a second regular tab/window will reuse the same `localStorage` that "users" get saved to despite opening a second socket connection (known issue, documented below) so make sure you open up either a separate incognito/private window or just message from a different device or browser!

# Design / Architecture Diagrams

This first diagram depicts the lifetime of a chat app, identifying and dissecting 4 distinct phases of the user-experience from a higher-level system view

- New User Connecting
- User selecting a client to chat with
- User sending a message
- User disconnecting
  ![](./lifetime.jpeg)

This diagram is very similar to the lifetime diagram above but is sprinkled with various design decisions, trade-offs, and technical choices that I made or thought of throughout the development
![](./design-decisions-made.jpeg)

And this diagram is a slightly closer view of how data moves and more specifically how the 3 main players in this app - The browser (React App), the server (Web Sockets), and the database (MongoDB) - interact and share data
![](./data-stages.jpeg)

# Future Features

- Implement [IO Types](https://github.com/gcanti/io-ts). This is a run-time type-checking library that allows any type mismatches between domain seams (UI to API, API to DB, etc) that cannot be _compile time_ verified to be verified in a single point of failure at runtime. It's a library I've used numerous times before and has saved me countless troubles tracking down sneaky changes in underlying data contract changes.
- CI/CD Pipeline. Would pull in an E2E testing library like Cypress and have future development be driven through PRs that require automation to pass, code-coverage to not have dipped, etc so that I can confidently refactor and know that I won't break prod
- Use [React Storybook](https://storybook.js.org/) to build out a reusable component library and better stress test individual UI components (username under avatar that is too big, loading states, error states, etc)
- Performance Testing. I would like to set up a performance test runner that can automatically spin up hundreds of client sockets that each fire in hundreds of messages per second to help identify bottlenecks. Do I need to throttle the functions that write to the database? Do I need to throttle renders on incoming messages to the UI? Can I get away with just one server? How does the `wss.clients` object stored in memory behave past a certain point?
- Refactor. I'm a big believer that code never gets written perfectly the first time and that revisiting an app after a while allows you to see different patterns, better approaches, and better abstractions that you may not have noticed the first time. I would go back to the UI and chunk out a lot more components, isolate CSS to be more tightly coupled to the specific component (would likely have pulled in [Styled Components](https://www.styled-components.com/), and separated more of the presentational/rendering logic from the data handling logic. On the server side I'd break out a lot of the helper utility functions into separate files, and abstracted more of the functionality that takes place in the socket handlers

# Things I wish I had done differently

- Done more testing during the development instead of just adding more features. I was extremely eager and excited when I saw the write up and let feature-creep sneak in features that came at the cost of more testing. Always a balance between velocity and testing
- Understand / learn more about scaling problems. What changes in some of the upfront design would have to happen if this was meant for 100,000 users? 1,000,000 users? 1 billion users?
- Dived into Serverless Architecture / Lambda functions. At the beginning I was torn between:

  - diving into WebSockets and having a persistent server with real-time data on the client or
  - forgoing WebSockets, using a naive poll on the client-side, and setting up a server-less API using AWS Lambdas (the option to both was out there, seemed very daunting for this project however)

  I ended up settling on WebSockets for the sake of development speed (have had limited experience from before) but would love to do it over again with Serverless architecture and compare the two solutions

# Known Bugs / Issues

- ~~Message board doesn't scroll as you type, you lose sight of the input chat field as more messages pile up~~ fixed since
- Handling simultaneous socket connections from the same user. A new "user" gets "generated" and "saved" every time a new web app opens. The user is represented by a random avatar and username that get generated when the web app first starts up. This metadata gets sent to the websocket server by virtue of URL parameters in the wss socket connect string. And it gets persisted on the browser side in localStorage. Problems arise when multiple client sockets are opened up under the same username. The UI displays them as multiple users because there multiple sockets when in reality they should be represented as a single user
- ~~No automated tests. As mentioned above, the focus the last week was on features and velocity to validate the feasibility of the technical direction. If I had more time, I would establish a suite of lower level unit tests (tests around some of my helper functions both on the backend and frontend), API tests (set up an automated web socket client that connects to the server, fires in messages, validates for a response), and full E2E tests (Cypress, spin up the UI and API and validate that an end to end test works with sending messages to the Server and getting responses displayed correctly)~~ A single Cypress test has been added to cover the base functionality - web app opens, renders available server "user", allows a message to be sent, and gets a response. Much more tests are needed (including several smaller unit tests), but this one Cypress test covers a wide bredth of functionality including socket conenction, socket message handling with author/recipient, and React rendering
- Currently "solution" for mobile responsiveness is far from ideal. Combination of `mediaQueries` and `react-responsive`, messy unorganized CSS code, etc. Next step will be to clean up the use of styles across the app and likely pull in Styled Components
- ~~Timezone on the server is odd/not lined up with the client (timezone data formats aren't really handled)~~ fixed since
- Text input box doesn't grow as you type your message. Ideally it would grow like a textarea element and grow vertically
- ~~UI starts lagging after a lot of messages are rendered~~
- ~~Chat Message board doesn't update as well as it needs to - when a client disconnects the chat history gets weird and refreshes oddly. Also, when a client disconnects, it should disallow any other users chatting with that client from sending them messages~~ fixed since
- ~~When a client disconnects, if you have the chat window open, you can still send a message to this client. By doing so, the `recipientSocket.send()` call fails and ultimately brings down the server, causing all clients to lose their socket connections, causing havoc. 2 steps to fix this:~~ fixed since

  - One, have the server be smarter and if detects that the recipient client has closed down despite having a message for that user, respond back to author with a message indicating the client is no longer available instead of imploding
  - ~~Two, have the client UI remove the chat screen when it senses a client has disconnect instead of leaving it up and allowing the user to send a message to a non-existent user~~

# Building Locally

All 3 components of this app (React Web App, Express Node Server, MongoDB instance) are hosted in the cloud (Netlify, Heroku, Mongodb Atlas).

**The easiest way to use the app is to simply use the hosted [App](https://www.messengy.com/)!**

However, it is still possible to run locally.

**Running the UI Locally:**  
 `git clone https://github.com/Tzinov15/messaging-app-frontend.git`  
 `yarn start`  
 Visit `localhost:3000`

This will point the UI to the Heroku instance of the WebSocketServer. To point it to a local version, change  
 `wss://secure-shelf-01153.herokuapp.com` to `ws://localhost:9009`

inside of `Main.tsx`

**Running the WS Server Locally:**  
 `git clone https://github.com/Tzinov15/messaging-app-backend.git`  
 Create `.env` file with:

```bash
NODE_ENV=development
SERVER_PORT=9191
MONGO_USER=messaging-app
PORT=9090
MONGO_PASSWORD=<password> # I can provide DB password if needed
MONGO_PATH=@messaging-app-backend-wgo7f.mongodb.net/messaging-app-backend?retryWrites=true&w=majority&authSource=admin
```

`yarn start`
