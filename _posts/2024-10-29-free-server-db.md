---
layout: post
title: A free server and database for technical experiments
date: 2024-10-29 00:09:58
description: As app side developers, we get used to do client side dev work and experiments; but for technologies are backend side, we may acknowledge them but not having enough chances or playground for us to test it. Here, I'm using some free cloud-based services to set up an environment for us to do it.
tags: BackEnd, Vercel, MongoDB, API, DB
categories: Node
tabs: true
featured: true
---

## Intuition

<a href="https://vercel.com/">Vercel</a> a static website host, works more than what GitHub pages does, it not only provides static website hosting service, but also we can deploy a node based application. The good point is it's free.

MongoDB, a cloud-based db provider, give us a free plan of 512MB space. Combing the two together, we can build our own state-full backend server free. It's flexibility and ability is good enough for us play or experiment intentions.

## Set up MongoDB

### Concepts to know

MongoDB is an implementation of NoSQL database. It manages data with different concepts compared with traditional SQL database, including database, collection, and document.

The concept database maybe is the only one that keep the same meaning in both MongoDB and a SQL database.

A collection in MongoDB is like what a table is in a SQL database. However, in MongoDB, a collection looks like a normal JSON file.

Instead, a document is not a file. It's just a record in a collection. One collection consists of one or more documents. You can treat a document as a segment of a JSON file, it's the basic block to compose a single collection. The structure of documents in the same collection don't have to the same, thus make the collection flexible to express various info.

{% include figure.liquid loading="eager" path="assets/img/mongo-db/map.png" class="img-fluid rounded z-depth-1" %}

### Management

MongoDB has its own shell to interact with it, the commands used to manipulate the data is straightforward. But sometimes it still take some to think of the right command to use if you're not familiar enough with it.

Fortunately, there's also a GUI tool MongoDB Compass to help make it easy. It's provided by MongoDB official, so you should pay enough confident on it.

{% include figure.liquid loading="eager" path="assets/img/mongo-db/compass.png" class="img-fluid rounded z-depth-1" %}

### Connect string

The connect string is the only thing that we need to link to the MongoDB. Once you find it in your account, we can link to the DB through MongoDB Compass or through any other MongoDB libraries, for example, the npm `mongodb` we'll in the following demo.

## GitHub

GitHub is Vercel friendly. Just create a new repo on GitHub to host our code.

Here we need logics on the backend and some frontend code to call our backend service.

### FrontEnd

To keep things simple, I'll use plain HTML and JS codes to develop the front end. No extra npm packages are used, we'll just use Web and DOM apis to accomplish the logics.

DOM apis to create UI and Web api `XMLHttpRequest` used to make the api requests:

```js
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0">
		<title>Welcome</title>
	</head>
	<body>
			<h1>Welcome</h1>
			<button onclick="getGreeting()">Get greeting</button>
			<p>
				<pre id="greeting"></pre>
			</p>
			<button onclick="getVersionAndDes()">Get version</button>
			<p>
				<pre id="get-version"></pre>
			</p>
			<button onclick="updateDescription()">Update description</button>
			<p>
				<label for="fname">New description:</label>
				<input type="text" id="new_des" name="new_des">
			</p>
			<p>
				<pre id="update-result"></pre>
			</p>
			<script>
				const HOST = 'https://blog-vercel-azure.vercel.app'
				// const HOST = 'http://localhost:3000'
				function getGreeting() {
					fetchAndUpdate(`${HOST}/api/greeting`, 'greeting')
				}
				function getVersionAndDes() {
					fetchAndUpdate(`${HOST}/api/version`, 'get-version')
				}
				function fetchAndUpdate(url, id) {
					const Http = new XMLHttpRequest();
					Http.open("GET", url);
					Http.send();

					Http.onreadystatechange = (e) => {
						const obj = JSON.parse(Http.responseText);
						document.getElementById(id).innerText = JSON.stringify(obj, null, 4)
						document.getElementById("update-result").innerText = ''
					}
				}
				function updateDescription() {
					const new_des = document.getElementById('new_des').value
					console.log(new_des)
					const Http = new XMLHttpRequest();
					Http.open("POST", `${HOST}/api/version`);
					Http.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
					Http.send(`description=${new_des}`);

					Http.onreadystatechange = (e) => {
						document.getElementById("update-result").innerText = JSON.stringify(JSON.parse(Http.responseText), null, 4)
					}
				}
			</script>
	</body>

</html>
```

### Backend

In the backend, we need to npm packages:

1. `@vercel/node`. Used to implement apis on vercel.
1. `mongodb`. Used to manipulate our mongodb.

```js
import { VercelRequest, VercelResponse } from "@vercel/node";
import { MongoClient, MongoServerError } from "mongodb";
const CONNECTION_STRING = process.env["mogodb_connect_string"]

module.exports = async (req: VercelRequest, res: VercelResponse) => {
  const client = await MongoClient.connect(CONNECTION_STRING ?? "");
  const db = await client.db("blog");
  if (req.method == 'GET') {
    var result = await db.collection("demo").find().toArray();
    res.status(200).json(result);
  } else if(req.method == 'POST') {
    try {
      const newDes = req.body['description']
      var updateResult = await db.collection("demo").updateOne({ version: "1.0.0" }, { $set: { description: newDes }})
      res.status(200).json(updateResult);
    } catch(error) {
      if (error instanceof MongoServerError) {
        res.status(405).json('Wrong parameter')
      }
      throw error;
    }
  }
};

```

```markdown
> Note: a environment variable is used to keep our connection string private.
```

## Set up Vercel

Vercel is good at helping us automate the process of deploying our changes from a GitHub project.

First we should connect to the repo on GitHub we want to use. The process is easy by following the guide on Vercel.

Second we have to create a environment variable to help store the connection string used to connect to our MongoDB server. The we can read it as what we did in previous backend server code.

After that, if we push any new commits to our GitHub repo, a new deploy will be made automatically.

{% include figure.liquid loading="eager" path="assets/img/mongo-db/vercel.png" class="img-fluid rounded z-depth-1" %}

## Demo

Here's the final result, feel free to have a try: <a href="https://blog-vercel-azure.vercel.app">Demo</a>
