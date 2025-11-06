# Creating a DApp on Arweave: Querying Arweave

This README documents the first part of the "Creating a DApp" tutorial from the Arweave Wiki, focusing on querying data from the Arweave blockweave. This tutorial is designed for developers building decentralized applications (dApps) using Arweave and React. Estimated time: 25-30 minutes.

The full tutorial series covers building a decentralized social media platform called "Public Square" using Arweave for permanent data storage.

## Overview

In this section, you'll learn how to:
- Set up a basic React application.
- Use the Arweave HTTP API to query transactions.
- Filter transactions by tags (e.g., for posts in a social feed).
- Display the queried data in your app.

Arweave is a permanent, decentralized storage network where data is stored forever. Transactions can be tagged for easy querying, making it ideal for dApps like social media.

## Prerequisites

- Node.js (v14 or higher) installed.
- Basic knowledge of React and JavaScript.
- An Arweave wallet (create one at [arweave.org](https://arweave.org) if needed; no AR tokens required for querying).
- Git for cloning the project (optional).

## Setup

1. **Create a new React app** (if starting from scratch):
   ```
   npx create-react-app public-square-query
   cd public-square-query
   ```

2. **Start the development server**:
   ```
   npm start
   ```
   Your app will be available at `http://localhost:3000`.

3. **Install dependencies** (for this querying tutorial, we use fetch for API calls; later parts use arweave-js):
   No additional installs needed for basic querying, as we'll use native `fetch`.

## Querying Arweave: Step-by-Step

### Step 1: Understand Arweave Querying
Arweave uses an HTTP API accessible via gateways like `arweave.net`. To query:
- Use `/tx` endpoint to get transaction details by ID.
- Use `/graph` (GraphQL-like) or simple tag filters for searching.

For a social dApp, we'll query transactions tagged with `App-Name: PublicSquare` and `Content-Type: text/plain`.

### Step 2: Write the Query Code
Replace the contents of `src/App.js` with the following code. This example fetches recent posts (transactions) tagged as "PublicSquare" posts.

```jsx
import React, { useState, useEffect } from 'react';

function App() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    queryPosts();
  }, []);

  const queryPosts = async () => {
    try {
      // Query recent transactions with specific tags
      const response = await fetch('https://arweave.net/graphql', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          query: `
            {
              transactions(
                tags: [
                  { name: "App-Name", values: "PublicSquare" },
                  { name: "Content-Type", values: "text/plain" }
                ],
                first: 10,
                sort: HEIGHT_DESC
              ) {
                edges {
                  node {
                    id
                    tags {
                      name
                      value
                    }
                    data {
                      size
                    }
                    block {
                      timestamp
                    }
                  }
                }
              }
            }
          `
        })
      });

      const { data } = await response.json();
      const postEdges = data.transactions.edges || [];
      
      // Fetch full data for each post (simplified; in production, use Bundlr or optimize)
      const postDetails = await Promise.all(
        postEdges.map(async (edge) => {
          const txId = edge.node.id;
          const txResponse = await fetch(`https://arweave.net/tx/${txId}`);
          const tx = await txResponse.json();
          return {
            id: txId,
            content: new TextDecoder().decode(new Uint8Array(await (await fetch(`https://arweave.net/${txId}`)).arrayBuffer())),
            timestamp: new Date(edge.node.block.timestamp * 1000).toLocaleString(),
          };
        })
      );

      setPosts(postDetails);
    } catch (error) {
      console.error('Error querying posts:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading posts...</div>;

  return (
    <div className="App">
      <h1>Public Square: Decentralized Social Feed</h1>
      <div>
        {posts.map((post) => (
          <div key={post.id} style={{ border: '1px solid #ccc', margin: '10px', padding: '10px' }}>
            <p>{post.content}</p>
            <small>Posted: {post.timestamp}</small>
          </div>
        ))}
      </div>
      {posts.length === 0 && <p>No posts found. Start by creating one!</p>}
    </div>
  );
}

export default App;
```

**Notes**:
- This uses Arweave's GraphQL API for efficient querying.
- Fetching full data (`/tx/{id}` and raw data) can be rate-limited; for production, use a bundler like Bundlr.
- Image: (If the tutorial has one, it's likely a screenshot of the app displaying posts.)

### Step 3: Run and Test
- Save the file and refresh your browser.
- You should see a list of recent "Public Square" posts (if any exist; otherwise, an empty feed).
- Test by opening the browser console for errors.

### Step 4: Explore Further
- View a transaction: Visit `https://arweave.net/tx/{transactionId}`.
- Add error handling and pagination for better UX.
- For real-time updates, poll the query every 30 seconds.

## Next Steps
This covers querying. Proceed to the next tutorial part:
- [Integrating arweave-js](https://arwiki.arweave.net/#/en/creating-a-dapp-02) for wallet integration and posting.

## Troubleshooting
- **CORS issues**: Use a proxy or browser extension if testing locally.
- **No data**: Ensure tags match existing transactions. Search Arweave explorer for "PublicSquare" tags.
- **Rate limits**: Arweave gateways have limits; consider running your own node.

## Resources
- [Arweave Docs: HTTP API](https://docs.arweave.org/developers/server/http-api)
- [Arweave GraphQL](https://docs.arweave.org/developers/server/http-api#graphql-api)
- Full Tutorial Series: [Arweave Wiki](https://arwiki.arweave.net/#/en/creating-a-dapp)

## License
This project is based on the open-source Public Square protocol. See [GitHub repo](https://github.com/DanMacDonald/public-square-app) for details.
