# ai-discord-bot
A Discord bot made with the purpose of simplifying using AI through Discord. Installable as a personal Discord app or as a regular server bot.
Intended as a semi-successor to [the Akatsuki AI bot](https://github.com/osuAkatsuki/ai-discord-bot).
(Made because I am bored of revising IADS)

## Interactions
Here are the intended ways that you are supposed to interact with the bot.

### Discord Threads
This idea is inspired by the traditional chatbot website.
All messages sent in a thread will be considered valid context and prompt a response.

### Query command and replies.
A user may use the `/query` command to initiate a new context within an existing channel (cannot be used in threads).
The user may then reply to the response message to continue the context.

### Set Per-User Default Model
While the model for each interaction can be specified (when creating a thread or initiating a `/query` context), it is an
optional parameter. While it will defauly to DeepSeek Chat (as it is cheaper, while being more than enough for most queries),
the user can override this using the `/model` command.

## Implementation Details

### Bot Responses
The bot should aim to forward messages from the AI to account for all the common formats used in AI responses and Discord's limitations.

- Messages > 2000 characters (Discord message limit) shall be split up into multiple messages.
  - If a markdown code bloc is split, make sure to re-open it in the next message with the same language highlighting.
    - Or if the code is < 2000 characters, send it in a standalone message.
  - If a paragraph is split mid-way through, try to move it to the next message for reading continuity.
  - If a bloc contains standalone LaTeX, replace it with an image of the rendered LaTeX.
  - If a bloc contains inline LaTeX, replace it with an image of the full rendered paragraph.

### Message Context
The bot will aim to maximise input token count using message token count and always forward previously used attachments.
Branching contexts should be supported for replies in a linked-list-esque manner (replying to a previous bot response should continue the context from that point).

At the beginning of each context, the bot will also be provided with a `role:system` message explaining that it is currently in a Discord server.

### Supported Models
The bot aims to support these models due to their compliance with the OpenAI API.
- GPT 4.1
- GPT o1 (maybe pro and mini modes too)
- GPT o3-mini
- DeepSeek Chat
- DeepSeek Reasoner

## Data Models
Using a MySQL database to store message history and user settings.

### Messages
Used to keep track of message histories.

- Primary Auto-Increment ID
- Message ID (Discord determined)
- User ID (Discord determined, nullable)
- (Optional) Thread ID
- Message Content
- Model Used
- Token count (used for message maximising context history)
- Role (OpenAI format, used for flexibility)
- Is Response (whether the message is a response from the bot).


### Message Attachments
Used to keep track of attachments used in contexts (not sure how useful).

- Internal Message ID
- Attachment URL

### Threads
Used to keep track of which threads to track.

- Thread ID (Discord determined)
- Server ID (Discord determined)
- Tracked (boolean, threads can be set as untracked or tracked)

### Default Model
Used to store the user's default choice of model.

- User ID (Discord determined, unique)
- Model

### User Token Use
This will be used for cost analysis as messages can use more tokens than just the sum of the prompt and response.

- User ID (Discord determined)
- Input Tokens Used
- Output Tokens Used
- Total Cost (since costs change over time)
- Used At

### User Whitelist
A list of all users that are allowed to interact with the bot. 
