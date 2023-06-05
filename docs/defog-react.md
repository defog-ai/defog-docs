---
sidebar_position: 4
---

# Using Defog in React

## Installing defog-components
You can install defog in your React project with 
```bash
npm install defog-components --legacy-peer-deps
```

## Using defog-components in your project
You can import our `AskDefogChat` component in your React project, using the code below.

```javascript
import React from "react";
import { AskDefogChat } from "defog-react";

const App = () => {
    return (
        <AskDefogChat
            maxWidth={"100%"}
            maxHeight={500}
            apiEndpoint="YOUR_ENDPOINT"
            buttonText={"Ask Defog"}
            debugMode={true}
            sqlOnly={false}
            // additionalParams={{ "test": "test" }}
            // additionalHeaders={{ "test": "test" }}
        />
    );
};

export default App;
```

## Properties

### maxWidth
The maximum width the Defog component can take of it's parent container. We recommend that this always be `100%`

### maxHeight
The maximum height that the Defog component can be

### apiEndpoint
Your API endpoint can be any of the following:
- the endpoint of an AWS Lambda function or Google Cloud Fuction you created using `defog deploy`
- the endpoint of a service you have created on your own server
- a locally running instance of Defog using our [minimal webserver example](/defog-python)

### buttonText
The text of the button with the call to action. You can personalize this to reflect your own company

### debugMode
If this is true, the SQL query used to get the result will be shown. If it is false, only the data will be shown and the data will be hidden

### sqlOnly
If this is true, only the SQL query will be shown and the data will be hidden

### additionalParams
This can be a dict of any additional parameters you want to send along with your post request, in addition to the user's question and `previous_context`

### additionalHeaders
This can be any additional headers that you want to send along with your post request