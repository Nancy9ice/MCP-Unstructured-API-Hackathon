# MCP-Unstructured-API-Hackathon

## Invoice Tracker

![Project Architecture](<images/Invoice Tracker Project Architecture (1).png>)

This project was built as an inspiration from the [hackathon](https://unstructured.io/blog/unstructured-mcp-virtual-hackathon-build-share-and-win) hosted by [Unstructured](https://unstructured.io/).

Let me briefly explain the problem I intended to solve...

## The Problem

As a business person, have you been in any of these situations:

- You're casually going through your emails when you see an email reminder from a vendor that makes you realize that you're owing him for a service that could be critical to your business if shutdown.

- Your bookkeeper asks, "Hey, did we pay this?" and you’re suddenly digging through six months of Gmail like a detective.

- Your bank balance looks suspicious. "Who the heck is ‘Premium Services LLC,’ and why did they charge us $2,500?" Spoiler: It’s that obscure SaaS tool your team forgot about.

- Your accountant asks for all Q2 utility bills. You spend 45 minutes combing through Dropbox, Slack, and that one intern’s forwarded email chain from April.

- You accidentally pay the same invoice twice—and now you’re stuck begging Vendor Guy for a refund.

**Can you relate to any of these scenarios?**

If yes, what could help you?

An AI Assistant? Hmmmmmm. Probably.

But how can this AI assistant get the invoices in pdf formats and still interpret them properly after you ask certain questions.

That's the problem I intend to solve.

## The Solution

Let's pause here a bit...

I'm not an AI engineer at least not for now (or anytime soon). I'm just playing my part as a data engineer by moving the data and making it useful for analysis.

Now that you're aware of that. Let me brief you on my solution to the problem I described above.

With the help of Unstructured API MCP server, I built an MCP server that interfaces with Claude Desktop so that you can move multiple pdf files containing vendor bill invoices from Google Drive to MongoDB. Before the movement to MongoDB, some transformations were made on the data using features on Unstructured (P.S. Unstructured is a Product).

The following features in Unstructured made the data usable:

- Partitioning: The partitioning feature turns the data from the pdf files into a semi-structured JSON

- Chunking: The chunking feature breaks the data into parts based on the stated criteria (in this case, chunked by similarity). It was also chunked by max of 1000 characters

- Enrichment: This feature was important to me because I didn't want to build a RAG application if I used embeddings. Instead, I skipped the embeddings feature and used this enrichment feature to summarize my data using an integrated Large Language Model. This made it easy to summarize the information on the invoice documents which were then sent to the Mongodb destination.

Here's a screenshot of how the workflow looks:

![Workflow](<images/Workflow Image.png>)

The good thing about this solution is that you can do every single thing from Claude Desktop from creating the Google Drive Source to moving the summarized information to MongoDB to asking Claude questions about your invoices. Except of course the part of creating a search index on the MongoDB interface so that querying the database to answer your questions can be easy and more informative.

In summary, here's what you can do with this solution leveraging Unstructured API MCP Server:

- Create a Google Drive source containing your invoice documents directly from Claude Desktop

- Create a MongoDB destination directly from Claude Desktop 

- Create a workflow that connects the source and destination directly from Claude Desktop while including the transformation steps which can actually be customized by you

- Update the source, destination or workflow directly from Claude Desktop

- Delete the source, destination or workflow directly from Claude Desktop

- Trigger or run the workflow directly from Claude Desktop

- Ask questions on Claude Desktop about the vendor invoices

- And more...

**It is important to note that the invoice pdf files used in this project were invoice documents sourced from Google Images and converted to pdfs**

Now I'll get into the details on how to set this up.

## Prerequisites
- Have Python installed

- Google Cloud Service Account key: You need this to connect to your Google drive that contains the invoice pdfs. You can follow the guide by Unstructured [here](https://youtu.be/ZiRD7hC-wh8?si=K3RLi9BF2S9-Tq2W) on how to set it up

- MongoDB Database and Collection: You need this to host the transformed unstructured data that's now in a semi-structured format. You can follow the guide [here](https://youtu.be/g6qDfbg808M?si=muQnsCuoZcYwB5r4) to set it up.

- Unstructured API Key: You need the API key to access the functionalities of Unstructured API. Follow this [guide](https://youtu.be/FucugLkYB6M?si=Q1TURP3RTdUe3g5S) to set it up

- Claude Desktop: This is where you can access the MCP server so download it [here](https://claude.ai/download)

## Setting up Your Environment

1. Clone the repository

```bash
https://github.com/Nancy9ice/MCP-Unstructured-API-Hackathon.git
```

2. Set up your environment variables in the root folder. Ensure it contains the following environment variables.

```bash
UNSTRUCTURED_API_KEY=
GOOGLEDRIVE_SERVICE_ACCOUNT_KEY=
MONGO_DB_CONNECTION_STRING=
MONGO_DB_DATABASE=
MONGO_DB_COLLECTION=
```

Ensure that the **GOOGLEDRIVE_SERVICE_ACCOUNT_KEY** key is the file name of your service account key that should be present in your root folder too.

3. Unfortunately, I didn't configure resource templates because just as stated [here](https://github.com/modelcontextprotocol/python-sdk/issues/141#:~:text=Browser%20Chrome-,Additional%20context,-Although%20this%20will), resource templates are not visible in Claude Desktop as at when this project was done. So you would have to do some edits on the static resource functions that have the @mcp.resource decorators in the [server.py file](https://github.com/Nancy9ice/MCP-Unstructured-API-Hackathon/blob/main/uns_mcp/server.py). The variable assignments were already made at the first few lines of the function so you can change the values to your desired values.

4. Install dependencies by running the following commands:

    ```bash
    pip install uv
    ```

    ```bash
    uv add "mcp[cli]"
    ```

    ```bash
    uv pip install --upgrade unstructured-client python-dotenv
    ```

5. Run the server to check for errors.

    ```bash
    uv run uns_mcp/server.py
    ```

    Running the server with this command won't produce any output if your code has no errors. It's just a way of checking that there are no code errors before launching Claude Desktop.

6. Integrate your code into Claude Desktop.

    To setup your code in Claude Desktop, do the following:

    - Go to `~/Library/Application Support/Claude/` and create a `claude_desktop_config.json` file.

    - Add this content below to the file:
    ``` bash
    {
        "mcpServers":
        {
            "UNS_MCP":
            {
                "command": "ABSOLUTE/PATH/TO/.local/bin/uv",
                "args":
                [
                    "--directory",
                    "ABSOLUTE/PATH/TO/UNS_MCP/FOLDER/IN/SOURCE/CODE",
                    "run",
                    "server.py"
                ],
                "env":
                {
                "UNSTRUCTURED_API_KEY":"<your key>"
                },
                "disabled": false
            }
        }
    }
    ```

    You can get the unstructured api key after signing up on [unstructured api](https://unstructured.io/)

    To get the absolute path to uv, run the command in your terminal: `which uv`.

    Get the absolute path to the `uns_mcp` folder in the code you cloned from the github repo.

    - Restart Claude Desktop

    - If an error pops up at the top right of claude desktop, click the action button to see the logs

7. Now that everything is fine on Claude Desktop, you can start making your prompts. To confirm that Claude Desktop recognizes the MCP server, you will see the hammer icon circled orange. You can click it to see the list of tools that the configured MCP server provides.

The icon circled blue contains the configured prompt templates and resources that you can use to give predefined prompts and static data to Claude Desktop respectively.

![Claude Desktop](<images/Claude Desktop Launch Page.png>)

8. You can view the [project demo screenshots here](https://amandinancy16.medium.com/how-i-built-an-invoice-tracker-using-unstructured-api-mcp-server-0bafebe6eb3d#7636)


