# TIKI Client Libraries

The TIKI APIs comprise a set of HTTP REST APIs designed for seamless integration with any standard HTTP client. The Client Libraries serve as a user-friendly layer around the TIKI APIs, introducing methods for common operations such as authorization, licensing, capture, card-linked offers, and rewards. It is a collection of pre-existing code with minimal dependencies, offering a streamlined integration process with TIKI Rest APIs, which reduces the amount of code necessary for integration.

## TikiClient

TikiClient is the top-level entry point for of the TIKI Client Library. It offers simple methods that calls the underlying libraries to perform common operations. Programmers can use it to simplify the integration process or opt for individual libraries based on their specific needs.

### Parameters

- providerId: The data provider's TIKI Publishing ID. Get one in [TIKI Console](https://console.mytiki.com)
- userId: The user identification from the Provider. Ideally it is the same as the user id in the provider's internal systems. Avoid personal information in this field, hashing personal identifiable information like e-mails or personal ID numbers.
- Company: The company legal information. It is used to setup the terms for the licensed data.
  - name: The business legal name. e.g.: "Company Inc"
  - jurisdiction: The jurisdiction in which the business is stabelished. e.g.: "Tennessee, USA"
  - privacy: The business privacy terms URL. e.g: "https://companyinc.com/privacy"
  - terms: The user terms and conditions URL. e.g: "https://companyinc.com/terms"

### Scan Physical Receipt

- Method: `scan()`
- Description: Scans a physical receipt and publish it.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. `TikiClient.license.verify`: Verify the user license. If there's no valid license, throw an error.
  3. `TikiClient.capture.camera`: Open the device's camera to capture an image. It will return the binary image data. 
  4. `TikiClient.capture.publish`: Upload the captured image to TIKI.   

### Scrape Receipts from Emails

#### Add an Email Account

- Method: `login(provider)`
- Description: Autheticates with OAuth2 and add an email account for scraping receipts.
- Parameters: 
  - provider: "GOOGLE" or "OUTLOOK"
- Process:
  1. `TikiClient.email.login`: Login with OAuth2 and retrieve the authentication and refresh tokens.

#### Remove Email Account

- Method: `logout(email)`
- Description: Removes a previously added email account.
- Parameters: The email account to be removed. e.g: "client@gmail.com"
- Process:
  1. `TikiClient.email.logout`: Revoke the tokens with OAuth2.

#### Retrieve Connected Email Accounts

- Method: `accounts()`
- Description: Retrieves the list of connected email accounts.
- Process:
  1. Retrieve the emails that has tokens saved in the device's encrypted storage.

#### Scrape Email

- Method: `scrape()`
- Description: Initiates the process of scraping receipts from emails and publishing it.
- Process:
  1. Retrieve the saved tokens from the device's encrypted storage. 
  2. `TikiClient.capture.email`: Download the emails and attachments from known receipt senders.
  3. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  4. `TikiClient.capture.publish`: Publish the emails to TIKI.

### Card-Linked Offers

#### Add a Card

- Method: `card(last4, bin, issuer, network)`
- Description: Adds a card for accessing card-linked offers.
- Parameters: 
  - last4: last 4 digits of the card
  - bin: the Bank Identification Number
  - issuer: the card Issuer
  - network: "VISA", "MASTERCARD", "AMERICAN EXPRESS" or "DISCOVERY"
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. `TikiClient.clo.card`: add the card to the user

#### Get Offers

- Method: `offers()`
- Description: Retrieves card-linked offers for the user.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. `TikiClient.clo.offers`: retrieve user offers

### Submit transactions
- Method: `transaction()`
- Description: Send transaction for CLO matching
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. `TikiClient.clo.transaction`: send a transaction to TIKI

### Rewards

#### Get User Rewards

- Method: `rewards()`
- Description: Retrieves information about the user's rewards.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. `TikiClient.rewards.get`: get the user rewards

### UI

#### Show widget

- Method: `widget()`
- Start a widget that will prompt user to accept the license and show the UI for adding cards, accounts, get rewards, and redeem offers 

## TikiClient.auth

### Authenticate

- Method: `authenticate`
- Description: Authenticates with TIKI and save the auth and refresh tokens in the device's secure storage.
- Process:
  1. Authenticate with TIKI using the `providerId` and `userId`.
  2. Save the JWT in the device's encrypted storage.
  3. Return the JWT.

### Get Token

- Method: `token`
- Description: Retrieves the authentication token. 
  1. Get the JWT from the device's encrypted storage.
  2. If there is no token saved, use `TikiClient.auth.authenticate`, save and return it.
  3. If it is expired, use `TikiClient.auth.refresh` to update and return the updated JWT.

### Revoke Token

- Method: `revoke`
- Description: Revokes the authentication token.
- Process:
  1. Revoke the JWT with TIKI.
  2. Remove the token from the device's encrypted storage.

### Refresh Token

- Method: `refresh`
- Description: Refreshes the authentication token.
- Process:
  1. Request a new token from TIKI.
  2. Update the saved token in the device's encrypted storage
  3. Return the updated token

## TikiClient.license

### Create a License

- Method: `create`
- Description: Creates a new license for the user.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Get or create a Title for the user with the following parameters:
    - ptr: userId
    - tag: PURCHASE_HISTORY
  3. Create a License for the user
    - titleId: The title created before
    - uses
      - usecase: attribution
      - destination: "*"
    - terms: autogenerated terms from Company information
    - description: "Receipt data from e-mail or physical receipts scan"
  4. Return the License Record.

### Get Active License

- Method: `get`
- Description: Retrieves the user's active license.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Get the Title using the userId as PTR
  3. Get the latest License with the Title ID.
  4. Return the License

### Revoke a License

- Method: `revoke`
- Description: Revokes the user's existing license.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Get the Title using the userId as PTR
  3. Create a License without usecases.
  4. Return the License

### Verify License

- Method: `verify`
- Description: Verifies the validity of the user's license.
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Get the Title using the userId as PTR
  3. Get the latest License with the Title ID.
  4. Return a boolean checking if the License has the correct usecases
  
## Email

### Add an Email Account

- Method: `login`
- Description: Authenticate with OAuth and add an email account for scraping receipts.
- Parameters: 
  - provider: "GOOGLE" or "OUTLOOK"
- Process:
  1. Login with OAuth2 and retrieve the authentication and refresh tokens.
  2. Save the tokens into device's encrypted storage.

### Retrieve Connected Email Accounts

- Method: `accounts`
- Description: Retrieves the list of connected email accounts.
- Process:
  1. Get the emails with saved tokens from encrypted storage.
  2. Return just the emails, not the tokens.
   
### Remove Email Account

- Method: `logout`
- Description: Removes a previously added email account.
- Parameters: The email account to be removed.
- Process:
  1. Revoke the tokens with the email provider
  2. Remove the email tokens from encrypted storage
   
## TikiClient.capture

### Capture Receipt Image

- Method: `camera`
- Description: Captures an image of a receipt for processing.
- Process:
  1. Open the device's camera
  2. Return the image captured by the user or `null/nil` if the user cancelled.

### Download and Publish Email Receipt Data

- Method: `email`
- Description: Downloads potential receipt data from a known list of receipt email senders.
- Parameters: 
  - `onPublish`: a function to be called on each uploaded email. It receives the `receiptId` as parameter. 
- Process:
  1. `TikiClient.email.accounts`: Get connected email accounts.
  2. Loop through each account async
     1. Get the account token from encrypted storage. Refresh if expired.
     2. Index new async
        1. Get the last index datetime.
        2. Index all emails received after it.
        3. Save the index datetime.
     3. Index older async
        1. Get the latest indexed email date and time.
        2. Index all emails sent before that.
        3. Save the last indexed email date and time.
     4. Download async
        1. Get the indexed emails ids.
        2. Download the email body and its attachments by id.
        3. Call `TikiClient.email.publish` for each downloaded e-mail.
        4. Send the `receiptId` to the callback and remove the indexed email id.

### Upload Receipt Images

- Method: `publish`
- Description: Uploads receipt images or email data for receipt data extraction.
- Parameters: 
  - A binary image or the email body and an array with its attachments.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Send the image/email data to the REST API.
  3. Returnt the `receiptId` to check publishing process.

### Check publishing status

- Method: `status`
- Description: Give the status of the published data.
- Parameters: 
  - The `receiptId` received when publishing the data.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Check the publishing status in the REST API.

## TikiClient.clo

### Add a Card to the User

- Method: `card`
- Description: Adds a card to the user's account.
- Parameters: [Specify parameters]
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Use the CLO rest API to add the card

### Get Offers

- Method: `offers`
- Description: Retrieves card-linked offers for the user.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Use the CLO rest API to add get the offers and return them

### Get User Rewards

- Method: `rewards`
- Description: Retrieves information about the user's rewards.
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Use the CLO REST API to add get the user rewrds and return them.

### Publish transaction information

- Method: `transaction`
- Description: Send transaction information to match CLOs
- Process:
  1. `TikiClient.auth.token`: Authenticate with TIKI REST API and retrieve the authentication token
  2. Use the CLO REST API to send the transaction information.