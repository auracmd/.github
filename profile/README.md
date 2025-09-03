## auracmd: The Autonomous Command Agent
`auracmd` is an autonomous, multi-agent system designed to fulfill natural language requests by orchestrating a series of system commands. It acts as a smart central hub that intelligently coordinates between user input, an AI reasoning engine, and a command execution service.

### How it works: A self-correcting loop
The system uses an iterative **"Act, Execute, Reflect"** cycle:

- **Request:** A user submits a request, such as "Write me a poem and save it to my desktop," through the middleService.

- **Act:** The AIService (a specialized reasoning agent) analyzes the user's intent, the system's state, and available tools. It generates a structured command, like `{"cmd":"create_file", "path":"$HOME/Desktop/poem.txt"}`.

- **Execute:** The middleService routes this command to the cmdService, which attempts to perform the action.

- **Reflect:** The cmdService returns the command's output (e.g., "`Folder not found: Desktop`"). The AIService analyzes this output, identifies the error, and generates a new, corrective command (e.g., `{"cmd":"create_dir", "path":"$HOME/Desktop"}`). This cycle continues until the task is complete.

- **Result:** When the goal is achieved, the middleService sends the final result back to the user, including details like the file's final name and location.

>[!NOTE]
>While this is just an example of the internal process, the main goal is to have a system that can effectively fulfill a user's request.

This communication allows auracmd to handle edge cases and adapt its strategy, ensuring a successful outcome for multi-step tasks.

### Architecture overview

>[!IMPORTANT]
>We will start by making each service a library. We are not sure if it's a good idea to go as far as making each one a microservice. I doubt there will be a significant performance hit since this is a very simple app, but if the overhead turns out to be too much, it will be easier to transition from libraries to microservices using RPC later on.
>
>Doing it this way is meant to simplify the development process, as each component will be a separate library instead of being tightly coupled as part of the main app. This allows for better modularity and easier maintenance, since individual components can evolve independently without disrupting the overall system.

##### auracmd is powered by three collaborating microservices:

![topology](https://github.com/auracmd/.github/blob/main/profile/AI-Agent-Orch%20(1).png)

- middleService (**Orchestrator**): The central hub that manages the conversation flow, routes requests, and handles the Act, Execute, Reflect loop.
- AIService (**Reasoning Agent**): Contains the core AI logic, powered by an LLM, to interpret user intent, plan commands, and analyze feedback from the execution service.
- cmdService (**Execution Agent**): A specialized service that securely executes system commands, receives their output, and returns it to the orchestrator.
