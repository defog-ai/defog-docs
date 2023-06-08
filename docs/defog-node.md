---
sidebar_position: 3
---

# Using the Node Library

Once you have set up defog using `defog init` from the command line, you can start using it in your applications.

## Initializing defog in your Node app
Initializing Defog is as simple as

```javascript
import Defog from defog
defog = Defog(
    api_key = "YOUR_DEFOG_API_KEY",
    db_type = "YOUR_DB_TYPE", // must be one of postgres, redshift, mysql, bigquery. Our Node library does not yet support mongo and snowflake
    db_creds = YOUR_DB_CREDS // example formats here: https://github.com/defog-ai/defog-node#usage
)
```

## Using Defog to answer questions

### To automatically convert a query to SQL, and run it on your DB
```javascript
const defog = Defog(...)
const results = await defog.runQuery("how many users do we have?")
results
{
  columns: [ 'num_users' ],
  data: [ { num_users: '49990' } ],
  ran_successfully: true,
  query_generated: 'SELECT COUNT(DISTINCT userid) AS num_users\nFROM users;',
  error_message: undefined,
  previous_context: [
    'how many users do we have?',
    'SELECT COUNT(DISTINCT userid) AS num_users\nFROM users;'
  ],
  reason_for_query: '\n' +
    'The question asks for the total number of users in the database. We can answer this question by simply counting the number of distinct user IDs in the users table.\n',
  query_db: 'redshift'
}
```

Optionally, you can also pass the list in `previous_context` to the `runQuery` function, so that the model can adapt to your question.

```javascript
await defog.runQuery("how many from san francisco?", hard_filters=null, previous_context=previous_results['previous_context'])
{
  columns: [ 'num_users' ],
  data: [ { num_users: '50' } ],
  ran_successfully: true,
  query_generated: "SELECT COUNT(*) AS num_users\nFROM users\nWHERE city = 'San Francisco';",
  error_message: undefined,
  previous_context: [
    'how many users do we have',
    'SELECT COUNT(DISTINCT userid) AS num_users\nFROM users;',
    'how many from san francisco?',
    "SELECT COUNT(*) AS num_users\nFROM users\nWHERE city = 'San Francisco';"
  ],
  reason_for_query: '\n' +
    'The question asks for the number of users from San Francisco. The users table contains the city column, which can be used to filter for users from San Francisco. We can use the COUNT() function to count the number of users that match the filter.\n',
  query_db: 'redshift'
}
```

### Adding hard filters for secure data access
If you want to restrict which results a user can see, you can just add the `hard_filters` property to the `runQuery` function.

```javascript
const defog = Defog()
await defog.runQuery(
    "how many users do we have?",
    hard_filters=" Only include data for the venueid 220, and no other venue",
    previous_context=[]
)
{
  columns: [ 'total_users' ],
  data: [ { total_users: '49990' } ],
  ran_successfully: true,
  query_generated: 'SELECT COUNT(DISTINCT userid) AS total_users\nFROM users;',
  error_message: undefined,
  previous_context: [
    'how many users do we have?',
    'SELECT COUNT(DISTINCT userid) AS total_users\n' +
      'FROM users\n' +
      'WHERE venueid = 220; \n' +
      '\n' +
      '-- We use the COUNT() function to count the number of distinct user IDs in the `users` table. We also use the DISTINCT keyword to ensure that each user is only counted once. We add a WHERE clause to filter the results to only include data for the venueid 220, as specified in the prompt.'
  ],
  reason_for_query: '\n' +
    'The question asks for the total number of users in the database. The relevant table for this query is the `users` table, which contains a unique identifier for each user. We can simply count the number of distinct user IDs in the `users` table to get the total number of users.\n',
  query_db: 'redshift'
}
```


## Fine-tuning instructions with updateGlossary

You can use the updateGlossary function in Defog to fine-tune instructions that the model can follow while generating queries. An example of this is below:

```javascript
const defog = Defog(...)
await defog.updateGlossary("If a user asks for data in this week, return data for the current ISO week and not for the last 7 days.")
```