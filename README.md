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

1. [**Back4app (BaaS)**](https://www.back4app.com/) is a fully-fledged backend solution. It includes user management, authentication, real-time databases, custom code execution, auto-generated APIs, SDKs, push notifications, and more.
2. [**Back4app Containers (CaaS)**](https://www.back4app.com/container-as-a-service-caas) is a Docker-powered container management and deployment platform. It allows you to spin up Docker containers in a few clicks!
6. [**Back4app AI-agent**](https://www.back4app.com/agent) is a brand new AI-powered agent. It allows you to perform all cloud-related tasks with the power of conversation. The agent integrates tightly with the other two Back4app solutions.

Throughout the article, we'll be using Back4app BaaS and Back4app Containers. Nevertheless, you should check out [How to use AI for web development?](https://blog.back4app.com/ai-web-application-development/)  to learn how to leverage AI to speed up your development process.

### Project Overview

We'll be building a simple budget-tracking web application. The web app will allow users to add expenses, remove them, and calculate different statistics.

The app will be split into the backend and the frontend. The backend will be built with Back4app (backed by PostgreSQL), and the frontend will be built with React (using Next.js). We'll connect the two using [Parse SDK](https://parseplatform.org/) and, lastly, deploy the frontend to Back4app Containers.

### Backend

Let's start off with the backend.

#### Create Back4app App

To create a Back4app app, first navigate to your [Back4app dashboard](https://dashboard.back4app.com/apps), and click "Build new app".

![Back4app Build New App](https://i.ibb.co/FDYy5P0/back4app-create-app.png)

Next, select "Backend as a Service" since we're building a backend.

![Back4app Backend as a Service](https://i.ibb.co/ssDnZDY/back4app-backend-as-a-service.png)

Name your application, select "PostgreSQL" as the database, and click "Create".

![Back4app App Configuration](https://i.ibb.co/DDZm3Xx/back4app-app-configuration.png)

> At the moment there's not much of a difference between the two database types from the developer's point of view. Parse SDK can be used for both of them, and the same methods can be leveraged.

Back4app will take a little while to prepare everything required for your application. That includes the database, application layer, auto-scaling, auto-backup, and security settings.

As soon as your app is ready, you'll be redirected to your real-time database view.

![Back4app Database View](https://i.ibb.co/LxfnQPC/back4app-database-view.png)

#### Database Architecture

Moving along, let's design the database.

Since our app is fairly simple, we'll only need one class. Let's call it `Expense`. 

To create a new database class click on the "Create a class" button, name it `Expense`, and enable "Public Read and Write".

![Back4app Create New Class](https://i.ibb.co/qmxGgj4/back4app-create-database-class.png)

> Enabling "Public Read and Write" is bad practice, since it allows anyone to perform CRUD operations on your classes. Security is out of scope of this article, but it might be a good idea to review [Parse Server Security](https://blog.back4app.com/parse-server-security/).

By default database classes come with the following four fields:

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
| ACLs      | Allow you to control the access to the object (eg. read, update).       |
+-----------+-------------------------------------------------------------------------+
```

Take a quick look at them, since we'll use them when building the frontend.

Next, add the following additional fields to the `Expense` class:

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

After that populate the database with some sample data. Create a few items by providing the name, description, and prices. Alternatively, you can import this [data dump](https://github.com/duplxey/back4app-postgres/blob/master/export/Expense.json).

![Back4app Database Populated](https://i.ibb.co/qCJJzLX/back4app-populate-database.png)

The test data will later allow us to test the backend and frontend.

#### Cloud Code

Back4app allows you to execute custom JavaScript code via Cloud Code functions. The functions can be scheduled as jobs, or invoked by Parse or HTTP requests. Since they're operated within a managed environment, that eliminates the necessity to handle and scale your own servers.

> To learn more about Functions as a Service (FaaS) check out [What are Serverless functions?](https://blog.back4app.com/what-are-serverless-functions-in-cloud-computing/)

We'll utilize a Cloud Code function to caclulate the expense statistics.

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

Our backend is now more or less complete. That was easy!

### Frontend

In this article section, we'll implement the app's frontend.

#### Create Next App

The easiest way to bootstrap a Next.js app is via [`create-next-app`](https://nextjs.org/docs/pages/api-reference/create-next-app) utility. To use it open the terminal and run the following command: 

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

> In case you've never used the `create-next-app` utility it'll automatically get installed.

> Instead of using a component library we'll utilize [TailwindCSS](https://tailwindcss.com/).

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

Lastly, start the development server:

```sh
$ next dev
```

Open your favorite web browser and navigate to [http://localhost:3000/](http://localhost:3000/). If everything goes well you should see the "Back4app rocks!" message.

#### Views

The frontend will have the following endpoints:

1. `/` displays the table of expenses and expense statistics
2. `/add/` displays a form for adding a new expense
3. `/delete/<objectId>/` displays a confirmation for deleting an expense

To implement these endpoints create the following directory structure:

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

---
---
---

#### Parse SDK

Start by installing Parse via npm:

```sh
$ npm install parse
```

Next, create *parseContext.js* in the *src/app/context* folder:

```jsx
import {createContext} from "react";

const ParseContext = createContext();

export default ParseContext;
```

Wrap the entire app using `ParseContext.Provider` like so:

```jsx
// src/app/layout.js

import Parse from "parse/dist/parse";
import ParseContext from "@/app/context/parseContext";

Parse.initialize(
  process.env.NEXT_PUBLIC_PARSE_APPLICATION_ID,
  process.env.NEXT_PUBLIC_PARSE_JAVASCRIPT_KEY,
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

![Back4app API Keys](https://i.ibb.co/vZ8LzCs/back4app-api-keys.png)

Create *.env.local* file in the project root:

```dotenv
NEXT_PUBLIC_PARSE_APPLICATION_ID=<your_parse_application_id>
NEXT_PUBLIC_PARSE_JAVASCRIPT_KEY=<your_parse_javascript_key>
```

Variables prefixed with `NEXT_PUBLIC_` will be accessible by the client.

Slightly modify the views:

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

Don't forget about the imports:

```jsx
import {useContext} from "react";
import ParseContext from "@/app/context/parseContext";
```

Great, that's it.

Your frontend is now connected to the backend. If you visit the app in your browser you should see that the data gets loaded correctly from the backend. All the changes on the frontend are now reflected in the backend.

---
---
---

#### Dockerization

To facilitate deployment on Back4app Containers, the initial step involves dockerizing your application. Dockerization is the practice of encapsulating code within a container, making it deployable across various environments. The recommended approach for dockerizing an application is by employing a Dockerfile.

The Dockerfile script provides instructions for crafting a Docker container image. It serves as a blueprint to specify the environment, install dependencies, and execute necessary commands for building and running the application. Create a Dockerfile in the project root with the following specifications:

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

This Dockerfile utilizes the node:18-alpine image, establishes the working directory, manages dependencies, copies the project, and builds the application. Upon completion, it exposes port 3000 and launches a Next.js server to listen on that port.


Minimizing the Docker image size is crucial. Achieve this by creating a .dockerignore file, specifying files and folders to exclude from the final image. For instance, exclude IDE files, build, .git, and node_modules. Here's a sample .dockerignore file:

*.dockerignore*

```
# .dockerignore

.idea/

node_modules/
.next/
/out/
build/

.vercel
```

Build, Run, Test

Prior to cloud deployment, locally test your Docker project. Install Docker Desktop for seamless testing. Once installed, build your image:

```
$ docker build -t back4app-postgres:1.0 .
```

List the images to verify successful creation:

```
$ docker run -it -p 3000:3000 back4app-postgres:1.0
```

```
$ docker images

REPOSITORY          TAG       IMAGE ID         CREATED            SIZE
back4app-postgres   1.0       d361534a68da     2 minutes ago      1.08GB
```


[http://localhost:3000/](http://localhost:3000/)

Access http://localhost:3000 in your web browser. Your app should be up and running!

> Terminate the container by pressing CTRL + c on your keyboard.

#### Push to VCS

```
$ git init
$ git add .
$ git commit -m "source code"
$ git push origin master
```

#### Deploy Code

## Conclusion

### Future ideas

- Implement User authentication
- Each user should have their own budget
- 
