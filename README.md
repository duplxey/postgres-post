# How to deploy a Postgres-backed web app?

![Back4app Postgres Cover](https://i.ibb.co/n7bLQ6j/back4app-postgres-cover.png)

In this article, you'll learn how to deploy a Postgres-backed application to Back4app.

## What is PostgreSQL?

...

## Relational vs NoSQL databases

...

![Relational databases vs NoSQL databases](https://i.ibb.co/pPVDYb7/back4app-database-types.png)

## How to deploy a Postgres-backed web app to Back4app?

...

### Back4app Stack

Before moving along, let's talk about the Back4app stack.

At the time of writing, Back4app offers three services:

- Back4app BaaS
- Back4app Containers
- Back4app AI-agent

In this article, we'll utilize Back4app BaaS to build our backend and Back4app Containers to deploy our frontend.

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

![Back4app Backend as a Service (BaaS)](https://i.ibb.co/MVHby8X/back4app-backend-as-a-service.png)

### Prerequisites

### Project Overview

database, application layer, auto-scaling, auto-backup, and security

### Backend

### Frontend

#### create-next-app

```sh
$ npx create-next-app@latest back4app-postgres

√ Would you like to use TypeScript? ... No
√ Would you like to use ESLint? ... Yes
√ Would you like to use Tailwind CSS? ... Yes
√ Would you like to use `src/` directory? ... Yes
√ Would you like to use App Router? (recommended) ... Yes
√ Would you like to customize the default import alias (@/*)? ... No

Creating a new Next.js app in /back4app-postgres.
```

remove everything in /public folder

change *app/src/globals.css* to this:

```css
/* app/src/globals.css */

@tailwind base;
@tailwind components;
@tailwind utilities;
```

```jsx
// src/app/page.js

export default function Page() {
  return (
    <p>Back4app rocks!</p>
  );
}
```

```sh
$ next dev
```

[http://localhost:3000/](http://localhost:3000/)

#### Views

```
src/
├── components/
│   ├── Container.js
│   └── Header.js
├── layout.js
└── page.js
```

```
const Container = ({children}) => {
  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8">
      {children}
    </div>
  )
}

export default Container;
```

```
import Container from "@/app/components/Container";
import Link from "next/link";

const Header = () => {
  return (
    <Container>
      <div className="py-4">
        <Link href="/">
          <div className="text-2xl font-semibold text-indigo-500 hover:text-indigo-700">
            back4app-postgres
          </div>
        </Link>
      </div>
    </Container>
  )
}

export default Header;
```

```
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

```
"use client";

import {useEffect, useState} from "react";
import {useRouter} from "next/navigation";

export default function Page() {

  const router = useRouter();

  const [items, setItems] = useState([]);
  const [statistics, setStatistics] = useState({
    totalItems: 0,
    totalSpent: 0,
    totalBudget: 0,
    spentPercentage: 0,
  });

  const fetchItems = () => {
    console.log("TODO: fetchItems()");
  }

  const fetchStatistics = () => {
    console.log("TODO: fetchStatistics()");
  }

  useEffect(() => {
    fetchItems();
    fetchStatistics();
  }, []);

  return (
    <div className="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4">
      <div className="w-full basis-3/4">
        <div className="bg-gray-50 p-6 rounded-lg shadow-md space-y-4">
          <div className="text-lg font-semibold">Items</div>
          <div className="relative overflow-x-auto">
            <table className="w-full text-left rtl:text-right text-gray-500 dark:text-gray-400">
              <thead className="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                <tr>
                  <th scope="col" className="px-6 py-3">
                    #
                  </th>
                  <th scope="col" className="px-6 py-3">
                    Item name
                  </th>
                  <th scope="col" className="px-6 py-3">
                    Item description
                  </th>
                  <th scope="col" className="px-6 py-3">
                    Price
                  </th>
                  <th scope="col" className="px-6 py-3">
                    Date added
                  </th>
                  <th scope="col" className="px-6 py-3">
                    Actions
                  </th>
                </tr>
              </thead>
              <tbody>
                {items.map((item, index) => (
                  <tr key={index} className="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                    <th scope="row" className="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                      {item.objectId}
                    </th>
                    <td className="px-6 py-4">
                      {item.name}
                    </td>
                    <td className="px-6 py-4">
                      {item.description}
                    </td>
                    <td className="px-6 py-4">
                      ${item.price}
                    </td>
                    <td className="px-6 py-4">
                      {item.createdAt.toString()}
                    </td>
                    <td className="px-6 py-4 space-x-2">
                      <button
                        className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
                        onClick={() => router.push("/delete/" + item.objectId)}
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          <button
            className="bg-indigo-500 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded"
            onClick={() => router.push("/add/")}
          >
            Add item
          </button>
        </div>
      </div>
      <div className="w-full basis-1/4">
        <div className="bg-gray-50 p-6 rounded-lg shadow-md space-y-4">
          <div className="text-lg font-semibold">
            Statistics
          </div>
          <ul className="mr-2 list-disc list-inside">
            <li>Total items: <span className="text-gray-500">{statistics.totalItems}</span></li>
            <li>Total spent: <span className="text-gray-500">${statistics.totalSpent}</span></li>
            <li>Total budget: <span className="text-gray-500">${statistics.totalBudget}</span></li>
          </ul>
          <div className="w-full bg-gray-200 rounded-full h-2.5 dark:bg-gray-700">
            <div className="bg-indigo-600 h-2.5 rounded-full" style={{width: statistics.spentPercentage + "%"}}></div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

```
"use client";

import {useState} from "react";
import {useRouter} from "next/navigation";

export default function Page() {

  const router = useRouter();

  const [name, setName] = useState("");
  const [description, setDescription] = useState("");
  const [price, setPrice] = useState(0);

  const onAddClick = () => {
    console.log("TOOD: onAddClick()");
  }

  const onCancelClick = () => {
    router.push("/");
  }

  return (
    <div className="bg-gray-50 p-6 rounded-lg shadow-md space-y-4">
      <div className="text-lg font-semibold">Add item</div>
      <div className="space-y-2">
        <label className="block text-sm font-medium text-gray-700 dark:text-gray-400">
          Item name
        </label>
        <input
          name="name"
          type="text"
          className="border border-gray-200 dark:border-gray-700 rounded-md w-full p-2"
          placeholder="Item name"
          value={name}
          onChange={(event) => setName(event.target.value)}
        />
      </div>
      <div className="space-y-2">
        <label className="block text-sm font-medium text-gray-700 dark:text-gray-400">
          Item description
        </label>
        <input
          name="description"
          type="text"
          className="border border-gray-200 dark:border-gray-700 rounded-md w-full p-2 mt-2"
          placeholder="Item description"
          value={description}
          onChange={(event) => setDescription(event.target.value)}
        />
      </div>
      <div className="space-y-2">
        <label className="block text-sm font-medium text-gray-700 dark:text-gray-400">
          Item price (in USD)
        </label>
        <input
          name="price"
          type="number"
          className="border border-gray-200 dark:border-gray-700 rounded-md w-full p-2 mt-2"
          placeholder="Price"
          value={price}
          onChange={(event) => setPrice(event.target.value)}
        />
      </div>
      <div className="space-x-2">
        <button
          className="bg-indigo-500 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded"
          onClick={onAddClick}
        >
          Add item
        </button>
        <button
          className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
          onClick={onCancelClick}
        >
          Cancel
        </button>
      </div>
    </div>
  );
}
```

```
"use client";

import {useParams, useRouter} from "next/navigation";

export default function Page() {

  const router = useRouter();

  const params = useParams();
  const objectId = params.objectId;

  const onDeleteClick = () => {
    console.log("TODO: onDeleteClick()");
  }

  const onCancelClick = () => {
    router.push("/");
  }

  return (
    <div className="bg-gray-50 p-6 rounded-lg shadow-md space-y-4">
      <div className="text-lg font-semibold">Delete item</div>
      <div className="">
        Are you sure you want to delete item #{objectId}?
      </div>
      <div className="space-x-2">
        <button
          className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
          onClick={onDeleteClick}
        >
          Delete item
        </button>
        <button
          className="bg-indigo-500 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded"
          onClick={onCancelClick}
        >
          Cancel
        </button>
      </div>
    </div>
  );
}
```

#### Backend Connect

```
$ npm install parse
```

*src/app/context/parseContext.js*:

```
import {createContext} from "react";

const ParseContext = createContext();

export default ParseContext;
```

```
"use client";

import {Inter} from "next/font/google";
import "./globals.css";
import Header from "@/app/components/Header";
import Container from "@/app/components/Container";
import Parse from "parse/dist/parse";
import ParseContext from "@/app/context/parseContext";

const inter = Inter({ subsets: ["latin"] });

Parse.initialize(
  process.env.NEXT_PUBLIC_PARSE_APPLICATION_ID,
  process.env.NEXT_PUBLIC_PARSE_JAVASCRIPT_KEY,
);
Parse.serverURL = "https://parseapi.back4app.com/";

export default function RootLayout({ children }) {
  return (
    <ParseContext.Provider value={Parse}>
      <html lang="en">
        <body className={inter.className}>
          <Header/>
          <Container>
            {children}
          </Container>
        </body>
      </html>
    </ParseContext.Provider>
  );
}
```

*.env.local*:

```
NEXT_PUBLIC_PARSE_APPLICATION_ID=7BzIPmHLuJO79TKK4QASWTQUhstNnIQbtDmrLAX3
NEXT_PUBLIC_PARSE_JAVASCRIPT_KEY=has7Kpxy3hhERCGs7BD6mHCdqTMIyrPpNTi2BHjv
```

```
// ...

export default function Page() {

  const router = useRouter();
  const parse = useContext(ParseContext);

  // ...

  const fetchItems = () => {
    const query = new parse.Query("Item");
    query.find().then((fetchedItems) => {
      const items = fetchedItems.map(item => ({
        objectId: item.id,
        name: item.get("name"),
        description: item.get("description"),
        price: item.get("price"),
        createdAt: item.get("createdAt"),
      }));
      setItems(items);
      console.log("Items fetched successfully.");
    }).catch((error) => {
      console.error("Error while fetching items:", error);
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

  useEffect(() => {
    fetchItems();
    fetchStatistics();
  }, []);

  return (
    // ...
  );
}

```

Don't forget about the imports:

```
import {useContext} from "react";
import ParseContext from "@/app/context/parseContext";
```

```
// ...

export default function Page() {

  const router = useRouter();
  const parse = useContext(ParseContext);
  
  // ...

  const onAddClick = () => {
    const Item = parse.Object.extend("Item");
    const item = new Item();

    item.set("name", name);
    item.set("description", description);
    item.set("price", parseFloat(price));

    item.save().then((item) => {
        console.log("Item created successfully with objectId: ", item.id);
        router.push("/");
      }, (error) => {
        console.error("Error while creating Item: ", error);
      }
    );
  }

  const onCancelClick = () => {
    router.push("/");
  }

  return (
      // ...
  );
}

```

```
// ...

export default function Page() {

  const router = useRouter();
  const parse = useContext(ParseContext);
  
  // ...

  const onDeleteClick = () => {
    const Item = parse.Object.extend("Item");
    const query = new parse.Query(Item);

    query.get(objectId).then((item) => {
      return item.destroy();
    }).then((response) => {
      console.log("Item deleted successfully");
      router.push("/");
    }).catch((error) => {
      console.error("Error while deleting Item: ", error);
    });
  }

  const onCancelClick = () => {
    router.push("/");
  }

  return (
    // ...
  );
}
```

#### Dockerization

Dockerfile

```
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

*.dockerignore*

```
.idea/
node_modules/
build/
/out/
.next/
```

```
$ docker build -t back4app-postgres:1.0 .
```

```
$ docker run -it -p 3000:3000 back4app-postgres:1.0
```

[http://localhost:3000/](http://localhost:3000/)


#### Push to VCS

1. Create a GitHub repository.

```
$ git init
$ git add .
$ git commit -m "source code"
$ git push origin master
```

#### Deploy Code

Back4app CaaS

## Conclusion

### Future ideas

- Implement User authentication
- Each user should have their own budget
- 
