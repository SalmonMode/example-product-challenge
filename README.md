# Example Product Challenge

Given a simple web-based service, i.e. a chat room service, as a fondation, this serves as an example of how changes in product implementation can introduce maintenance work for the test suite. The challenge is in creating a test and test tool structure that reduces the amount of maintanence work required, increases the readability of the code, reduces the complexity of diffs, and even increases the readability of diffs. The readability of the test output is also of significant performance.

### The rules are as follows:

1. All programming best practices should be applied in the most sensible fashion. If a best practice is not factored in, this does not entirely disqualify the proposed implementation, but rather means the implementation needs to be updated to factor it in before the implementation can be considered complete.
2. In the same way that general programming best practices should be applied, so must testing best practices. This means:
    1. 1 assert per test method
    2. Each test should target one specific behavior that can only be tested in the way the test is built
    3. "Arrange, Act, Assert", not "Arrange, Act, Assert, Act, Assert, ..."
    4. A test should involve only the bare minimum of what it needs to test the targetted behavior
    5. Tests are atomic
    6. Any test can be run on its own
    7. Tests shouldn't depend on other tests to be executed successfully
    8. More can be found here: https://salmonmode.github.io/Building-Good-Tests/ (not a complete list)
3. While the developer can know what the eventual change in the product will be while they are writing their initial implementation, alternate changes may be presented in the future, so the initial implementation should be written such that it can be modified more easily to handle any changes.
4. The acceptance criteria of the product must be provided before beginning.
5. The general structure of the product and the backen API must be provided before beginning, including the locators for significant frontend elements.
6. Code must not have linting errors.
7. Code must not have warnings.
8. Code must be formatted by the most prevalent formatting system for that language (e.g. for Python, this is `black`).
9. The initial implementation will be judged on its own once it is considered "complete".
    1. An implementation can be blocked from completed status if it does not adhere to a best practice, unless another, conflicting best practice can be cited as the reason for not adhering to the original best practice (e.g. "DRY was not followed in this case because of DAMP")
10. The same goes for the updated implementation.
11. The initial implementation must be judged before the implementation after the change is introduced, as the diff between the two is part of what is judged, and it can't be properly judged if the initial implementation is a moving target.
12. Repeated attempts at an initial implementation are allowed, but they nullify any judgement received for the updated implementation.
13. Pre-existing tools and libraries are allowed, but their effectiveness and the quality of the APIs they provide can be factored into judgement. Everything else must be provided in the proposed implementation.
14. Every test must clean up after itself, so that it leaves the environment in a state that makes it look like the test was never run.

### Judgement criteria:

The following should be factored into judgement as these are the actual criteria we judge our real work by during active development:

- Test code readability
- Test name readability
- Test result readability
- Test effectiveness
- Tool readability
- Diff length after change
- Diff complexity/readability after change
- Difficulty in understanding the resulting implementation after the change
- Best practice adherence

## The Product

The product will be a simple chat room service. There are 3 pages for this service:

### Chat Room Creation Page (`https://staging.chat-site.com/`)

This page allows a user to create a chat room using a simple form with only a single text field along with a submit button. The submitted chat room name must be unique, otherwise the API will return an error, and the frontend will present it. If the error is shown, it will contain the text `This name is already taken` if the name is already taken. The moment the enterred text is modified, the error will be automatically dismissed. The error can also be manually dismissed with an included `x` button.

There is frontend validation to ensure that valid text must be enterred. If no valid text is enterred, then the submit button will be disabled.

If the creation is successful, a modal will appear in front of the form with the text `Join Link:` and in another element, it will provide the join URL, and the modal will have a `x` button to close it. 

The join link is determined by the ID given to the created conversation by the backend. If the ID is `0`, then the join link would be `https://staging.chat-site.com/join?id=0`

#### Locators:
Element|Locator
-------|-------
Chat Room Name Text Field | `By.Css("#chat-room-name")`
Submit Button | `By.Css("button[type='submit'")`
Error Message Wrapper | `By.Css("#error")`
Error Message Text | `By.Css("#error p")`
Error Message Close button | `By.Css("#error button")`
Join URL Modal Wrapper | `By.Css("#join")`
Join URL Modal Join Text | `By.Css("#join p")`
Join URL Modal Join URL | `By.Css("#join a")`
Join URL Modal Close button | `By.Css("#join button")`

### Join Chat Room Page (`https://staging.chat-site.com/join?id=<room id>`)

This page is accessed by following the URL provided after creating the chat room. There is only a simple form with only a single text field along with a submit button. The submitted username name must be unique, otherwise the API will return an error, and the frontend will present it. If the error is shown, it will contain the text `This name is already taken` if the name is already taken. The moment the enterred text is modified, the error will be automatically dismissed. The error can also be manually dismissed with an included `x` button.

There is frontend validation to ensure that valid text must be enterred. If no valid text is enterred, then the submit button will be disabled.

If there is no conflict, the name is "reserved", and the user is given a cookie along with a success message. The success message is not relayed to the user. Instead, the JS of the page recognizes the success message and navigates to the chat room page.

If a user already has a cookie for this chat room, the JS of the page will automatically navigate to the chat room page without showing the form to the user.

If the user already has a cookie gained from joining another chat room, but not for this chat room, then they will be shown the form, and that cookie will be associated with their session after a successful name reservation. If no successful reservation is made, nothing is done with the cookie. If a successful reservation is made, then the page will navigate to the chat room as expected.

If the user attempts to join an already ended chat room, the form will be disabled and they will see a modal in front of everything that says `This chat room has ended` and has a button that says `Delete Chat Room`, which, if clicked, will delete the chat room. If the delete button is clicked, the page will navigate back to the create chat room page.

#### Locators:
Element|Locator
-------|-------
Chat Room Username Text Field | `By.Css("#name")`
Submit Button | `By.Css("button[type='submit'")`
Error Message Wrapper | `By.Css("#error")`
Error Message Text | `By.Css("#error p")`
Error Message Close button | `By.Css("#error button")`
Chat Room Ended Modal | `By.Css("#modal")`
Chat Room Ended Modal Text | `By.Css("#modal .text")`
Chat Room Ended Modal Delete Button | `By.Css("#modal button")`

### Chat Room Page (`https://staging.chat-site.com/room?id=<room id>`)

This is a simple page with only a single form and the chat feed. The form contains only a single text field along with a submit button.

There is frontend validation to ensure that valid text must be enterred. If no valid text is enterred, then the submit button will be disabled.

The chat feed contains the series of discrete message wrappers representing each message that has been sent. The entire feed is isolated from the form, so they can be located separately. Each message in the feed contains only the username of the user that sent the message, the text of that message, and the timestamp for when that message was sent (not from the client, but when the server accepted the message sent from the client and then attempted to forward it to all users. For the sake of simplicity, the time will use the ISO 8601 format (see API examples below for an example) and use the UTC timezone.

When a user sends a message, the message elements will not be automatically generated in the message feed. Instead, the message will only appear once it is received from the server.

If a user joins a chat room that already had messages sent, they will be given the list of all message details so they can reconstruct the message feed up until that point.

If any user sends the message `/end`, the chat room will be ended for all parties, and everyone will see a modal in front of everything that says `This chat room has ended`, and a button that says `Delete Chat Room`, which, if clicked, will delete the chat room. If the delete button is clicked, the page will navigate back to the create chat room page.

If a user goes to a chat room that already ended, but they had already joined, they will see the chat feed and form behind the modal.

If a user attempts to jump straight to the chat room page without having joined, they will receive a default 403 page, and a 404 if the chat room has been deleted.

#### Locators:
Element|Locator
-------|-------
Input field | `By.Css("#text")`
Submit Button | `By.Css("button[type='submit'")`
Chat Room Ended Modal | `By.Css("#modal")`
Chat Feed Wrapper | `By.Css("#chat-feed")`
Message Wrapper (non-unique) | `By.Css(".messageWrapper")`
Message Sender Name (non-unique) | `By.Css(".messageWrapper .senderName")`
Message Send Time (non-unique) | `By.Css(".messageWrapper .sendTime")`
Message Text (non-unique) | `By.Css(".messageWrapper .text")`
Chat Room Ended Modal | `By.Css("#modal")`
Chat Room Ended Modal Text | `By.Css("#modal .text")`
Chat Room Ended Modal Delete Button | `By.Css("#modal button")`

## The Backend API

The frontend communicates with the backend using a basic API. Through this API, chat rooms are created, their details gotten, joined (i.e. a username is reserved and a cookie is acquired), and deleted, and there is a websocket endpoint that the actual chat messages go through. Everything is done through JSON and query strings.

Here is a breakdown of the API:

### `/api/chat`

#### `POST`

Creates a chat room

Example request body:

```json
{
  "name": "My Chat Room"
}
```

Example response:

```json
{
  "name": "My Chat Room",
  "id": 0,
  "status": "live"
}
```

 If a chat room with that name already exists, then a `409` response (conflict) will come back.
 
 #### `GET`

Gets the details of a chat room. No JSON is needed for the request, as a query string will be used.

Example query string:

```
https://staging.chat-site.com/api/chat?id=0
```

Example response:

```json
{
  "name": "My Chat Room",
  "id": 0,
  "status": "live",
}
```

If a chat room with that ID does not exist, then a `404` response (conflict) will come back.
 
  #### `DELETE`

Gets the details of a chat room. No JSON is needed for the request, as a query string will be used.

Example query string:

```
https://staging.chat-site.com/api/chat?id=0
```

Example response:

```json
{
  "name": "My Chat Room",
  "id": 0,
  "status": "deleted",
}
```

If a chat room with that ID does not exist, then a `404` response (not found) will come back.

### `/api/chat/join`

#### `POST`

Combines a query string and request body to reserve a username in a given chat room

Example Query String:

```
https://staging.chat-site.com/api/chat/join?id=0
```

With example request body:

```json
{
  "name": "My Name"
}
```

If the reservation is successful, a `200` response is returned. If there's a user with that name in that chat room already, a `409` response (conflict) will be returned.

### `/api/chat/ws?id=<room id>`

Example query string:

```
https://staging.chat-site.com/api/chat/chat/ws?id=0
```

This is the websocket endpoint messages are exchanged over. For the sake of this exercise, we can assume that messages are sent and received using JSON, and that there is an asynchronous nature to how websockets work that must be accounted for. Implementing a websocket client can take a lot of code, so we will assume that
1. the client can be instantiated as long as the URL, chat room ID, cookie, and username are provided
2. messages can be sent using a `send` method that only takes the message text as an argument
3. There is a message queue provided by the client containing all messages received
4. that a wait method exists which allows you to wait for any number of new incoming messages so long as the number of messages you want it to have in its queue is provided
5. (Optional) It may be assumed that the client deals with `Message` objects, where the name, message, and time are attributes of an instance of that type, and the time attribute is a `datetime` object, or the relevant type for your language (this just makes the code mroe straightforward)

The number you want it to have is needed because it may receive the message you want it to wait for before you manage to call the wait method.

Example send message body:

```json
{
    "name": "My Name",
    "message": "Hello!"
}
```

Example message received body:

```json
{
    "name": "My Name",
    "message": "Hello!",
    "time": "2020-01-10T15:17:08.132263"
}
```

After assuming these are provided by the client, any desired features on top of this must be implemented in the proposed solution.

Here's an example of how the client might look and be used:

```py
>>> base_url = "https://staging.chat-site.com"
>>> bob_client = ChatClient(base_url=base_url, chat_room_id=0, cookie=bob_cookie, username="Bob")
>>> sally_client = ChatClient(base_url=base_url, chat_room_id=0, cookie=sally_cookie, username="Sally")
>>> bob_client.send(Message("Hi, Sally!"))
>>> sally_client.send(Message("Hi, Bob!"))
>>> bob_client.wait_for_messages(total_received=2)
>>> sally_client.wait_for_messages(total_received=2)
>>> print(bob_client.messages)
[<Message from: Bob at: 2020-01-15 15:58:13.055335 text: 'Hi, Sally!'>, <Message from: Sally at: 2020-01-15 15:58:13.384093 text: 'Hi, Bob!'>]
>>>
```
