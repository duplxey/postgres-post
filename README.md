# How to deploy a Postgres-backed web app?

![Back4app Postgres Cover](https://i.ibb.co/n7bLQ6j/back4app-postgres-cover.png)

In this article, you'll learn how to deploy a Postgres-backed application to Back4app.

## What is PostgreSQL?

...

## Relational vs NoSQL databases

...

![Relational databases vs NoSQL databases](https://i.ibb.co/pPVDYb7/back4app-database-types.png)

---
---
---

## How to deploy a Postgres-backed web app to Back4app?

In this article section, you'll learn how to deploy a [Postgres-backed](https://www.postgresql.org/) web application to [Back4app](https://www.back4app.com/).

### Prerequisites

- Experience with [JavaScript ES6](https://www.w3schools.com/js/js_es6.asp), [React](https://react.dev/), and [Next.js](https://nextjs.org/)
- Basic understanding of [Docker](https://www.docker.com/), and BaaS & CaaS cloud model
- JavaScript IDE and [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine
- A free [Back4app](https://www.back4app.com/) and [GitHub](https://github.com/) account

### What is the Back4app Stack?

Before diving into the deployment process, let's briefly discuss what solutions Back4app offers.

1. [**Back4app (BaaS)**](https://www.back4app.com/) is a fully-fledged backend solution. It includes user management, authentication, real-time databases (NoSQL/PostgreSQL), custom code execution, auto-generated APIs, SDKs, push notifications, and more.
2. [**Back4app Containers (CaaS)**](https://www.back4app.com/container-as-a-service-caas) is a Docker-powered container management and deployment platform. It allows you to spin up Docker containers in a few clicks!
6. [**Back4app AI-agent**](https://www.back4app.com/agent) is a brand new AI-powered agent. It allows you to perform all cloud-related tasks with the power of conversation. The agent integrates tightly with the other two Back4app solutions.

Throughout the article, we'll be using Back4app BaaS and Back4app Containers. Nevertheless, you should check out [How to use AI for web development?](https://blog.back4app.com/ai-web-application-development/)  to learn how to leverage AI to speed up your development process.

### Project Overview

We'll be building a simple budget-tracking web application. The web app will allow users to add expenses, remove them, and calculate different statistics (e.g. amount spent, budget percentage).

The app will be split into the backend and the frontend. The backend will be built with Back4app (backed by PostgreSQL), and the frontend will be built with React (using Next.js). We'll connect the two using [Parse SDK](https://parseplatform.org/) and deploy the frontend to Back4app Containers.

### Backend

Let's start off with the backend.

#### Create Back4app App

To create a Back4app app, first navigate to your [Back4app dashboard](https://dashboard.back4app.com/apps) and click "Build new app".

![Back4app Build New App](https://i.ibb.co/FDYy5P0/back4app-create-app.png)

Next, select "Backend as a Service" since we're building a backend.

![Back4app Backend as a Service](https://i.ibb.co/ssDnZDY/back4app-backend-as-a-service.png)

Give your application a descriptive name, select "PostgreSQL" as the database, and click "Create".

![Back4app App Configuration](https://i.ibb.co/DDZm3Xx/back4app-app-configuration.png)

> At the time of writing, there's not much difference between the two database types from the developer's point of view. The same Parse SDK methods apply to both of them.

Back4app will take a little while to prepare everything required for your application. That includes the database, application layer, auto-scaling, auto-backup, and security settings.

As soon as your app is ready, you'll be redirected to the app's real-time database view.

![Back4app Database View](https://i.ibb.co/LxfnQPC/back4app-database-view.png)

#### Database Architecture

Moving along, let's design the database.

Since our app is relatively simple, we'll only need one class. Let's call it `Expense`. 

To create a new database class, click "Create a class", name it `Expense`, and ensure "Public Read and Write enabled" is checked.

![Back4app Create New Class](https://i.ibb.co/qmxGgj4/back4app-create-database-class.png)

> Enabling public read and write is considered bad practice since it allows anyone to perform CRUD operations on your classes. Security is out of the scope of this article. Still, it might be a good idea to review [Parse Server Security](https://blog.back4app.com/parse-server-security/).

By default, database classes come with the following four fields:

```
+-----------+-------------------------------------------------------------------------+
| Name      | Explanation                                                             |
+-----------+-------------------------------------------------------------------------+
| objectId  | Object's unique identifier                                              |
+-----------+-------------------------------------------------------------------------+
| updatedAt | Date time of the object's last update.                                  |
+-----------+-------------------------------------------------------------------------+
| createdAt | Date time of object's creation.                                         |
+-----------+-------------------------------------------------------------------------+
| ACLs      | Allow you to control the access to the object (e.g. read, update).       |
+-----------+-------------------------------------------------------------------------+
```

Take a quick look at them since we'll use them when building the frontend.

Next, add the following fields to the `Expense` class:

```
+-----------+-------------+--------------------+----------+
| Data type | Name        | Default value      | Required |
+-----------+-------------+--------------------+----------+
| String    | name        | <leave blank>      | yes      |
+-----------+-------------+--------------------+----------+
| String    | description | <leave blank>      | no       |
+-----------+-------------+--------------------+----------+
| Number    | price       | 0                  | yes      |
+-----------+-------------+--------------------+----------+
```

After that, populate the database with some sample data. Create a few items by providing the names, descriptions, and prices. Alternatively, you can import this [data dump](https://github.com/duplxey/back4app-postgres/blob/master/export/Expense.json).

![Back4app Database Populated](https://i.ibb.co/qCJJzLX/back4app-populate-database.png)

The test data will later allow us to test the backend and frontend.

#### Cloud Code

Back4app allows you to execute custom JavaScript code via Cloud Code functions. The functions can be scheduled as jobs or invoked by Parse or HTTP requests. Since they're operated within a managed environment, that eliminates the necessity to handle and scale your own servers.

> To learn more about Functions as a Service (FaaS), check out [What are Serverless functions?](https://blog.back4app.com/what-are-serverless-functions-in-cloud-computing/)

We'll utilize a Cloud Code function to calculate the expense statistics.

To create one, select "Cloud Code > Functions & Web Hosting" on the sidebar. Then open *cloud/main.js* and paste in the following code:

```js
// cloud/main.js

const totalBudget = 100;

Parse.Cloud.define("getStatistics", async (request) => {
  const query = new Parse.Query("Expense");
  const totalExpenses = await query.count();
  const results = await query.find();
  
  const totalSpent = results.reduce(
    (sum, expense) => sum + expense.get("price"), 0);
  const spentPercentage = totalSpent > 0 ? 
    Math.round((totalSpent / totalBudget) * 100) : 0;

  return {
    totalExpenses,
    totalSpent,
    totalBudget,
    spentPercentage
  };
});
```

1. This code defines a new Cloud Code function named `getStatistics`.
2. The function aggregates the data and calculates `totalSpent` and `spentPercentage`.

Lastly, click "Deploy" to deploy the function to the cloud.

And we're done with the backend. That was easy!

### Frontend

In this article section, we'll implement the app's frontend.

#### Create Next App

The easiest way to bootstrap a Next.js app is via [`create-next-app`](https://nextjs.org/docs/pages/api-reference/create-next-app) utility. To use it, open the terminal and run the following command: 

```sh
$ npx create-next-app@latest back4app-postgres

√ Would you like to use TypeScript? ... No
√ Would you like to use ESLint? ... Yes
√ Would you like to use Tailwind CSS? ... Yes
√ Would you like to use `src/` directory? ... Yes
√ Would you like to use App Router? (recommended) ... Yes
√ Would you like to customize the default import alias (@)? ... No

Creating a new Next.js app in /back4app-postgres.
```

> If you've never used the `create-next-app` utility, it'll automatically get installed.

Ensure you enable [TailwindCSS](https://tailwindcss.com/) since we'll use it instead of a component library.

Next, clean up the bootstrapped project by performing the following:

1. Delete the contents of the *public/* folder.
2. Keep only the first three lines of *src/app/globals.css*:
    ```css
    /* app/src/globals.css */

    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```
3. Replace *src/app/page.js* with the following:
    ```jsx
    // src/app/page.js

    export default function Page() {
      return (
        <p>Back4app rocks!</p>
      );
    }
    ```

Start the development server:

```sh
$ next dev
```

Open your favorite web browser and navigate to [http://localhost:3000/](http://localhost:3000/). You should see the "Back4app rocks!" message if everything goes well.

#### Views

The frontend will have the following endpoints:

1. `/` displays the table of expenses and expense statistics
2. `/add/` displays a form for adding a new expense
3. `/delete/<objectId>/` displays a confirmation for deleting an expense

To implement these endpoints, create the following directory structure:

```
src/
├── add/
│   └── page.js
└── delete/
    └── [objectId]/
        └── page.js
```

Additionally, create the *components* folder with *Container.js* and *Header.js*:

```
src/
└── components/
    ├── Container.js
    └── Header.js
```

Paste the following in *Container.js*:

```jsx
// src/app/components/Container.js

const Container = ({children}) => {
  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8">
      {children}
    </div>
  )
}

export default Container;
```

And do the same for *Header.js*:

```jsx
// src/app/components/Header.js

import Container from "@/app/components/Container";
import Link from "next/link";

const Header = () => {
  return (
    <Container>
      <div className="py-4">
        <Link href="/">
          <div 
            className="text-2xl font-semibold text-indigo-500 hover:text-indigo-700"
          >
            back4app-postgres
          </div>
        </Link>
      </div>
    </Container>
  )
}

export default Header;
```

Make use of *Container.js* and *Header.js* in *layout.js* like so:

```jsx
// src/app/layout.js

"use client";

import {Inter} from "next/font/google";
import "./globals.css";
import Header from "@/app/components/Header";
import Container from "@/app/components/Container";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Header/>
        <Container>
          {children}
        </Container>
      </body>
    </html>
  );
}
```

Finally, paste the view code into the files accordingly:

1. [*src/app/page.js*](https://github.com/duplxey/back4app-postgres/blob/parseless/src/app/page.js)
2. [*src/app/add/page.js*](https://github.com/duplxey/back4app-postgres/blob/parseless/src/app/add/page.js)
3. [*src/app/delete/[objectId]/page.js*](https://github.com/duplxey/back4app-postgres/blob/parseless/src/app/delete/%5BobjectId%5D/page.js)

Rerun the development server and visit [http://localhost:3000](http://localhost:3000) in your browser. You should see something similar to this:

![Back4app App Postgress](https://i.ibb.co/hYmfbcY/back4app-postgress-app.png)

Clicking on the "Add expense" button should redirect you to the form.

#### Parse SDK

There are multiple ways to connect to a Back4app backend:

1. RESTful API
2. GraphQL API
3. Parse SDK

We'll go with the latter since it's the most robust and straightforward setup.

Parse SDK is a toolkit packed with handy tools for querying data, managing it, running Cloud Code functions, and more. It is available for many programming languages and frameworks, such as JavaScript, PHP, Flutter, and Objective-C.

Start by installing Parse via npm:

```sh
$ npm install parse
```

To use Parse in our React views, we first have to initialize it. But before doing that, we'll create a React context, allowing us to pass the Parse instance to all our views.

Create a *context* folder in the *src/app* folder, and a *parseContext.js* file in it:

```jsx
import {createContext} from "react";

const ParseContext = createContext();

export default ParseContext;
```

Then initialize Parse in *layout.js* and wrap the entire app with `ParseContext.Provider` like so:

```jsx
// src/app/layout.js

import Parse from "parse/dist/parse";
import ParseContext from "@/app/context/parseContext";

Parse.initialize(
  "<your_parse_application_id>",
  "<your_parse_javascript_key>",
);
Parse.serverURL = "https://parseapi.back4app.com/";

export default function RootLayout({ children }) {
  return (
    <ParseContext.Provider value={Parse}>
      <html lang="en">
        // ...
      </html>
    </ParseContext.Provider>
  );
}
```

> Make sure to replace `<your_parse_application_id>` and `<your_parse_javascript_key>` with your actual keys. To obtain them, navigate to your Back4app dashboard and select "App Settings > Security & Keys" on the sidebar.
> 
> ![Back4app API Keys](https://i.ibb.co/vZ8LzCs/back4app-api-keys.png)

We can now obtain the `Parse` instance in our views like so:

```jsx
const parse = useContext(ParseContext);
```

Then, slightly modify the views to invoke Parse methods.

*src/app/page.js*:

```jsx
// src/app/page.js

export default function Page() {

  // ...
    
  const parse = useContext(ParseContext);

  const fetchExpenses = () => {
    const query = new parse.Query("Expense");
    query.find().then((fetchedExpenses) => {
      const expenses = fetchedExpenses.map(expense => ({
        objectId: expense.id,
        name: expense.get("name"),
        description: expense.get("description"),
        price: expense.get("price"),
        createdAt: expense.get("createdAt"),
      }));
      setExpenses(expenses);
      console.log("Expenses fetched successfully.");
    }).catch((error) => {
      console.error("Error while fetching expenses:", error);
    });
  }

  const fetchStatistics = () => {
    parse.Cloud.run("getStatistics").then((statistics) => {
      setStatistics(statistics);
      console.log("Statistics fetched successfully.");
    }).catch((error) => {
      console.error("Error while fetching statistics:", error);
    });
  }

  // ...
}
```

*src/app/add/page.js*:

```jsx
// src/app/add/page.js

export default function Page() {

  // ...
    
  const parse = useContext(ParseContext);

  const onAddClick = () => {
    const Expense = parse.Object.extend("Expense");
    const expense = new Expense();

    expense.set("name", name);
    expense.set("description", description);
    expense.set("price", parseFloat(price));

    expense.save().then((expense) => {
        console.log("Expense created successfully with objectId: ", expense.id);
        router.push("/");
      }, (error) => {
        console.error("Error while creating expense: ", error);
      }
    );
  }

  const onCancelClick = () => {
    router.push("/");
  }
  
  // ...
}

```

*src/app/delete/[objectId]/page.js*:

```jsx
// src/app/delete/[objectId]/page.js

export default function Page() {

  // ...
    
  const parse = useContext(ParseContext);

  const onDeleteClick = () => {
    const Expense = parse.Object.extend("Expense");
    const query = new parse.Query(Expense);

    query.get(objectId).then((expense) => {
      return expense.destroy();
    }).then((response) => {
      console.log("Expense deleted successfully");
      router.push("/");
    }).catch((error) => {
      console.error("Error while deleting expense: ", error);
    });
  }

  const onCancelClick = () => {
    router.push("/");
  }
  
  // ...
}
```

Don't forget about the imports at the top of the file:

```jsx
import {useContext} from "react";
import ParseContext from "@/app/context/parseContext";
```

Great, that's it.

Your frontend is now connected to the backend. If you visit the app in your browser, you should see that the data gets loaded correctly from the backend. All the changes on the frontend are now reflected in the backend.

#### Dockerize

Since Back4app Containers is a CaaS platform, your project must be dockerized before deployment. The recommended way of dockerizing your project is via a Dockerfile. A Dockerfile is a blueprint script providing instructions to create a container image.

Create a *Dockerfile* in the project root:

```dockerfile
# Dockerfile

FROM node:18

WORKDIR /app
COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build
RUN npm install -g next

EXPOSE 3000

CMD ["next", "start", "-p", "3000"]
```

This Dockerfile utilizes the [`node:18-alpine`](https://hub.docker.com/_/node/tags) image, establishes the working directory, manages dependencies, copies over the project, and builds the application. Upon completion, it exposes port `3000` and launches a Next.js server to listen on that port.

Next, create a *.dockerignore* file to minimize the image's size:

```
# .dockerignore

.idea/

node_modules/
.next/
/out/
build/

.vercel
```

> *.dockerignore* files, work in a similar way as *.gitignore* files.

Make sure everything works by building and running the image locally:

```
$ docker build -t back4app-postgres:1.0 .
$ docker run -it -p 3000:3000 back4app-postgres:1.0
```

Open your web browser and navigate to [http://localhost:3000](http://localhost:3000). The web application should still be fully functional.

#### Push to VCS

To deploy your code to Back4app Containers, you must push it to GitHub. To do that, perform the following:

1. Log into your [GitHub account](https://github.com/login).
2. Create a new [repository](https://github.com/new).
3. Copy the remote origin URL -- e.g. `git@github.com:duplxey/repo.git`.
4. Open the terminal and execute the following commands:
   ```
   $ git init
   $ git remote add origin <your_remote_origin_url>
   $ git add . && git commit -m "project init"
   $ git push origin main
    ```
   The commands will initialize a new Git repository, add the remote, commit all the code, and push it to the main branch.
   
Open your favorite web browser and ensure all the code is added to the repository.

#### Deploy Code

Now that the app is dockerized and hosted on GitHub, we can finally deploy it.

Navigate to your [Back4app dashboard](https://dashboard.back4app.com/apps) and click on the "Build new app" button once again.

![Back4app Build New App](https://i.ibb.co/FDYy5P0/back4app-create-app.png)

Select "Containers as a Service" since we're deploying a dockerized application.

![Back4app Containers as a Service](https://i.ibb.co/bRHFWqw/back4app-containers-as-a-service.png)

If it's your first time working with Back4app Containers, you must link your GitHub to your Back4app account. When picking what repositories Back4app has access to, make sure to allow access to the repository created in the previous step.

Next, "Select" the repository.

![Back4app Select Repository](https://i.ibb.co/hcC1yzB/back4app-select-repository.png)

Back4app Containers allow you to configure deployment settings such as port, auto-deploy, environmental variables, and health checks.

Since our app is simple, we need to provide a name and can keep everything else as default.

![Back4app Configure Deployment](https://i.ibb.co/cx63hGC/back4app-configure-repository.png)

As you click "Create App", Back4app will pull the code from GitHub, build the Docker image, push it to the container registry, and deploy it.

After a few moments, your app will be available at the URL on the sidebar.

![Back4app Successful Deployment](https://i.ibb.co/grX0FZZ/back4app-successful-deployment.png)

---
---
---

## Conclusion

...

### Future ideas

- Implement User authentication
- Each user should have their own budget
- Custom domain

Get the final source code from [back4app-postgres](https://github.com/duplxey/back4app-postgres) repo.
