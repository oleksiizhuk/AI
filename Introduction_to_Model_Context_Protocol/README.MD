# Introduction to Model Context Protocol

# Welcome to the course

![Screenshot 2026-07-31 at 14.51.49.png](image/Screenshot%202026-07-31%20at%2014.51.49.png)

========================================== **Introducing MCP** ==========================================

# Introducing MCP

Model Context Protocol (MCP) is a communication layer that provides Claude with context and tools without requiring you
to write a bunch of tedious integration code. Think of it as a way to shift the burden of tool definitions and execution
away from your server to specialized MCP servers.

## The Problem MCP Solves

Let's say you're building a chat interface where users can ask Claude about their GitHub data. A user might ask "What
open pull requests are there across all my repositories?" To handle this, Claude needs tools to access GitHub's API.

GitHub has massive functionality - repositories, pull requests, issues, projects, and tons more. Without MCP, you'd need
to create an incredible number of tool schemas and functions to handle all of GitHub's features.

![Screenshot 2026-07-31 at 14.54.12.png](image/Screenshot%202026-07-31%20at%2014.54.12.png)

This means writing, testing, and maintaining all that integration code yourself. That's a lot of effort and ongoing
maintenance burden.

## How MCP Works

MCP shifts this burden by moving tool definitions and execution from your server to dedicated MCP servers. Instead of
you authoring all those GitHub tools, an MCP Server for GitHub handles it.

The MCP Server wraps up tons of functionality around GitHub and exposes it as a standardized set of tools. Your
application connects to this MCP server instead of implementing everything from scratch.

## MCP Servers Explained

MCP Servers provide access to data or functionality implemented by outside services. They act as specialized interfaces
that expose tools, prompts, and resources in a standardized way.

In our GitHub example, the MCP Server for GitHub contains tools like get_repos () and connects directly to GitHub's API.
Your server communicates with the MCP server, which handles all the GitHub-specific implementation details.

## Common Questions

### **Who authors MCP Servers?**

Anyone can create an MCP server implementation. Often, service providers themselves will make their own official MCP
implementations. For example, AWS might release an official MCP server with tools for their various services.

### **How is this different from calling APIs directly?**

MCP servers provide tool schemas and functions already defined for you. If you want to call an API directly, you'll be
authoring those tool definitions on your own. MCP saves you that implementation work.

### **Isn't MCP just the same as tool use?**

========================================== **Introducing MCP** ==========================================

# MCP clients

The MCP client serves as the communication bridge between your server and MCP servers. It's your access point to all the
tools that an MCP server provides, handling the message exchange and protocol details so your application doesn't have
to.

## Transport Agnostic Communication

One of MCP's key strengths is being transport agnostic - a fancy way of saying the client and server can communicate
over different protocols depending on your setup.

The most common setup runs both the MCP client and server on the same machine, communicating through standard
input/output. But you can also connect them over:

- HTTP
- WebSockets
- Various other network protocols

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849224_09_-
_002_-_MCP_Clients_03.1749849223875.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849224_09_-_002_-_MCP_Clients_03.1749849223875.png)

## MCP Message Types

Once connected, the client and server exchange specific message types defined in the MCP specification. The main ones
you'll work with are:

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849224_09_-
_002_-_MCP_Clients_05.1749849224367.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849224_09_-_002_-_MCP_Clients_05.1749849224367.png)

ListToolsRequest/ListToolsResult: The client asks the server "what tools do you provide?" and gets back a list of
available tools.

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849225_09_-
_002_-_MCP_Clients_07.1749849224854.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849225_09_-_002_-_MCP_Clients_07.1749849224854.png)

CallToolRequest/CallToolResult: The client asks the server to run a specific tool with given arguments, then receives
the results.

## How It All Works Together

Here's a complete example showing how a user query flows through the entire system - from your server, through the MCP
client, to external services like GitHub, and back to Claude.

Let's say a user asks "What repositories do I have?" Here's the step-by-step flow:

1) User Query: The user submits their question to your server
2) Tool Discovery: Your server needs to know what tools are available to send to Claude
3) List Tools Exchange: Your server asks the MCP client for available tools
4) MCP Communication: The MCP client sends a **ListToolsRequest** to the MCP server and receives a **ListToolsResult**
5) Claude Request: Your server sends the user's query plus the available tools to Claude
6) Tool Use Decision: Claude decides it needs to call a tool to answer the question
7) Tool Execution Request: Your server asks the MCP client to run the tool Claude specified
8) External API Call: The MCP client sends a CallToolRequest to the MCP server, which makes the actual GitHub API call
9) Results Flow Back: GitHub responds with repository data, which flows back through the MCP server as a CallToolResult
10) Tool Result to Claude: Your server sends the tool results back to Claude
11) Final Response: Claude formulates a final answer using the repository data
12) User Gets Answer: Your server delivers Claude's response back to the user

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849232_09_-
_002_-_MCP_Clients_19.1749849231568.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849232_09_-_002_-_MCP_Clients_19.1749849231568.png)

Yes, this flow involves many steps, but each component has a clear responsibility. The MCP client abstracts away the
complexity of server communication, letting you focus on your application logic while still getting access to powerful
external tools and data sources.

Understanding this flow is crucial because you'll see all these pieces when building your own MCP clients and servers in
the upcoming sections.

# Project setup

# Defining tools with MCP

Building an MCP server becomes much simpler when you use the official Python SDK. Instead of writing complex JSON
schemas by hand, you can define tools with decorators and let the SDK handle the heavy lifting.

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849221_09_-
_004_-_Defining_Tools_with_MCP_00.1749849221750.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849221_09_-_004_-_Defining_Tools_with_MCP_00.1749849221750.png)

In this example, we're creating a document management server with two core tools: one to read documents and another to
update them. All documents exist in memory as a simple dictionary where keys are document IDs and values are the
content.

## Setting Up the MCP Server

The Python MCP SDK makes server creation straightforward. You can initialize a server with just one line:

```
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DocumentMCP", log_level="ERROR")
```

Your documents can be stored in a simple dictionary structure:

```
docs = {
"deposition.md": "This deposition covers the testimony of Angela Smith, P.E.",
"report.pdf": "The report details the state of a 20m condenser tower.",
"financials.docx": "These financials outline the project's budget and expenditures",
"outlook.pdf": "This document presents the projected future performance of the system",
"plan.md": "The plan outlines the steps for the project's implementation.",
"spec.txt": "These specifications define the technical requirements for the equipment"
}
```

## Creating a Document Reader Tool

The first tool reads document contents by ID. Here's the complete implementation:

```
@mcp.tool(
name="read_doc_contents",
description="Read the contents of a document and return it as a string."
)
def read_document(
doc_id: str = Field(description="Id of the document to read")
):
if doc_id not in docs:
raise ValueError(f"Doc with id {doc_id} not found")

    return docs[doc_id]
```

The decorator specifies the tool name and description, while the function parameters define the required arguments. The
Field class from Pydantic provides argument descriptions that help Claude understand what each parameter expects.

## Building a Document Editor Tool

The second tool performs simple find-and-replace operations on documents:

```
@mcp.tool(
    name="edit_document",
    description="Edit a document by replacing a string in the documents content with a new string."
)
def edit_document(
    doc_id: str = Field(description="Id of the document that will be edited"),
    old_str: str = Field(description="The text to replace. Must match exactly, including whitespace."),
    new_str: str = Field(description="The new text to insert in place of the old text.")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    
    docs[doc_id] = docs[doc_id].replace(old_str, new_str)

```

This tool takes three parameters: the document ID, the text to find, and the replacement text. The implementation
includes error handling for missing documents and performs a straightforward string replacement.

## Key Benefits of the SDK Approach

- No manual JSON schema writing required
- Type hints provide automatic validation
- Clear parameter descriptions help Claude understand tool usage
- Error handling integrates naturally with Python exceptions
- Tool registration happens automatically through decorators

The MCP Python SDK transforms tool creation from a complex schema-writing exercise into simple Python function
definitions. This approach makes it much easier to build and maintain MCP servers while ensuring Claude receives
properly formatted tool specifications.

# The server inspector

When building MCP servers, you need a way to test your functionality without connecting to a full application. The
Python MCP SDK includes a built-in browser-based inspector that lets you debug and test your server in real-time.

# Starting the Inspector

First, make sure your Python environment is activated (check your project's README for the exact command). Then run the
inspector with:

```
mcp dev mcp_server.py
```

This starts a development server and gives you a local URL, typically something like http://127.0.0.1:6274. Open this
URL in your browser to access the MCP Inspector.

## Using the Inspector Interface

The inspector interface is actively being developed, so it may look different when you use it. However, the core
functionality remains consistent. Look for these key elements:

- A Connect button to start your MCP server
- Navigation tabs for Resources, Tools, Prompts, and other features
- A tools listing and testing panel

Click the Connect button first to initialize your server. You'll see the connection status change from "Disconnected" to
"Connected".

## Testing Your Tools

Navigate to the Tools section and click "List Tools" to see all available tools from your server. When you select a
tool, the right panel shows its details and input fields.

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849212_09_-
_005_-_The_Server_Inspector_10.1749849212534.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849212_09_-_005_-_The_Server_Inspector_10.1749849212534.png)

For example, to test a document reading tool:

1) Select the read_doc_contents tool
2) Enter a document ID (like "deposition.md")
3) Click "Run Tool"
4) Check the results for success and expected output

The inspector shows both the success status and the actual returned data, making it easy to verify your tool works
correctly.

## Testing Tool Interactions

You can test multiple tools in sequence to verify complex workflows. For instance, after using an edit tool to modify a
document, immediately test the read tool to confirm the changes were applied correctly.

The inspector maintains your server state between tool calls, so edits persist and you can verify the complete
functionality of your MCP server.

## Development Workflow

The MCP Inspector becomes an essential part of your development process. Instead of writing separate test scripts or
connecting to full applications, you can:

- Quickly iterate on tool implementations
- Test edge cases and error conditions
- Verify tool interactions and state management
- Debug issues in real-time

- This immediate feedback loop makes MCP server development much more efficient and helps catch issues early in the
  development process.

# Implementing a client

Now that we have our MCP server working, it's time to build the client side. The client is what allows our application
code to communicate with the MCP server and access its functionality.

# Understanding the Client Architecture

In most real-world projects, you'll either implement an MCP client or an MCP server - not both. We're building both in
this project just so you can see how they work together.

The MCP client consists of two main components:

- MCP Client - A custom class we create to make using the session easier
- Client Session - The actual connection to the server (part of the MCP Python SDK)

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849286_09_-
_006_-_Implementing_a_Client_02.1749849285884.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849286_09_-_006_-_Implementing_a_Client_02.1749849285884.png)

The client session requires careful resource management - we need to properly clean up connections when we're done.
That's why we wrap it in our own class that handles all the cleanup automatically.

## How the Client Fits Into Our Application

Remember our application flow diagram? The client is what enables our code to interact with the MCP server at two key
points:

![instructor_a46l9irobhg0f5webscixp0bs_public_1749849286_09_-
_006_-_Implementing_a_Client_06.1749849286672.png](image/instructor_a46l9irobhg0f5webscixp0bs_public_1749849286_09_-_006_-_Implementing_a_Client_06.1749849286672.png)

Our CLI code uses the client to:

- Get a list of available tools to send to Claude
- Execute tools when Claude requests them

## Implementing Core Client Functions

We need to implement two essential functions: ```list_tools()``` and ```call_tool().```

## List Tools Function

This function gets all available tools from the MCP server:

```aiignore
async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools
```

It's straightforward - we access our session (the connection to the server), call the built-in list_tools () method, and
return the tools from the result.

## Call Tool Function

```aiignore
async def call_tool(
    self, tool_name: str, tool_input: dict
) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)
```

We pass the tool name and input parameters (provided by Claude) to the server and return the result.

## Testing the Client

```
uv run mcp_client.py
```

## Putting It All Together

Once the client functions are implemented, you can test the complete flow by running your main application:

```
uv run main.py
```

Try asking: "What is the contents of the report.pdf document?"

Here's what happens behind the scenes:

- Your application uses the client to get available tools
- These tools are sent to Claude along with your question
- Claude decides to use the read_doc_contents tool
- Your application uses the client to execute that tool
- The result is returned to Claude, who then responds to you

The client acts as the bridge between your application logic and the MCP server's functionality, making it easy to
integrate powerful tools into your AI workflows.

# Defining resources

# Accessing resources

# How Prompts Work

Prompts define a set of user and assistant messages that clients can use. They should be high-quality, well-tested, and
relevant to your MCP server's purpose. The workflow is:

- Write and evaluate a prompt relevant to your server's functionality
- Define the prompt in your MCP server using the @mcp.prompt decorator
- Clients can request the prompt at any time
- Arguments provided by the client become keyword arguments in your prompt function
- The function returns formatted messages ready for the AI model

This system creates reusable, parameterized prompts that maintain consistency while allowing customization through
variables. It's particularly useful for complex workflows where you want to ensure the AI receives properly structured
instructions every time.

# MCP review

Now that we've built our MCP server, let's review the three core server primitives and understand when to use each one.
The key insight is that each primitive is controlled by a different part of your application stack.

# Tools: Model-Controlled

Tools are controlled entirely by Claude. The AI model decides when to call these functions, and the results are used
directly by Claude to accomplish tasks.

Tools are perfect for giving Claude additional capabilities it can use autonomously. When you ask Claude to "calculate
the square root of 3 using JavaScript," it's Claude that decides to use a JavaScript execution tool to run the
calculation.

# Resources: App-Controlled

Resources are controlled by your application code. Your app decides when to fetch resource data and how to use it -
typically for UI elements or to add context to conversations.

In our project, we used resources in two ways:

- Fetching data to populate autocomplete options in the UI
- Retrieving content to augment prompts with additional context Think of the "Add from Google Drive" feature in Claude's
  interface - the application code determines which documents to show and handles injecting their content into the chat
  context.

# Prompts: User-Controlled

Prompts are triggered by user actions. Users decide when to run these predefined workflows through UI interactions like
button clicks, menu selections, or slash commands.

Prompts are ideal for implementing workflows that users can trigger on demand. In Claude's interface, those workflow
buttons below the chat input are examples of prompts - predefined, optimized workflows that users can start with a
single click.

# Choosing the Right Primitive

Here's a quick decision guide:

- Need to give Claude new capabilities? Use tools
- Need to get data into your app for UI or context? Use resources
- Want to create predefined workflows for users? Use prompts

You can see all three primitives in action in Claude's official interface. The workflow buttons demonstrate prompts, the
Google Drive integration shows resources in action, and when Claude executes code or performs calculations, it's using
tools behind the scenes.

These are high-level guidelines to help you choose the right primitive for your specific use case. Each serves a
different part of your application stack - tools serve the model, resources serve your app, and prompts serve your
users.


